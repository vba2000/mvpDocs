# Aggregate Price — `/aggregate-price/*`

[← Прочие приватные эндпоинты](../other.md)

Аутентификация обязательна.

## GET `/aggregate-price/latest`

**Назначение:** последние агрегированные цены по списку рынков.

| Параметр | Обяз. | Описание |
|----------|-------|----------|
| `markets` | да | Повторяющийся query или список ID, напр. `BTC_USDT` |

**Ответ `result`:** массив `AggregatePriceDto`

| Поле | Описание |
|------|----------|
| `marketId` | Рынок |
| `aggregatePrice` | Значение индекса |
| `change24` | Изменение за 24ч |
| `timestamp` | Метка времени (`Instant`) |

## GET `/aggregate-price/history`

**Назначение:** история агрегата по рынкам и интервалу.

| Параметр | Обяз. | Описание |
|----------|-------|----------|
| `markets` | да | Список рынков |
| `from`, `to` | да | Epoch **миллисекунды** |
| `interval` | да | `1min`, `10min`, `1h`, `1d`, `1w` |
| `limit` | нет | Макс. точек на рынок (до 1000) |

**Ответ `result`:** `List<AggregatePriceDto>`.

## См. также

- [Публичный WS: aggregate price](../../../public/ws/stream-aggregate-price.md)
