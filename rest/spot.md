# REST: spot trading

## Назначение

Группа `/spot/*` покрывает основной торговый контур:

- выставление ордеров;
- отмену и cancel-and-replace;
- batch операции;
- чтение активных и исторических состояний;
- публичный snapshot стакана через отдельный `GET /spot/orderbook`.

Все command и private read методы этого раздела рассчитаны на вызов по `API key` и HMAC-подписи. Публичный `GET /spot/orderbook` описан в [market.md](market.md).

## Базовый flow ордера

1. Клиент читает `/market/exchange-info`.
2. Клиент валидирует рынок, order type, time in force, precision, `minNotional`, execution flags и price protection.
3. Клиент формирует `OrderCreateRequest`.
4. Клиент подписывает `POST /api/v1/gateway/spot/orders/create`.
5. Gateway возвращает `202 Accepted` и `CommandResponseDto`.
6. Финальное состояние отслеживается через:
   - `GET /spot/orders/{orderId}`
   - `GET /spot/orders/active`
   - `GET /spot/orders/history`
   - private WebSocket `user`

Важно: create/cancel/cancel-and-replace это асинхронные команды. Ответ `202` подтверждает прием в обработку, а не окончательное исполнение.

## Состояния ордера

Для интегратора ордер проходит через два слоя состояний:

1. command acceptance на уровне gateway;
2. фактический жизненный цикл ордера в matcher / history storage.

Практически безопасная модель:

- `202 Accepted` означает только прием команды;
- актуальное состояние надо читать через `GET /spot/orders/{orderId}` или `user` events;
- история закрытых и исполненных ордеров закрепляется в `orders/history`.

## OrderCreateRequest

| Поле | Тип | Назначение |
|------|-----|------------|
| `marketId` | string | Идентификатор рынка, обычно `BTC_USDT` |
| `baseAssetId` | string | Базовый актив |
| `quoteAssetId` | string | Котируемый актив |
| `side` | enum | `BUY` или `SELL` |
| `orderType` | enum | `LIMIT`, `MARKET`, `STOP_LIMIT`, `STOP_MARKET` |
| `timeInForce` | enum | `GTC`, `IOC`, `FOK`, `AON`, `GTD` |
| `quantity` | string | Обязательное количество |
| `quoteAmount` | string | Quote-side budget для части market-сценариев |
| `price` | string | Цена лимитной ноги |
| `stopPrice` | string | Trigger price для stop-ордеров |
| `postOnly` | boolean | Maker-only ограничение |
| `accountType` | enum | Поддерживается только `SPOT` |
| `marketSeq` | integer | Техническое поле рынка |
| `expiresAt` | string | ISO-8601 timestamp для `GTD` |
| `timestamp` | string | Клиентская временная метка |
| `clientOrderId` | string | Клиентский идентификатор ордера |

## Фактические правила валидации

### Общие

- `quantity` должна быть положительной decimal строкой.
- Если переданы `price`, `stopPrice`, `quoteAmount`, они тоже должны быть положительными decimal значениями.
- `accountType` должен быть `SPOT`.
- Рынок должен существовать и быть в статусе `TRADING`.
- `orderType` и `timeInForce` должны поддерживаться рынком.
- Проверяются шаги, precision, min/max quantity и `minNotional`.

### Order types

| Тип | Что обязательно | Особенности |
|-----|------------------|-------------|
| `LIMIT` | `price`, `quantity` | Может использовать `postOnly` |
| `MARKET` | `quantity` | Для `BUY + FOK` обязателен `quoteAmount` |
| `STOP_LIMIT` | `stopPrice`, `price`, `quantity` | Может использовать `postOnly` |
| `STOP_MARKET` | `stopPrice`, `quantity` | Для `BUY + FOK` обязателен `quoteAmount` |

### Time in force

