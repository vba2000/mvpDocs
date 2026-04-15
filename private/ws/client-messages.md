# WebSocket: сообщения от клиента

Все варианты описаны в **`WsClientMessage`** ([`protos/ws.proto`](../../protos/ws.proto)). Опционально на любом сообщении: **`trace_id`** для сквозной трассировки.

В **JSON**-режиме обычно поле **`action`** со значением, совпадающим с именем protobuf-поля в snake_case (`subscribe`, `create_order`, …).

## Разделы

| Тема | Документ |
|------|----------|
| Подписка, отписка, pong | [messages/subscribe-heartbeat.md](messages/subscribe-heartbeat.md) |
| RFQ и маркет-мейкинг | [messages/rfq-mm.md](messages/rfq-mm.md) |
| Спотовые ордера (create/cancel/batch) | [messages/spot-trading.md](messages/spot-trading.md) |

## Ошибки

Сервер может вернуть событие ошибки с кодом из **`WsErrorCode`** (`INVALID_MESSAGE`, `AUTHENTICATION_REQUIRED`, …).

## См. также

- [Обзор приватного WS](README.md)
- [Публичный WS: подписки](../../public/ws/README.md)
