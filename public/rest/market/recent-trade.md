# GET `/market/recent-trade`

[← Каталог `/market`](../market.md)

**Назначение:** последние публичные сделки (до 100).

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `symbol` | нет | string | Фильтр пары, напр. `BTC_USDT` |
| `limit` | нет | int | 1…100, по умолчанию 60 |

**Ответ `result`:** `RecentTradesDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `list` | array | `RecentTradeItemDto` |

Элемент: `execId`, `symbol`, `price`, `size`, `side`, `time`.
