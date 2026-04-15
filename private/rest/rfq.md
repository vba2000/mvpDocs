# Приватный REST: `/rfq/*`

Префикс **`/rfq`**. Базовые операции RFQ доступны **всем аутентифицированным** пользователям. Методы котировок **MARKET_MAKER** помечены ниже.

---

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

---

## POST `/rfq/providers/quote`

**Назначение:** отправить котировку провайдера ликвидности.

**Тело:** `RfqQuoteRequest`.

**Ответ `result`:** `QuoteResponseDto`.

---

## POST `/rfq/quote/indicative`

**Назначение:** индикативная (необязывающая) котировка.

**Роль:** **MARKET_MAKER** (иначе 403).

**Тело:** `SubmitIndicativeQuoteRequest`.

**Ответ `result`:** `QuoteAckDto`.

---

## POST `/rfq/quote/firm`

**Назначение:** твёрдая (обязывающая) котировка.

**Роль:** **MARKET_MAKER** (иначе 403).

**Тело:** `SubmitFirmQuoteRequest`.

**Ответ `result`:** `FirmQuoteResponseDto`.

---

## GET `/rfq/provider/quotes`

**Назначение:** список котировок текущего маркет-мейкера.

**Роль:** **MARKET_MAKER** (иначе 403).

| Параметр | Описание |
|----------|----------|
| `status` | Статус котировки |
| `rfqStatus` | Статус RFQ |

Плюс `Pageable`.

**Ответ `result`:** `ProviderQuoteListResponseDto`.

---

## См. также

- [Приватный WS: RFQ](../ws/stream-rfq-provider.md)
- [Обзор приватного REST](README.md)
