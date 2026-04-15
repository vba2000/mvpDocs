# GET `/market/exchange-info`

[← Каталог `/market`](../market.md)

**Назначение:** список спотовых рынков в стиле Binance `exchangeInfo`: точность, статус, типы ордеров.

**Параметры:** нет.

**Ответ `result`:** `SpotExchangeInfoDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `markets` | array | `SpotMarketDto` |

`SpotMarketDto`: `symbol`, `status`, `baseAsset`, `quoteAsset`, `baseAssetPrecision`, `quoteAssetPrecision`, `orderTypes`.
