# Перевод и депозит — `/accounts/*`

[← Каталог Accounts](../accounts.md)

## POST `/accounts/transfer`

**Назначение:** внутренний перевод между счетами пользователя (напр. SPOT → другой тип).

**Тело:** `TransferRequestDto` (см. OpenAPI).

| Параметр query | Описание |
|----------------|----------|
| `userId` | OPERATOR |

**Ответ `result`:** `TransferResponseDto`.

---

## POST `/accounts/deposit`

**Назначение:** зачисление средств на счёт.

**Требования:** роль **OPERATOR**.

**Тело:** `DepositRequestDto`.

**Ответ `result`:** `CommandResponseDto`.
