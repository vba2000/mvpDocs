# Публичный WebSocket: подключение

## Назначение

Потоковая доставка рыночных данных: стакан, сделки, тикер, свечи, агрегированный индекс цены.

## URL

Зависит от деплоя. Пример: `wss://cex-dev.web3tech.ru/api/v1/ws-gw/ws/public`

См. также [auth/websocket.md](../../auth/websocket.md) для рукопожатия.

## Формат транспорта

Query-параметр **`format`**: protobuf (**binary**) или **json**. По умолчанию — из конфигурации сервера (`default-transport-format`).

## Сообщения клиента → сервер

Логически соответствуют `WsClientMessage` в [`protos/ws.proto`](../../protos/ws.proto):

- **Подписка:** список имён потоков `streams[]`
- **Отписка:** те же имена
- **Pong:** ответ на серверный ping (см. heartbeat в конфиге: интервал ping, таймаут pong)

В JSON обычно поле `action`: `subscribe`, `unsubscribe`, `pong`.

## Сообщения сервера → клиент

Оболочка `WsEnvelope` / `WsResult`: события `subscribed`, `unsubscribed`, `ping`, `data`. Полезная нагрузка рынка лежит в **`data`** как protobuf (binary) или декодированный JSON.

## Лимиты (типичный конфиг)

- Макс. подписок на сессию: смотрите `max-subscriptions-per-session` (по умолчанию 10 в `application.yml`).
- Ограничение числа соединений с одного IP на публичном endpoint — `limits.max-connections-per-ip` (0 = выкл.).
- Throttle: интервалы `orderbook-flush-ms`, `trades-flush-ms`, `candle-flush-ms` и т.д.

Коды ошибок: enum `WsErrorCode` в `ws.proto` (`INVALID_STREAM`, `SUBSCRIPTION_LIMIT_EXCEEDED`, …).

## Каталог потоков (публичные)

| Документ | Маска потока |
|----------|----------------|
| [stream-orderbook.md](stream-orderbook.md) | `spot/orderbook.{market_id}` |
| [stream-trades.md](stream-trades.md) | `spot/trades.{market_id}` |
| [stream-ticker.md](stream-ticker.md) | `spot/ticker.{market_id}` |
| [stream-candle.md](stream-candle.md) | `spot/candle.{market_id}.{timeframe}` |
| [stream-aggregate-price.md](stream-aggregate-price.md) | `spot/aggregate_price` |

`market_id` обычно в формате `BTC_USDT` (подчёркивание).

## См. также

- [Приватный WS](../../private/ws/README.md)
- [Авторизация WS](../../auth/websocket.md)
- [← Оглавление](../../README.md)
