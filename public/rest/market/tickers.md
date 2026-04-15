# GET `/market/tickers`

[← Каталог `/market`](../market.md)

**Назначение:** агрегированная 24h-статистика по спотовым рынкам (кэшируется).

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `category` | нет | string | Категория, напр. `spot` |
| `symbol` | нет | string | Одна пара, напр. `BTC_USDT` |
| `baseCoin` | нет | string | Фильтр базового актива, напр. `USDT` |

**Ответ `result`:** `TickersResponseDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `category` | string | Категория выборки |
| `list` | array | `TickerItemDto` |

`TickerItemDto`: `symbol`, `bid1Price`, `bid1Size`, `ask1Price`, `ask1Size`, `lastPrice`, `prevPrice24h`, `price24hPcnt`, `highPrice24h`, `lowPrice24h`, `turnover24h`, `volume24h`.
