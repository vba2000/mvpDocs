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

## Ошибки

Типовые причины ошибок:

- неподдерживаемый stream;
- превышен лимит подписок;
- действие недоступно на данном сокете;
- нет прав на stream или команду;
- некорректный JSON payload;
- поле передано числом вместо строки.

## Когда выбирать JSON

- интеграция делается быстро и без protobuf toolchain;
- нужен websocket в браузерном proxy/service-layer;
- команда предпочитает human-readable payloads для отладки.

## Когда выбирать binary

- важна компактность сообщений;
- высокие нагрузки и частые market updates;
- уже есть protobuf pipeline.

## См. также

- [message-format-protobuf.md](message-format-protobuf.md)
- [public.md](public.md)
- [private.md](private.md)
