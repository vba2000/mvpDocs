# WebSocket: обзор

## Назначение

WebSocket-контур разделен на два независимых сценария:

- public WebSocket для рыночных данных;
- private WebSocket для пользовательских событий и клиентских команд.

Это разные точки интеграции с разными правилами доступа и разными типами потоков.

## URL тестовой среды

| Канал | URL |
|-------|-----|
| Public | `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public` |
| Private | `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private` |

## Transport format

Оба сервиса поддерживают query-параметр `format`:

- `format=binary` — protobuf;
- `format=json` — text frames.

Если `format` не передан, используется `binary`.

## Главные отличия public и private

| Признак | Public | Private |
|---------|--------|---------|
| Аутентификация | не требуется | обязательна |
| Цель | market data | user events и команды |
| Потоки | `spot/orderbook.*`, `spot/trades.*`, `spot/ticker.*`, `spot/candle.*`, `spot/aggregate_price` | `user`, служебные private streams |
| Client actions | `subscribe`, `unsubscribe`, `pong` | `subscribe`, `unsubscribe`, `pong`, trading/RFQ commands |

## Общий жизненный цикл соединения

1. Клиент выбирает public или private endpoint.
2. Клиент выбирает `format`.
3. Клиент открывает WebSocket.
4. После соединения клиент отправляет subscribe/unsubscribe messages.
5. Сервер периодически отправляет `ping`.
6. Клиент отвечает `pong`.
7. Сервер отправляет envelopes с `subscribed`, `unsubscribed`, `data`, `error`.

## Что важно знать заранее

- Public и private нельзя смешивать: private socket не предназначен для market streams, а public socket не принимает private commands.
- Для private API key WebSocket HMAC считается по фиксированной canonical string, а не по фактическому URL.
- В JSON-режиме числа во многих payload полях сериализуются как строки.

## См. также

- [public.md](public.md)
- [private.md](private.md)
- [message-format-json.md](message-format-json.md)
- [message-format-protobuf.md](message-format-protobuf.md)
