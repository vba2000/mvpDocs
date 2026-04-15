# WS: RFQ и маркет-мейкинг

[← Сообщения клиента](../client-messages.md)

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
