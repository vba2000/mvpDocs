# RFQ: провайдер и маркет-мейкер

[← Каталог RFQ](../rfq.md)

Методы с ролью **MARKET_MAKER** помечены; без неё — **403**.

## POST `/rfq/providers/quote`

**Назначение:** отправить котировку провайдера ликвидности.

**Тело:** `RfqQuoteRequest`.

**Ответ `result`:** `QuoteResponseDto`.

---

## POST `/rfq/quote/indicative`

**Назначение:** индикативная (необязывающая) котировка.

**Роль:** **MARKET_MAKER**.

**Тело:** `SubmitIndicativeQuoteRequest`.

**Ответ `result`:** `QuoteAckDto`.

---

## POST `/rfq/quote/firm`

**Назначение:** твёрдая (обязывающая) котировка.

**Роль:** **MARKET_MAKER**.

**Тело:** `SubmitFirmQuoteRequest`.

**Ответ `result`:** `FirmQuoteResponseDto`.

---

## GET `/rfq/provider/quotes`

**Назначение:** список котировок текущего маркет-мейкера.

**Роль:** **MARKET_MAKER**.

| Параметр | Описание |
|----------|----------|
| `status` | Статус котировки |
| `rfqStatus` | Статус RFQ |

Плюс `Pageable`.

**Ответ `result`:** `ProviderQuoteListResponseDto`.
