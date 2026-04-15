# Приватный REST: `/accounts/*`

Префикс **`/accounts`**. Все методы требуют аутентификации. Query **`userId`** (где указан) — только для **OPERATOR**; иначе игнорируется.

---

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

---

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

**Требования:** роль **OPERATOR** (в описании API).

**Тело:** `DepositRequestDto`.

**Ответ `result`:** `CommandResponseDto`.

---

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

**Ответ `result`:** `TransfersHistoryResponseDto` (тот же тип ответа в контракте).

---

## См. также

- [Spot](spot.md)
- [Обзор приватного REST](README.md)
