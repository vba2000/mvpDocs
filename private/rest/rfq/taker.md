# RFQ: taker и общие методы

[← Каталог RFQ](../rfq.md)

Операции со стороны запрашивающего котировку и вспомогательные GET.

## POST `/rfq/create`

**Назначение:** создать запрос котировки (taker).

**Тело:** `RfqCreateRequest`.

**Ответ `result`:** `RfqResponseDto`.

---

## GET `/rfq/{rfqId}`

**Назначение:** получить RFQ по ID.

| Параметр | Описание |
|----------|----------|
| `userId` query | OPERATOR |

**Ответ `result`:** `RfqResponseDto`.

---

## POST `/rfq/{rfqId}/cancel`

**Назначение:** отменить активный RFQ.

| Параметр | Описание |
|----------|----------|
| `userId` query | OPERATOR |

**Ответ `result`:** `RfqResponseDto`.

---

## GET `/rfq/list`

**Назначение:** список RFQ с пагинацией Spring.

| Параметр | Описание |
|----------|----------|
| `status` | Фильтр статуса |
| `userId` | OPERATOR: конкретный пользователь; без — все (OPERATOR) |

**Ответ `result`:** `RfqListResponseDto`.

---

## GET `/rfq/indicative-price`

**Назначение:** агрегированная индикативная цена для рынка, стороны и объёма.

| Параметр | Обяз. | Описание |
|----------|-------|----------|
| `marketId` | да | Напр. `BTC_USDT` |
| `side` | да | `BUY` или `SELL` |
| `quantity` | да | Количество (decimal string) |

**Ответ `result`:** `IndicativePriceResponseDto`.
