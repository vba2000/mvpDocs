# WebSocket: поток тикера

## Имя потока

```
spot/ticker.{market_id}
```

Пример: `spot/ticker.BTC_USDT`

## Кому доступен

Публичный поток.

## Полезная нагрузка

**`TickerProto`**:

| Поле | Описание |
|------|----------|
| `event` | Тип события |
| `market_id` | Рынок |
| `last_price` | Последняя цена (optional) |
| `best_bid`, `best_ask` | Лучший bid/ask (optional) |
| `best_bid_size`, `best_ask_size` | Объёмы на лучших уровнях (optional) |
| `volume_24h`, `turnover_24h` | Объём и оборот за 24ч |
| `high_24h`, `low_24h` | Экстремумы |
| `change_24h`, `prev_price_24h` | Изменение и предыдущая цена |
| `timestamp` | Метка времени |
| `trace_id` | Корреляция |

На стороне сервера возможен интервал широковещания тикера (например 1 минута) — уточняйте по конфигу `ticker-broadcast` в `application.yml`.

## См. также

- [Обзор публичного WS](README.md)
- [Свечи](stream-candle.md)
