# История переводов и депозитов — `/accounts/*`

[← Каталог Accounts](../accounts.md)

## GET `/accounts/transfers/history`

**Назначение:** история внутренних переводов.

| Параметр | Описание |
|----------|----------|
| `status`, `assetId` | Фильтры |
| `dateFromMs`, `dateToMs` | Интервал |
| `userId` | OPERATOR |
| `limit`, `offset` | Пагинация |

**Ответ `result`:** `TransfersHistoryResponseDto`.

---

## GET `/accounts/deposits/history`

**Назначение:** история депозитов.

Параметры аналогичны transfers history.

**Ответ `result`:** `TransfersHistoryResponseDto` (тот же тип в контракте).
