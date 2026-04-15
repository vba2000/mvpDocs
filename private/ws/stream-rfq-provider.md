# WebSocket: поток `rfq/provider`

## Имя потока

Каноническое имя из конфига стримов: обычно **`rfq/provider`** (совпадает с ключом `rfq-provider` → префикс path в настройках).

## Кому доступен

Пользователи с ролью **`MARKET_MAKER`**. Префикс попадает под приватные (`rfq/`).

## Полезная нагрузка

**`RfqEventProto`** — события жизненного цикла RFQ и котировок с точки зрения провайдера:

| Поле | Описание |
|------|----------|
| `event` | Тип события (строка) |
| `rfq_id`, `status` | Идентификатор и статус RFQ |
| `market_id`, `side`, `quantity` | Параметры заявки |
| `quotes` | Список `RfqQuoteProto` |
| `quote_id`, `client_quote_id`, `provider_user_id` | Уточнения по котировке |
| `price`, `reason`, `error` | Цена и диагностика |
| `timeout_ms`, `timestamp` | Тайминги |
| `trace_id` | Корреляция |

## См. также

- [Приватный REST: RFQ](../rest/rfq.md)
- [Сообщения клиента: firm/raw RFQ](messages/rfq-mm.md)
- [Обзор приватного WS](README.md)
