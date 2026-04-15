# WebSocket: сообщения от клиента

Все варианты описаны в **`WsClientMessage`** ([`protos/ws.proto`](../../protos/ws.proto)). Опционально на любом сообщении: **`trace_id`** для сквозной трассировки.

В **JSON**-режиме обычно используется поле **`action`** со значением, совпадающим с именем protobuf-поля в snake_case (`subscribe`, `create_order`, …).

---

## Подписка / отписка / heartbeat

### `SubscribeMessage` — `subscribe`

| Поле | Описание |
|------|----------|
| `streams` | Имена потоков, напр. `spot/trades.BTC_USDT` |
| `trace_id` | Опционально |

### `UnsubscribeMessage` — `unsubscribe`

| Поле | Описание |
|------|----------|
| `streams` | Имена отключаемых потоков |
| `trace_id` | Опционально |

### `PongMessage` — `pong`

Ответ на серверный `ping`; поле `trace_id` по желанию.

---

## RFQ и маркет-мейкинг

### `SubmitFirmQuoteMessage` — `submit_firm_quote`

Твёрдая котировка в ответ на запрос.

| Поле | Описание |
|------|----------|
| `rfq_id` | ID RFQ |
| `side` | `BID_WS` / `ASK_WS` |
| `price`, `quantity`, `min_quantity` | Decimal string |
| `ttl_ms` | TTL котировки |
| `client_quote_id` | Клиентский идентификатор |
| `market_id` | Пара (проверка баланса) |
| `trace_id` | Опционально |

### `SubmitRawPriceMessage` — `submit_raw_price`

Индикативные цены (стриминг).

| Поле | Описание |
|------|----------|
| `symbol` | Пара `BTC_USDT` |
| `bid`, `bid_size`, `bid_min_size` | Сторона bid |
| `ask`, `ask_size`, `ask_min_size` | Сторона ask |
| `ttl_ms` | TTL (часто default 5000 ms) |
| `bids_levels`, `asks_levels` | Опционально уровни стакана |
| `trace_id` | Опционально |

### `CreateRfqMessage` — `create_rfq`

Создание RFQ с клиента WS (taker).

| Поле | Описание |
|------|----------|
| `market_id` | Пара |
| `side` | `BUY` / `SELL` |
| `quantity`, `worst_price` | Decimal string |
| `ttl_ms` | Время жизни RFQ |
| `trace_id` | Опционально |

---

## Спотовая торговля

### `CreateOrderMessage` — `create_order`

| Поле | Описание |
|------|----------|
| `market_id` | Пара |
| `side` | `BUY` / `SELL` |
| `order_type` | `LIMIT`, `MARKET`, стоп-типы |
| `time_in_force` | `GTC`, `IOC`, `FOK`, `GTD` |
| `quantity`, `price`, `stop_price`, `quote_amount` | Decimal string |
| `post_only` | boolean |
| `account_type` | Обычно `SPOT` |
| `client_order_id` | Опционально |
| `expires_at_ms` | Для GTD |
| `trace_id` | Опционально |

### `CancelOrderMessage` — `cancel_order`

Указать **`user_order_id`** (клиентский) **или** **`order_id`** (системный).

### `BatchCreateOrderMessage` — `batch_create_order`

`orders[]` — список `CreateOrderMessage`.

### `BatchCancelOrderMessage` — `batch_cancel_order`

| Поле | Описание |
|------|----------|
| `order_ids` | Список системных ID |
| `account_type` | Обычно `SPOT` |
| `trace_id` | Опционально |

### `CancelAllOrdersMessage` — `cancel_all_orders`

| Поле | Описание |
|------|----------|
| `account_type` | Обычно `SPOT` |
| `market_id` | Пусто = все рынки |
| `trace_id` | Опционально |

---

## Ошибки

Сервер может вернуть событие ошибки с кодом из **`WsErrorCode`** (`INVALID_MESSAGE`, `AUTHENTICATION_REQUIRED`, …).

## См. также

- [Обзор приватного WS](README.md)
- [Публичный WS: подписки](../../public/ws/README.md)
