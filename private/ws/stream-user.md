# WebSocket: поток `user`

## Имя потока

```
user
```

На приватном URL подписка выполняется **автоматически** после успешной аутентификации.

## Кому доступен

Аутентифицированный пользователь с правом **`READ`**.

## Полезная нагрузка

**`UserEventBatchProto`**: поле `events` — список **`UserEventProto`**, каждый элемент с `oneof payload`:

| Вариант | Сообщение | Содержание |
|---------|-----------|------------|
| `order` | `UserOrderProto` | События ордера: статус, цены, объёмы, `order_id`, `user_order_id`, комиссии и т.д. |
| `fill` | `UserFillProto` | Исполнение: `trade_id`, цена, объём, комиссия, maker/taker, `execution_id` |
| `balance` | `UserBalanceProto` | Остатки по активу и типу счёта (`available`, `reserved`, `version`) |
| `transfer` | `UserTransferProto` | Внутренний перевод между типами счетов |
| `rfq` | `RfqEventProto` | События RFQ в пользовательском контексте |

Полные поля — в [`protos/ws.proto`](../../protos/ws.proto).

События накапливаются в батч с интервалом `private-flush-ms` (конфиг).

## См. также

- [Обзор приватного WS](README.md)
- [RFQ провайдера](stream-rfq-provider.md)