| TIF | Правило |
|-----|---------|
| `GTC` | Базовый сценарий хранения до отмены/исполнения |
| `IOC` | Немедленное частичное/полное исполнение, остаток отменяется |
| `FOK` | Должен исполниться целиком, иначе отменяется |
| `AON` | Поддерживается в enum; внутренне может маппиться в `GTC`, но доступность задается рынком |
| `GTD` | Требует `expiresAt` в будущем |

### `postOnly`

`postOnly` допустим только при одновременном выполнении двух условий:

- `orderType` это `LIMIT` или `STOP_LIMIT`;
- `timeInForce` это `GTC` или `GTD`.

Во всех остальных комбинациях запрос должен считаться невалидным.

### `quoteAmount`

`quoteAmount` обязателен для:

- `MARKET` + `BUY` + `FOK`;
- `STOP_MARKET` + `BUY` + `FOK`.

В остальных сценариях сервер может вычислять нужный quote budget сам или игнорировать клиентское значение.

### `expiresAt`

Для `GTD`:

- поле обязательно;
- формат должен быть валидным ISO-8601;
- значение должно лежать строго в будущем.

## Командные методы

### `POST /spot/orders/create`

Создает один ордер и возвращает `202 Accepted`.

`CommandResponseDto`:

| Поле | Назначение |
|------|------------|
| `success` | Команда принята gateway |
| `message` | Технический текст результата |
| `orderId` | Системный numeric id ордера |
| `clientOrderId` | Эхо клиентского id |

Recommended usage:

1. сохранить `orderId`;
2. начать follow-up через `GET /spot/orders/{orderId}` или private WS;
3. не считать команду завершенной только по `success=true`.

### `POST /spot/orders/cancel`

Отменяет один ордер.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `orderId` | string | нет | Системный order id |
| `clientOrderId` | string | нет | Клиентский order id |

Нужно передать хотя бы одно из двух значений.

### `POST /spot/orders/cancel-and-replace`

Атомарный сценарий: отменить старый ордер и поставить новый.

Тело:

- `cancelOrderId` или `cancelClientOrderId`;
- `newOrder`.

Важно: `cancelClientOrderId` присутствует в API-модели, но на wire-path matcher принимает numeric cancel target. Поэтому нельзя обещать этот сценарий как полностью самостоятельный идентификатор без предварительного resolve в numeric `orderId`.

### `POST /spot/orders/batch/create`

Создает несколько ордеров одним запросом. Каждый элемент обрабатывается независимо.

### `POST /spot/orders/batch/cancel`

Массовая отмена по `orderIds`. Список должен содержать numeric ids.

### `POST /spot/orders/cancel-all`

Отменяет все активные ордера пользователя, опционально по одному `marketId`.

## Методы чтения

### `GET /spot/orders/{orderId}`

Возвращает текущее состояние одного ордера.

Используется для poll после `create`, `cancel` и `cancel-and-replace`.

Ключевые поля ответа `SpotOrderHistoryDto`:

| Поле | Назначение |
|------|------------|
| `orderId` | Системный id |
| `clientOrderId` | Клиентский id |
| `symbol` | Рынок |
| `side` | Направление |
| `type` | Тип ордера |
| `status` | Текущее состояние |
| `timeInForce` | Режим исполнения |
| `price` | Лимитная цена |
| `quantity` | Исходное количество |
| `executedQuantity` | Уже исполненный объем |
| `remainingQuantity` | Оставшийся объем |
| `avgExecutionPrice` | Средняя цена исполнения |
| `stopPrice` | Trigger price |
| `postOnly` | Maker-only флаг |
| `feeAsset` / `feePaid` | Комиссия |
| `createdAt` / `updatedAt` / `closedAt` | Таймлайн ордера |
| `rejectReason` | Причина reject, если был отказ |
| `trades[]` | Сделки по ордеру, если включены |

### `GET /spot/orders/active`

Возвращает все активные ордера пользователя.

Фильтры:

