# Каталог ошибок

## Назначение

Этот раздел собирает типовые ошибки внешней интеграции: ошибки HMAC, бизнес-валидации REST и коды ошибок WebSocket.

## REST: типовые ошибки аутентификации

### Признаки

- HTTP `401`
- REST response wrapper с ненулевым `code`
- детали причины могут быть в `extInfo`

### Частые причины

| Причина | Что проверить |
|---------|---------------|
| Неверный `secret` | Сравнить HMAC и canonical string |
| Неверный путь в подписи | Использовать путь после хоста, включая `/api/v1/gateway` и query |
| Подписан другой body | Убедиться, что подписанная JSON-строка совпадает с HTTP body побайтно |
| Просроченный `timestamp` | Синхронизировать часы через `GET /market/time` |
| Ключ неактивен или истек | Проверить статус и срок жизни API key |
| IP не в whitelist | Проверить настройки ключа |

## REST: типовые ошибки торговой валидации

Для `spot` чаще всего встречаются:

- неподдерживаемый `orderType`
- неподдерживаемый `timeInForce`
- некорректный `price`, `quantity`, `stopPrice`, `quoteAmount`
- нарушение `tickSize`, `lotSize`, `minQty`, `maxQty`, `minNotional`
- недопустимый `postOnly`
- отсутствие `quoteAmount` в `BUY + FOK` market-сценарии
- `expiresAt` отсутствует или не лежит в будущем для `GTD`
- неверный `accountType`

Подробности: [rest/spot.md](rest/spot.md)

## REST: ошибки transfer / deposit

Типовые проблемы:

- неподдерживаемое направление перевода;
- недостаточный баланс;
- дубликат или отсутствие `idempotencyKey` в retry-sensitive flow;
- лимит на депозитные адреса;
- неподдерживаемая сеть или актив;
- депозитная операция в неподходящем статусе для бизнес-ожидания клиента.

## WebSocket error codes

Runtime WebSocket использует следующие коды:

| Код | Имя | Значение |
|-----|-----|----------|
| `4001` | `SUBSCRIPTION_LIMIT_EXCEEDED` | Превышен лимит подписок |
| `4002` | `INVALID_STREAM` | Указан неподдерживаемый stream |
| `4003` | `STREAM_NOT_SUBSCRIBED` | Попытка отписки/работы с неподписанным stream |
| `4004` | `INVALID_MESSAGE` | Некорректное сообщение клиента |
| `4005` | `AUTHENTICATION_REQUIRED` | Нужна аутентификация или нет прав |
| `4006` | `RATE_LIMIT_EXCEEDED` | Превышен connection/subscription limit |
| `5000` | `INTERNAL_ERROR` | Внутренняя ошибка сервера |

В JSON-режиме эти коды приходят строками в поле `code`.

## Практика retry и recovery

### REST

- Для retry-safe сценариев использовать `clientOrderId` и `idempotencyKey`.
- При неясном результате команды не повторять blindly, пока не проверен `GET /spot/orders/{orderId}` или history endpoint.

### WebSocket

- При разрыве private WS переподключить соединение и пересобрать состояние через REST snapshots/history.
- Для public orderbook после разрыва безопаснее заново взять snapshot и только потом возобновлять stream consumption.

## См. также

- [concepts.md](concepts.md)
- [auth/api-key-rest.md](auth/api-key-rest.md)
- [ws/message-format-json.md](ws/message-format-json.md)
