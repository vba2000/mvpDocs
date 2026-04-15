# Приватный REST: `/accounts/*`

Префикс **`/accounts`**. Аутентификация обязательна. Query **`userId`** (где указан) — только для **OPERATOR**; иначе игнорируется.

## Каталог

| Тема | Документ |
|------|----------|
| Баланс, fee tier | [accounts/balance-fee.md](accounts/balance-fee.md) |
| Перевод, депозит (OPERATOR) | [accounts/transfer-deposit.md](accounts/transfer-deposit.md) |
| История переводов и депозитов | [accounts/history.md](accounts/history.md) |

## См. также

- [Spot](spot.md)
- [Обзор приватного REST](README.md)
