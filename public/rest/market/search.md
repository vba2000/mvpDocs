# GET `/market/search`

[← Каталог `/market`](../market.md)

**Назначение:** поиск пар для автодополнения.

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `query` | да | string | Строка поиска, напр. `BTC` |
| `type` | нет | string | Фильтр типа символа |
| `exchange` | нет | string | Фильтр биржи |
| `limit` | нет | int | Макс. число результатов |

**Ответ `result`:** `SearchSymbolsDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `result` | array | Список `SymbolSearchResultDto` |

`SymbolSearchResultDto`: `symbol`, `fullName`, `description`, `exchange`, `ticker`, `type`.
