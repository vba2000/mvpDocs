# REST: spot trading

## Назначение

Группа `/spot/*` покрывает основной торговый контур: выставление ордеров, отмену, пакетные операции и чтение активных/исторических данных.

Все методы раздела требуют HMAC-аутентификацию по `API key`.

## Бизнес-флоу

### Базовый сценарий создания ордера

1. Клиент получает market metadata через `/market/exchange-info`.
2. Клиент валидирует `marketId`, `orderType`, `timeInForce`, precision и минимальные шаги.
3. Клиент формирует `OrderCreateRequest`.
4. Клиент подписывает `POST /api/v1/gateway/spot/orders/create`.
5. Gateway принимает команду и возвращает `CommandResponseDto`.
6. Дальнейшая жизнь ордера отслеживается через:
   - `GET /spot/orders/active`
   - `GET /spot/orders/history`
   - private WebSocket поток `user`

## Командные методы

### `POST /spot/orders/create`

Создает один ордер.

Ключевые поля `OrderCreateRequest`:

| Поле | Тип | Назначение |
|------|-----|------------|
| `marketId` | string | Идентификатор рынка, обычно `BTC_USDT` |
| `baseAssetId` | string | Базовый актив |
| `quoteAssetId` | string | Котируемый актив |
| `side` | enum | `BUY` или `SELL` |
| `orderType` | enum | `LIMIT`, `MARKET`, `STOP_LOSS`, `STOP_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` |
| `timeInForce` | enum | `GTC`, `IOC`, `FOK`, `AON`, `GTD` |
| `quantity` | string | Количество в base asset |
| `quoteAmount` | string | Сумма в quote asset для некоторых market-сценариев |
| `price` | string | Цена лимитного ордера |
| `stopPrice` | string | Триггер для stop/take-profit |
| `postOnly` | boolean | Ограничение maker-only |
| `accountType` | enum | Обычно `SPOT` |
| `marketSeq` | integer | Технический sequence номер рынка |
| `expiresAt` | string | Срок действия, если поддерживается стратегией |
| `timestamp` | string | Клиентская метка времени |
| `clientOrderId` | string | Клиентский id ордера |

Практически важные требования:

- `clientOrderId` рекомендуется всегда передавать;
- `price` обязателен для лимитных типов;
- `stopPrice` нужен для stop/take-profit сценариев;
- все количества и цены должны быть строками.

Ответ `CommandResponseDto`:

| Поле | Значение |
|------|----------|
| `success` | Команда принята или отклонена |
| `message` | Текст результата |
| `orderId` | Системный идентификатор ордера |
| `clientOrderId` | Эхо клиентского идентификатора |

### `POST /spot/orders/cancel`

Отменяет один активный ордер.

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `orderId` | string | нет | Системный id ордера |
| `clientOrderId` | string | нет | Клиентский id ордера |

Нужно передать хотя бы один идентификатор.

### `POST /spot/orders/cancel-and-replace`

Атомарный сценарий: отменить старый ордер и сразу выставить новый.

Полезен, когда клиент не хочет ловить окно между отдельными cancel/create.

Тело `CancelAndReplaceRequest`:

- `cancelOrderId` или `cancelClientOrderId`;
- `newOrder` с полной структурой `OrderCreateRequest`.

### `POST /spot/orders/batch/create`

Создает несколько ордеров одним запросом.

Тело:

```json
{
  "orders": [
    {
      "marketId": "BTC_USDT"
    }
  ]
}
```

Каждый элемент обрабатывается независимо, итог приходит в `BatchCommandResponseDto`.

### `POST /spot/orders/batch/cancel`

Массовая отмена по списку `orderIds`.

### `POST /spot/orders/cancel-all`

Отменяет все активные ордера клиента, опционально по одному `marketId`.

## Методы чтения

### `GET /spot/orders/active`

Возвращает все активные ордера пользователя.

Полезные фильтры:

- `symbol`
- `orderId`
- `clientOrderId`
- `orderTypeFilter`

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

Ответ:

- `orders[]`
- `nextCursor`
- `hasMore`

### `GET /spot/executions/history`

История execution/fill событий.

Подходит для reconciliation и формирования trade ledger.

Ключевые query-поля:

- `symbol`
- `baseAsset`
- `orderId`
- `clientOrderId`
- `tradeId`
- `side`
- `dateFromMs`
- `dateToMs`
- `cursor`
- `limit`

### `GET /spot/trades/history`

Постраничная история исполненных trades/deals для аналитики и отчетности.

### `GET /spot/deals/history`

Order-centric представление исполнений. Удобно, когда нужно смотреть связку `tradeId + orderId`.

## Типичные ошибки и граничные случаи

- Неверный `marketId` или неподдерживаемый `orderType`.
- `price`/`quantity` не соответствуют precision рынка.
- Отмена несуществующего или уже закрытого ордера.
- Переиспользование `clientOrderId`, если на стороне бизнеса он должен быть уникальным.

## Что важно бизнес-аналитику

- REST-команда лишь инициирует операцию, а не всегда означает мгновенное исполнение.
- Источник правды по жизненному циклу ордера это сочетание REST history и private WebSocket `user`.
- `cancel-and-replace` нужно документировать как отдельный бизнес-флоу, а не как простую комбинацию двух методов.

## См. также

- [market.md](market.md)
- [../ws/private.md](../ws/private.md)
- [../flows/quickstart.md](../flows/quickstart.md)
