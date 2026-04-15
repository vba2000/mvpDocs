# GET `/market/symbols`

[← Каталог `/market`](../market.md)

**Назначение:** метаданные торговой пары для графика.

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `symbol` | да | string | Символ, напр. `BTC/USDT` |

**Ответ `result`:** `SymbolInfoDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `name`, `ticker`, `description`, `type`, `session`, `timezone`, `exchange` | string | Идентификация и отображение |
| `minmov`, `pricescale` | int | Шаг цены (UDF) |
| `hasIntraday`, `hasWeeklyAndMonthly` | boolean | Доступность интервалов |
| `supportedResolutions` | string[] | Разрешения для символа |
| `volumePrecision` | int | Точность объёма |
| `dataStatus` | string | Статус данных |
