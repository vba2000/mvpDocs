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

Важная деталь по payload:

- orderbook может приходить как snapshot, single delta или batched deltas;
- trades в live feed обычно приходят батчами `TradeBatchProto`, а не одиночными сделками;
- в orderbook payload используются не только `market_seq`, но и `matcher_seq`.

## Бизнес-смысл потоков

### `spot/orderbook.<market_id>`

Для построения depth, best bid/ask, мониторинга ликвидности и книг заявок.

Типовые payloads:

- `OrderbookSnapshotProto`
- `OrderbookDeltaProto`
- `OrderbookDeltaBatchProto`

Практически для клиента это означает, что после snapshot нужно уметь применять последующие deltas и batched deltas.

Recommended pattern:

1. получить initial snapshot;
2. сохранить текущий `seq_num` и `matcher_seq`;
3. последовательно применять `delta` и `delta_batch`;
4. при разрыве соединения или нарушении последовательности заново брать REST snapshot.

### `spot/trades.<market_id>`

Для tape, recent trades, activity feed, trade analytics.

В runtime-потоке сделки чаще доставляются батчами, поэтому клиент должен уметь разбирать массив `trades[]`, а не только один trade event.

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
- Public socket не предназначен для пользовательских событий, депозитных уведомлений и торговых команд.

## `matcher_seq` и `seq_num`

- `seq_num` помогает клиенту контролировать порядок событий внутри stream feed;
- `matcher_seq` отражает последовательность matcher-side книги заявок;
- для orderbook recovery безопаснее ориентироваться не на "догоняющие" эвристики, а на полный resync после потери последовательности.

## Recovery после разрыва

Если клиент потерял public WS:

1. открыть новое соединение;
2. заново подписаться на нужные streams;
3. для `spot/orderbook.*` взять `GET /spot/orderbook`;
4. только после этого возобновить применение live deltas;
5. recent trades/ticker при необходимости добрать через публичные REST endpoints.

## Когда использовать public WS вместо REST

- Нужны постоянные обновления рынка;
- критична low-latency доставка;
- нужно избегать частого polling;
- нужен единый канал для нескольких потоков сразу.

## См. также

- [overview.md](overview.md)
- [message-format-json.md](message-format-json.md)
- [../rest/market.md](../rest/market.md)
