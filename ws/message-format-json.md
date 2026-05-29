# WebSocket JSON format

## Назначение

JSON-режим нужен командам, которым удобнее интегрироваться без protobuf-клиента. Он включается query-параметром:

`?format=json`

## Общий envelope

Типовой ответ сервера выглядит так:

```json
{
  "code": "0",
  "msg": "OK",
  "time": "1713200000123",
  "event": "data",
  "stream": "user",
  "data": {}
}
```

### Поля envelope

| Поле | Тип в JSON | Назначение |
|------|------------|------------|
| `code` | string | Код результата |
| `msg` | string | Текстовый статус |
| `time` | string | Время события |
| `event` | string | Тип envelope |
| `stream` | string | Имя потока, если это `data` |
| `data` | object | Payload события |
| `message` | string | Текст ошибки при `event=error` |
| `ext_info` | object | Дополнительные детали |

## Важное правило по типам

В JSON-режиме числовые значения в сообщениях часто сериализуются как строки.

Это касается:

- цен;
- количеств;
- сумм;
- многих numeric полей внутри event payload;
- полей envelope вроде `code` и `time`.

Практический вывод: клиент должен парсить JSON не по наивному ожиданию `number`, а по фактическому контракту stringified numbers.

## Client actions

### Public/private общие

```json
{
  "action": "subscribe",
  "streams": ["spot/ticker.BTC_USDT"]
}
```

```json
{
  "action": "unsubscribe",
  "streams": ["spot/ticker.BTC_USDT"]
}
```

```json
{
  "action": "pong"
}
```

### Private trading actions

Примеры action names:

- `create_order`
- `cancel_order`
- `batch_create_order`
- `batch_cancel_order`
- `cancel_all_orders`
- `create_rfq`

## Server events

| `event` | Назначение |
|---------|------------|
| `subscribed` | Подтверждение подписки |
| `unsubscribed` | Подтверждение отписки |
| `ping` | Heartbeat сервера |
| `data` | Полезная нагрузка потока |
| `error` | Ошибка протокола, авторизации или бизнес-операции |
| `empty` | Технический пустой envelope |

## Ошибки

Серверные ошибки в runtime кодируются числовыми кодами диапазона `4001`-`5000`, которые в JSON приходят как строки в поле `code`.

Типовые причины ошибок:

- неподдерживаемый stream;
- превышен лимит подписок;
- действие недоступно на данном сокете;
- нет прав на stream или команду;
- некорректный JSON payload;
- поле передано числом вместо строки.

## Streams в JSON-режиме

### Public

Основные `stream` значения:

- `spot/orderbook.<market_id>`
- `spot/trades.<market_id>`
- `spot/ticker.<market_id>`
- `spot/candle.<market_id>.<timeframe>`
- `spot/aggregate_price`

### Private

Основные `stream` значения:

- `user`
- `notifications`

Для `user` payload содержит user event batch, для `notifications` — notification batch по депозитным операциям.

### Как читать `user`

Для внешнего клиента в `user` важны следующие смысловые категории payload:

- `orders` — создание, обновление, отмена, закрытие ордеров;
- `fill` — отдельные execution/trade события;
- `balance` / `balances` — обновления свободного и зарезервированного остатка;
- `transfer` — завершенный внутренний перевод;
- `rfq` — изменение статуса RFQ и итог его исполнения.

### Как читать `notifications`

`notifications` используется для депозитного lifecycle:

- появление новой операции;
- переход в `CHECK`;
- переход в финальный статус вроде `DEPOSITED`, `FAILED`, `REJECTED`, `REFUNDED`.

Типовой production pattern:

1. получать live envelopes через WS;
2. при важных сменах статуса дополнительно перечитывать `GET /accounts/deposit-operations`.

## Когда выбирать JSON

- интеграция делается быстро и без protobuf toolchain;
- нужен websocket в proxy/service-layer;
- команда предпочитает human-readable payloads для отладки.

## Когда выбирать binary

- важна компактность сообщений;
- высокие нагрузки и частые market updates;
- уже есть protobuf pipeline.

## См. также

- [message-format-protobuf.md](message-format-protobuf.md)
- [public.md](public.md)
- [private.md](private.md)
