# WS: спотовая торговля

[← Сообщения клиента](../client-messages.md)

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

## См. также

- Подробные enum и правила REST: [POST /spot/orders/create](../../rest/spot/orders-create.md)
