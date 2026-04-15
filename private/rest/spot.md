# Приватный REST: `/spot/*`

Префикс **`/spot`**. Аутентификация обязательна. Для ряда GET допускается query **`userId`** только у **OPERATOR** (см. описание каждого метода).

Обёртка ответа: стандартная (`result` содержит DTO ниже).

---

## POST `/spot/orders/create`

**Назначение:** создать спотовый ордер.

**Тело:** `OrderCreateRequest`

| Поле | Тип | Описание |
|------|-----|----------|
| `marketId` | string | Рынок, напр. `BTC_USDT` |
| `baseAssetId`, `quoteAssetId` | string | Активы пары |
| `side` | enum | `BUY` / `SELL` |
| `orderType` | enum | `LIMIT`, `MARKET`, стоп-типы и т.д. |
| `timeInForce` | enum | `GTC`, `IOC`, `FOK`, `GTD` (по умолчанию GTC) |
| `quantity` | string | Количество (decimal) |
| `quoteAmount` | string? | Для MARKET BUY |
| `price` | string? | Цена лимита |
| `stopPrice` | string? | Стоп-цена |
| `postOnly` | boolean | Только мейкер |
| `accountType` | enum | Обычно `SPOT` |
| `marketSeq` | long? | Опционально для согласованности |
| `expiresAt`, `timestamp` | string? | ISO/строковые метки по контракту API |
| `clientOrderId` | string? | Клиентский идентификатор |

**Ответ `result`:** `CommandResponseDto` — `success`, `message`, `orderId?`, `clientOrderId?`.

---

## POST `/spot/orders/cancel`

**Назначение:** отменить активный ордер.

| Параметр query | Обяз. | Описание |
|-----------------|-------|----------|
| `orderId` | одно из двух | Системный ID ордера |
| `clientOrderId` | одно из двух | Клиентский ID |

**Ответ `result`:** `CommandResponseDto`.

---

## POST `/spot/orders/cancel-and-replace`

**Назначение:** атомарно отменить ордер и выставить новый.

**Тело:** `CancelAndReplaceRequest` (см. OpenAPI).

**Ответ `result`:** `CommandResponseDto`.

---

## POST `/spot/orders/batch/create`

**Назначение:** пакетное создание ордеров.

**Тело:** `BatchOrderCreateRequest`.

**Ответ `result`:** `BatchCommandResponseDto` — `results[]` (`CommandResponseDto`), `totalSuccess`, `totalFailed`.

---

## POST `/spot/orders/batch/cancel`

**Назначение:** пакетная отмена по списку ID.

**Тело:** `BatchOrderCancelRequest`.

**Ответ `result`:** `BatchCommandResponseDto`.

---

## POST `/spot/orders/cancel-all`

**Назначение:** отменить все активные ордера пользователя.

**Тело:** `CancelAllOrdersRequest` (фильтр по рынку — см. OpenAPI).

**Ответ `result`:** `BatchCommandResponseDto`.

---

## GET `/spot/orders/active`

**Назначение:** активные лимитные и стоп-ордера.

| Параметр | Обяз. | Описание |
|----------|-------|----------|
| `symbol` | нет | Фильтр рынка |
| `orderId`, `clientOrderId` | нет | Точечный фильтр |
| `orderTypeFilter` | нет | `LIMIT`, `MARKET`, … |
| `userId` | нет | OPERATOR: чужой пользователь |

**Ответ `result`:** `SpotActiveOrdersResponseDto` (см. OpenAPI).

---

## GET `/spot/orders/history`

**Назначение:** история ордеров с курсором.

| Параметр | Описание |
|----------|----------|
| `symbol`, `orderId`, `clientOrderId`, `orderTypeFilter`, `side` | Фильтры |
| `dateFromMs`, `dateToMs` | Интервал (epoch ms) |
| `includeTrades` | boolean, по умолчанию false |
| `limit` | размер страницы, макс. 100, default 100 |
| `cursor` | курсор |
| `userId` | OPERATOR |

**Ответ `result`:** `SpotOrdersHistoryResponseDto`.

---

## GET `/spot/trades/history`

**Назначение:** исполненные сделки пользователя (пагинация Spring `Pageable`).

| Параметр | Описание |
|----------|----------|
| `marketId`, `marketQuery`, `direction`, `status` | Фильтры |
| `dateFromMs`, `dateToMs` | Интервал |
| `userId` | OPERATOR; без него — все пользователи для OPERATOR |

Плюс стандартные `page`, `size`, `sort` (см. OpenAPI).

**Ответ `result`:** `TradesHistoryResponseDto`.

---

## GET `/spot/executions/history`

**Назначение:** история исполнений (fills) с курсором.

| Параметр | Описание |
|----------|----------|
| `symbol`, `baseAsset`, `orderId`, `clientOrderId`, `tradeId`, `side` | Фильтры |
| `dateFromMs`, `dateToMs` | Интервал |
| `cursor`, `limit` (макс. 100, default 20) | Пагинация |
| `userId` | OPERATOR |

**Ответ `result`:** `ExecutionsHistoryResponseDto`.

---

## GET `/spot/deals/history`

**Назначение:** история сделок матчера «в разрезе ордера» (order-centric).

Те же фильтры, что у trades history в целом; `Pageable`.

**Ответ `result`:** `DealsHistoryResponseDto`.

---

## См. также

- [Счета](accounts.md)
- [Приватный WS: торговые сообщения](../ws/client-messages.md)
- [Обзор приватного REST](README.md)