- `symbol`
- `orderId`
- `clientOrderId`
- `orderTypeFilter`

Ответ `SpotActiveOrdersResponseDto` содержит массив `orders[]`, где каждый элемент повторяет основную карточку активного ордера: `orderId`, `symbol`, `side`, `type`, `status`, `price`, `quantity`, `executedQuantity`, `remainingQuantity`, `avgExecutionPrice`, `stopPrice`, `postOnly`, `createdAt`, `updatedAt`.

### `GET /spot/orders/history`

Cursor-based история ордеров.

Ключевые query-поля:

- `symbol`
- `orderId`
- `clientOrderId`
- `orderTypeFilter`
- `side`
- `dateFromMs`
- `dateToMs`
- `includeTrades`
- `limit`
- `cursor`

Ответ `SpotOrdersHistoryResponseDto`:

| Поле | Назначение |
|------|------------|
| `orders[]` | Исторические записи ордеров |
| `nextCursor` | Курсор следующей страницы |
| `hasMore` | Есть ли следующая страница |

### `GET /spot/executions/history`

История execution/fill событий для reconciliation и trade ledger.

Ответ `ExecutionsHistoryResponseDto`:

| Поле | Назначение |
|------|------------|
| `executions[]` | Список execution/fill записей |
| `nextCursor` | Курсор следующей страницы |
| `hasMore` | Флаг продолжения |

Ключевые поля execution:

- `executionId`
- `tradeId`
- `orderId`
- `clientOrderId`
- `symbol`
- `side`
- `isMaker`
- `price`
- `quantity`
- `quoteQuantity`
- `feeAsset`
- `feeAmount`
- `feeRate`
- `isSelfTrade`
- `executedAt`

### `GET /spot/trades/history`

Постраничная история executed trades для аналитики и отчетности.

Лучше подходит для trade-centric аналитики, где важна каждая отдельная сделка как элемент отчета.

Ответ `TradesHistoryResponseDto`:

- `trades[]`
- `total`
- `limit`
- `offset`

### `GET /spot/deals/history`

Order-centric представление matcher deals с акцентом на связку `tradeId + orderId`.

Лучше подходит для reconciliation order execution на уровне matcher-потока.

Ответ `DealsHistoryResponseDto`:

- `deals[]`
- `total`
- `limit`
- `offset`

## Как выбрать history endpoint

| Endpoint | Когда использовать |
|----------|--------------------|
| `/spot/orders/history` | Нужен полный жизненный цикл ордера |
| `/spot/executions/history` | Нужны fills/executions и fee details |
| `/spot/trades/history` | Нужен trade-centric отчет |
| `/spot/deals/history` | Нужна matcher-oriented сверка `tradeId + orderId` |

## Типичные причины ошибок

- неподдерживаемый `orderType` или `timeInForce` для рынка;
- `price`, `quantity` или `stopPrice` не проходят decimal/precision validation;
- нарушены `tickSize`, `lotSize`, `minQty`, `maxQty`, `minNotional`;
- `postOnly` используется с недопустимым типом ордера или TIF;
- для `BUY + FOK` market-логики не передан `quoteAmount`;
- `GTD` передан без `expiresAt` или с прошлой датой;
- указан несуществующий или уже закрытый order id.

## Что важно бизнес-аналитику

- `exchange-info` это источник торговых правил рынка, а не просто список инструментов.
- `202 Accepted` подтверждает прием команды, а не финальный outcome.
- `cancel-and-replace` нужно описывать как отдельный атомарный сценарий.
- Источник правды по жизненному циклу ордера это комбинация REST history и private WebSocket `user`.
- Для production-интеграции безопасно считать `clientOrderId` обязательным, даже если DTO допускает пустое значение.

## См. также

- [market.md](market.md)
- [../ws/private.md](../ws/private.md)
- [../flows/quickstart.md](../flows/quickstart.md)
