# Авторизация: обзор

Интеграция с биржей возможна по двум каналам — **REST** и **WebSocket**. В обоих случаях можно использовать **API-ключ + HMAC-подпись** или **сессию браузера** (cookie). Для WebSocket дополнительно поддерживаются **JWT** в заголовке или query.

## Сводная таблица

| Канал | API-ключ (боты, сервер-к-сервер) | Сессия (BFF / браузер) | JWT (WS) |
|-------|----------------------------------|-------------------------|----------|
| REST | [rest-api-key.md](rest-api-key.md) | [rest-session.md](rest-session.md) | Не используется на Gateway |
| WebSocket | Те же заголовки, **другая строка подписи** | Cookie `GATEWAY_SESSION` | `Authorization: Bearer` или `?token=` |

Важно:

- Для **REST** подпись считается от **реального** HTTP-метода, пути (с query) и тела.
- Для **WebSocket** при API-ключе подпись считается от **фиксированных** значений: метод `WEBSOCKET`, путь `/ws`, тело `WS_CONNECT` — см. [websocket.md](websocket.md).

## Документы раздела

- [REST: API-ключ и HMAC](rest-api-key.md)
- [REST: сессия (cookie)](rest-session.md)
- [WebSocket: рукопожатие и методы входа](websocket.md)

## Верхний уровень

[← К оглавлению](../README.md) · [Понятия](../concepts.md)
