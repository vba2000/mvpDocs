# Public WebSocket

## Назначение

Public WebSocket доставляет потоковые рыночные данные без аутентификации. Он нужен для low-latency подписок, которые в REST пришлось бы постоянно polling-ить.

URL тестовой среды:

`wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public`

## Поддерживаемые streams

| Stream pattern | Назначение |
|----------------|------------|
| `spot/orderbook.<market_id>` | Стакан по рынку |
| `spot/trades.<market_id>` | Поток сделок |
| `spot/ticker.<market_id>` | Ticker обновления |
| `spot/candle.<market_id>.<timeframe>` | Поток свечей |
| `spot/aggregate_price` | Агрегированная цена |

`market_id` обычно имеет формат `BTC_USDT`.

## Client actions

В public сокете поддерживаются только:

- `subscribe`
- `unsubscribe`
- `pong`

### Пример JSON subscribe

```json
{
  "action": "subscribe",
  "streams": ["spot/orderbook.BTC_USDT", "spot/trades.BTC_USDT"]
}
```

### Пример JSON unsubscribe

```json
{
  "action": "unsubscribe",
  "streams": ["spot/trades.BTC_USDT"]
}
```

## Серверные события

Сервер использует envelope:

- `event=subscribed`
- `event=unsubscribed`
- `event=ping`
- `event=data`
- `event=error`

Для `event=data` ключевые поля:

- `stream`
- `data`

## Бизнес-смысл потоков

### `spot/orderbook.<market_id>`

Для построения depth, best bid/ask, мониторинга ликвидности и книг заявок.

### `spot/trades.<market_id>`

Для tape, recent trades, activity feed, trade analytics.

### `spot/ticker.<market_id>`

Для карточек рынка, summary widgets, 24h statistics.

### `spot/candle.<market_id>.<timeframe>`

Для real-time обновления графиков по таймфреймам.

### `spot/aggregate_price`

Для real-time reference/index price.

## Ограничения

- Public socket не принимает private streams.
- Сервер ограничивает количество подписок на одну сессию.
- Клиент обязан отвечать `pong` на heartbeat.

## Когда использовать public WS вместо REST

- Нужны постоянные обновления рынка;
- критична low-latency доставка;
- нужно избегать частого polling;
- нужен единый канал для нескольких потоков сразу.

## См. также

- [overview.md](overview.md)
- [message-format-json.md](message-format-json.md)
- [../rest/market.md](../rest/market.md)
