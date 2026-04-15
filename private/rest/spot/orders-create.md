# POST `/spot/orders/create` — подробно

[← Каталог Spot](../spot.md)

**Назначение:** создать спотовый ордер. Тело: JSON **`OrderCreateRequest`** (поля в **camelCase**). Валидация на Gateway: `HotBalanceOrderValidation` в репозитории `cex`.

**Ответ `result`:** `CommandResponseDto` — `success`, `message`, `orderId?`, `clientOrderId?`.

## Поля запроса (общая таблица)

| Поле | Тип | Описание |
|------|-----|----------|
| `marketId` | string | Идентификатор рынка, напр. `BTC_USDT` (должен совпадать с конфигом рынков Gateway). |
| `baseAssetId` | string | Базовый актив пары, напр. `BTC`. |
| `quoteAssetId` | string | Котируемый актив, напр. `USDT`. |
| `side` | enum | См. [OrderSide](#orderside). |
| `orderType` | enum | См. [OrderType](#ordertype) — от типа зависят обязательные `price` / `stopPrice` / `quoteAmount`. |
| `timeInForce` | enum | См. [TimeInForce](#timeinforce). По умолчанию в DTO: **`GTC`**. |
| `quantity` | string | Объём в **базовом** активе (decimal, &gt; 0). Для **MARKET BUY** дополнительно нужен `quoteAmount` (см. ниже). |
| `quoteAmount` | string? | Сумма в **котируемом** активе для **MARKET BUY** (обязательно). Для остальных сценариев опционально. |
| `price` | string? | Лимитная цена (quote за единицу base). Обязательна для типов, где указано в таблице [OrderType](#ordertype). |
| `stopPrice` | string? | Цена активации стоп-ордера. Обязательна для стоп-типов. |
| `postOnly` | boolean | Только ордер-мейкер (не забирает ликвидность сразу). **Только** при `orderType = LIMIT` и `timeInForce = GTC`. |
| `accountType` | enum | См. [AccountType](#accounttype). Для этого эндпоинта допустимо **только `SPOT`**. |
| `marketSeq` | long? | Опционально: ожидаемая последовательность/версия рынка для согласованности клиента с матчером. |
| `expiresAt` | string? | ISO-8601 instant. **Обязательно**, если `timeInForce = GTD`: время в будущем. |
| `timestamp` | string? | Опциональная метка времени клиента (передаётся в downstream по контракту). |
| `clientOrderId` | string? | Ваш идемпотентный/внешний ID ордера; возвращается в ответе и в стримах. |

Числовые строки нормализуются по точности рынка (`price` / `quantity` / стопы / `quoteAmount`).

---

## OrderSide

Сторона сделки **с точки зрения тейкера** (вашего ордера).

| Значение | JSON | Назначение |
|----------|------|------------|
| `BUY` | `"BUY"` | Купить **base** за **quote** (на балансе резервируется quote согласно типу ордера). |
| `SELL` | `"SELL"` | Продать **base** за **quote** (резервируется `quantity` в base). |

На исполнение и книгу заявок влияет в паре с `orderType`, `price` и `timeInForce`.

---

## OrderType

Определяет механику размещения и **какие поля обязательны** (проверка Gateway).

| Значение | JSON | Обязательные поля | Смысл |
|----------|------|-------------------|--------|
| `LIMIT` | `"LIMIT"` | `price` | Лимитный ордер в книгу по указанной цене (при необходимости с `postOnly`, `timeInForce`). |
| `MARKET` | `"MARKET"` | **BUY:** `quoteAmount`; **SELL:** достаточно `quantity` | Рыночный: исполнение против книги по доступной ликвидности. Покупка по сумме в quote — через `quoteAmount`. |
| `STOP_LOSS` | `"STOP_LOSS"` | `stopPrice` | Стоп-лосс: активация при достижении стоп-цены (дальнейшая логика — как стоп в матчере/account). |
| `STOP_LIMIT` | `"STOP_LIMIT"` | `stopPrice` и `price` | Сначала стоп по `stopPrice`, затем лимит исполнения по `price`. |
| `TAKE_PROFIT` | `"TAKE_PROFIT"` | `stopPrice` | Тейк-профит (триггер по `stopPrice`). |
| `TAKE_PROFIT_LIMIT` | `"TAKE_PROFIT_LIMIT"` | `stopPrice` и `price` | Тейк-профит с лимитной ценой исполнения `price`. |

Для **MARKET** при **SELL** поле `quoteAmount` в валидации не требуется — задаётся объём в base через `quantity`.

---

## TimeInForce

Срок действия и допустимость частичного исполнения передаются в матчер через gRPC.

| Значение | JSON | Назначение |
|----------|------|------------|
| `GTC` | `"GTC"` | **Good-Till-Cancelled** — ордер живёт в книге, пока не исполнится или не отменён. Значение по умолчанию. Обязательно для `postOnly = true`. |
| `IOC` | `"IOC"` | **Immediate-Or-Cancel** — исполнить доступное сразу, остаток **отменить** (не остаётся в книге как активный объём). |
| `FOK` | `"FOK"` | **Fill-Or-Kill** — исполнить **весь** заявленный объём целиком или **отклонить** заявку. |
| `AON` | `"AON"` | **All-Or-Nothing** — на стороне матчера ордер не исполняется частично как мейкер: либо полный объём, либо ожидание/отказ по правилам книги (см. `OrderBookAonProcessor` в `spot-balance-node`). |
| `GTD` | `"GTD"` | **Good-Till-Date** — ордер действителен до момента **`expiresAt`** (ISO-8601, строго в будущем). Без валидного `expiresAt` запрос отклоняется. |

---

## AccountType

Тип счёта в доменной модели; в protobuf уходит как тип аккаунта.

| Значение | JSON | Для `POST /spot/orders/create` |
|----------|------|--------------------------------|
| `SPOT` | `"SPOT"` | **Единственное допустимое значение.** Любой другой тип → ошибка `ACCOUNT_TYPE_UNSUPPORTED`. |
| `RFQ` | `"RFQ"` | Не для этого REST-метода. |
| `FUTURES` | `"FUTURES"` | Не для этого REST-метода. |
| `FUNDING` | `"FUNDING"` | Не для этого REST-метода. |

Поле можно не передавать: в DTO по умолчанию **`SPOT`**.

---

## Дополнительные правила

- **`postOnly`:** разрешено только при **`LIMIT`** и **`timeInForce = GTC`**. Иначе ошибки `POST_ONLY_ONLY_LIMIT` / `POST_ONLY_REQUIRES_GTC`.
- **Положительные decimal:** `quantity`, при наличии — `price`, `quoteAmount`, `stopPrice` должны быть &gt; 0 (корректные decimal-строки).
- Сверка деталей исполнения стопов и рыночных заявок с матчером — по OpenAPI и логам; таблицы выше отражают **контракт Gateway + обязательные поля**.

Пример тела и подписи запроса: [auth/rest-api-key.md](../../../auth/rest-api-key.md).

## См. также

- [Каталог Spot](../spot.md)
- [Приватный WS: торговые сообщения](../../ws/client-messages.md)
