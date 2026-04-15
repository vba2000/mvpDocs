# Приватный WebSocket: обзор

## Приватные префиксы потоков

Задаются конфигурацией `ws-gateway.auth.private-stream-prefixes` (типично):

- `user`
- `provider.`
- `rfq/`

Подписка на такие потоки требует **аутентифицированной** сессии и удовлетворения правил ниже.

## Матрица «поток → доступ»

| Поток | Условие |
|-------|---------|
| `user` или `user.*` | Право **`Permission.READ`** |
| `provider.*` (кроме спец. случаев) | Роль **`MARKET_MAKER`** |
| `rfq/provider` | Роль **`MARKET_MAKER`** |
| Остальные (`spot/...`) | Публичные с точки зрения прав (но см. ограничения public-only сервиса) |

Логика: `WsAuthService.hasStreamPermission` в коде шлюза.

## Приватный endpoint

URL примера: `wss://cex-dev.web3tech.ru/api/v1/ws-private/ws/private`

На приватном пути соединение закрывается, если нельзя подписаться на **`user`**. После успешного handshake сервер **автоматически** подписывает клиента на поток `user`.

## Документы по потокам и сообщениям

- [Поток `user`](stream-user.md) — батчи `UserEventBatchProto`
- [Поток `rfq/provider`](stream-rfq-provider.md)
- [Пуш `provider.markets`](stream-provider-markets.md)
- [Сообщения от клиента (торги, RFQ)](client-messages.md) — [подписки](messages/subscribe-heartbeat.md), [спот](messages/spot-trading.md), [RFQ/MM](messages/rfq-mm.md)

## См. также

- [Авторизация WS](../../auth/websocket.md)
- [Публичный WS](../../public/ws/README.md)
- [protos/ws.proto](../../protos/ws.proto)
- [← Оглавление](../../README.md)
