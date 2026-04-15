# WebSocket: поток публичных сделок

## Имя потока

```
spot/trades.{market_id}
```

Пример: `spot/trades.BTC_USDT`

## Кому доступен

Публичный поток.

## Полезная нагрузка

**`TradeBatchProto`** (`protos/ws.proto`):

| Поле | Описание |
|------|----------|
| `event` | Например `trade` |
| `market_id` | Рынок |
| `trades` | Массив `TradeProto` |
| `trace_id` | Корреляция |

**Элемент `TradeProto`:**

| Поле | Описание |
|------|----------|
| `trade_id` | Идентификатор сделки |
| `price`, `quantity` | Цена и объём (строки decimal) |
| `side` | Сторона сделки |
| `timestamp` | Время |
| `market_id`, `event`, `trace_id` | Метаданные |

Сделки накапливаются в батч за интервал `trades-flush-ms`.

## См. также

- [Обзор публичного WS](README.md)
- [Тикер](stream-ticker.md)
