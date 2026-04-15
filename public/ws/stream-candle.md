# WebSocket: поток свечей

## Имя потока

```
spot/candle.{market_id}.{timeframe}
```

Пример: `spot/candle.BTC_USDT.1m`

Компонент **`timeframe`** — строка таймфрейма (как в конфиге рынка / графика, напр. `1m`, `5m`, `1h`).

## Кому доступен

Публичный поток.

## Полезная нагрузка

**`CandleProto`**:

| Поле | Описание |
|------|----------|
| `event` | Тип события |
| `market_id` | Рынок |
| `timeframe` | Интервал свечи |
| `open_time`, `close_time` | Границы периода |
| `open`, `high`, `low`, `close` | OHLC |
| `volume`, `quote_volume` | Объёмы |
| `trades` | Число сделок в свече |
| `closed` | Закрыта ли свеча |
| `trace_id` | Корреляция |

Обновления могут батчиться с интервалом `candle-flush-ms`.

## См. также

- [Обзор публичного WS](README.md)
- [REST: история свечей](../rest/market.md)
