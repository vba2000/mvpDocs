# Баланс и комиссия — `/accounts/*`

[← Каталог Accounts](../accounts.md)

## GET `/accounts/balance`

**Назначение:** балансы пользователя по активам и типам счёта.

| Параметр | Описание |
|----------|----------|
| `userId` | OPERATOR: целевой пользователь |

**Ответ `result`:** `BalancesResponseDto`

| Поле | Описание |
|------|----------|
| `balances` | Список `BalanceItemDto` |

`BalanceItemDto`: `assetId`, `available`, `reserved`, `accountType`, `version`.

---

## GET `/accounts/fee-tier`

**Назначение:** уровень комиссии maker/taker (basis points).

| Параметр | Описание |
|----------|----------|
| `userId` | OPERATOR |

**Ответ `result`:** `FeeTierResponseDto` — `makerFeeBps`, `takerFeeBps`.
