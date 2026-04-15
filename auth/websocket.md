# WebSocket: аутентификация при рукопожатии

## URL подключения

Зависят от деплоя. В конфиге приложения задаются:

- **`service-type`**: `unified` | `public` | `private`
- Пути: по умолчанию корень `/ws`, публичный `/ws/public`, приватный `/ws/private`

Пример dev (за прокси):

- Публичный: `wss://cex-dev.web3tech.ru/api/v1/ws-gw/ws/public`
- Приватный: `wss://cex-dev.web3tech.ru/api/v1/ws-private/ws/private`

При **unified** один процесс обслуживает несколько путей; при **public** / **private** — только соответствующий режим (у публичного сервиса подписки на приватные потоки отклоняются).

## Формат канала

Query-параметр **`format`** (имя из конфига `format-query-param`): обычно `binary` (protobuf) или `json`. Значение по умолчанию — из `default-transport-format`.

## Способ 1: API-ключ + HMAC

Заголовки те же, что у REST:

- `X-API-Key`, `X-Signature`, `X-Timestamp`

**Критично:** строка для HMAC **не** совпадает с REST. В коде зашиты константы:

| Компонент | Значение |
|-----------|----------|
| Метод (в строке подписи) | `WEBSOCKET` |
| Путь (в строке подписи) | `/ws` |
| Тело (в строке подписи) | `WS_CONNECT` |

То есть:

```
message = "{timestamp}WEBSOCKET/wsWS_CONNECT"
signature = Base64( HMAC_SHA256(secret, UTF8(message)) )
```

Фактический URL рукопожатия может быть `/ws/public` или `/ws/private` — **в подписи всё равно путь `/ws`**, как в реализации сервера.

## Способ 2: JWT

- Заголовок `Authorization: Bearer <token>`, **или**
- Query `?token=<token>`

Проверка по JWKS (настройки `ws-gateway.auth.jwt`).

## Способ 3: сессия Gateway

Если включено `session-cookie.enabled`, cookie **`GATEWAY_SESSION`** — сервер обращается к URL профиля пользователя Gateway (`user-info-url`, например `GET .../auth/me`) и строит контекст сессии.

## Поведение приватного пути

На **private** WebSocket соединение допускается только если пользователь аутентифицирован и имеет право на поток **`user`** (`READ`). Сессия после успешного handshake **автоматически подписывается** на поток `user`.

## Публичный-only сервис

Если `service-type: public`, клиенту разрешены в основном только `subscribe` / `unsubscribe` / `pong`; торговые и RFQ-сообщения с клиента не принимаются.

## Приватные префиксы потоков

По конфигурации (`private-stream-prefixes`): `user`, `provider.`, `rfq/`. На такие потоки нужна аутентификация и соответствующие права/роли (см. [private/ws/README.md](../private/ws/README.md)).

## См. также

- [REST API-ключ](rest-api-key.md) (алгоритм HMAC для HTTP)
- [Публичный WS](../public/ws/README.md) · [Приватный WS](../private/ws/README.md)
- [protos/ws.proto](../protos/ws.proto)
- [← Обзор авторизации](README.md)
