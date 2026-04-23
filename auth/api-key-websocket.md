# API key для private WebSocket

## Назначение

Private WebSocket нужен для получения пользовательских событий и для отправки торговых команд в real time. Внешний клиент может аутентифицироваться по `API key`, и это основной интеграционный сценарий для non-browser клиентов.

## URL тестовой среды

- Public WebSocket: `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public`
- Private WebSocket: `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private`

Приложение поддерживает также служебный путь `/ws`, но во внешней интеграции нужно использовать явные public/private URL.

## Формат handshake

Для API key используются те же заголовки, что и в REST:

| Заголовок | Обязателен | Назначение |
|-----------|------------|------------|
| `X-API-Key` | да | Идентификатор ключа |
| `X-Timestamp` | да | Unix time в миллисекундах |
| `X-Signature` | да | Base64 HMAC-SHA256 |

## Критически важное отличие от REST

В WebSocket строка подписи не зависит от внешнего URL и не повторяет реальный путь `/ws/private`.

Сервер ожидает фиксированную строку:

```text
message = "{timestamp}WEBSOCKET/wsWS_CONNECT"
signature = Base64(HMAC_SHA256(secret, UTF8(message)))
```

То есть в подпись всегда входят:

- метод: `WEBSOCKET`
- путь: `/ws`
- body: `WS_CONNECT`

Даже если фактическое подключение идет на `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private?format=json`, в HMAC по-прежнему участвует только фиксированная строка выше.

## Пошаговый flow private WebSocket

1. Клиент выбирает URL private сокета.
2. Клиент выбирает формат транспорта:
   - `?format=binary`
   - `?format=json`
3. Клиент формирует `timestamp`.
4. Клиент вычисляет HMAC для строки `"{timestamp}WEBSOCKET/wsWS_CONNECT"`.
5. Клиент открывает WebSocket c тремя заголовками.
6. Сервер валидирует ключ, подпись, timestamp, статус ключа и IP.
7. После успешной аутентификации сервер проверяет право на поток `user`.
8. Если доступ разрешен, сервер автоматически подписывает сессию на `user`.
9. После этого клиент может:
   - принимать `user` events;
   - подписываться на дополнительные private streams;
   - отправлять private commands.

## Требования по правам

### Обязательный минимум

Чтобы private WebSocket вообще принял соединение, клиенту нужно право на поток `user`, а значит право `READ`.

### Что проверяется дальше

| Stream / действие | Требование |
|-------------------|------------|
| `user` | `READ` |
| Торговые команды | `TRADE` |
| `provider.*`, `rfq/provider` | роль `MARKET_MAKER` |

Market maker-сценарии не входят в эту пользовательскую документацию, но ограничение важно учитывать.

## Что происходит при ошибке

Соединение может быть закрыто еще на этапе handshake, если:

- отсутствуют обязательные заголовки;
- подпись не совпала;
- ключ неактивен;
- timestamp просрочен;
- у ключа нет прав на private streams;
- выбран неподдерживаемый `format`.

В JSON-режиме сервер использует стандартный envelope c `event=error`.

## Выбор формата

Поддерживаются:

- `format=binary` — protobuf frames;
- `format=json` — text frames с JSON envelope.

Если параметр `format` не передан, сервер по умолчанию работает в `binary`.

## Что важно разработчику

- Не включайте query string `format=json` в строку HMAC для WebSocket.
- Не подставляйте `/ws/private` в строку подписи.
- Если клиенту нужен только market data, используйте public WebSocket без авторизации.
- Для production tracing полезно логировать `timestamp`, canonical string и request id на стороне клиента, но никогда не логировать `secret`.

## См. также

- [signing-examples.md](signing-examples.md)
- [../ws/private.md](../ws/private.md)
- [../ws/message-format-json.md](../ws/message-format-json.md)
