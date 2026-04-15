# WebSocket: агрегированная цена

## Имя потока

```
spot/aggregate_price
```

Глобальный поток: идентификатор конкретного рынка передаётся **внутри сообщения** (`market_id`).

## Кому доступен

Публичный поток.

## Полезная нагрузка

**`AggregatePriceProto`**:

| Поле | Описание |
|------|----------|
| `event` | Тип события |
| `market_id` | Рынок |
| `aggregate_price` | Значение индекса / агрегата |
| `change_24h` | Изменение за 24ч |
| `timestamp` | Метка времени |
| `trace_id` | Корреляция |

## См. также

- [Обзор публичного WS](README.md)
- [REST: aggregate-price](../../private/rest/other/aggregate-price.md)
