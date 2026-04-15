# WebSocket: поток стакана

## Имя потока

```
spot/orderbook.{market_id}
```

Пример: `spot/orderbook.BTC_USDT`

## Кому доступен

Публичный поток; анонимное подключение разрешено на публичном WS URL.

## Полезная нагрузка в `DataEvent.data`

Сообщения protobuf (см. [`protos/ws.proto`](../../protos/ws.proto)):

1. **`OrderbookSnapshotProto`** — полный снимок стакана при подписке или восстановлении.
2. **`OrderbookDeltaBatchProto`** — батч дельт за интервал флаша; внутри агрегированные уровни bid/ask.

### OrderbookSnapshotProto (ключевые поля)

| Поле | Описание |
|------|----------|
| `event` | Тип события |
| `market_id` | Рынок |
| `bids`, `asks` | Уровни `PriceLevelProto` (price, size) |
| `market_seq` | Последовательность матчера |
| `depth` | Глубина снимка |
| `seq_num` | Для снимка обычно 0 |
| `timestamp_ms` | Время отправки шлюзом |
| `trace_id` | Корреляция |

### OrderbookDeltaBatchProto

| Поле | Описание |
|------|----------|
| `event` | Например `orderbook_delta` |
| `market_id` | Рынок |
| `bids`, `asks` | Изменённые уровни |
| `market_seq` | Последовательность матчера |
| `seq_num` | Счётчик в рамках сессии потока (с 1) |
| `timestamp_ms` | Время отправки |

## Рекомендации клиенту

- Применять снимок, затем накладывать дельты по `market_seq` / `seq_num` согласно правилам вашего клиента.
- Учитывать батчинг и throttle в конфиге шлюза.

## См. также

- [Обзор публичного WS](README.md)
- [Сделки](stream-trades.md)
