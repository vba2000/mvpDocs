# WS: подписка, отписка, heartbeat

[← Сообщения клиента](../client-messages.md)

### `SubscribeMessage` — `subscribe`

| Поле | Описание |
|------|----------|
| `streams` | Имена потоков, напр. `spot/trades.BTC_USDT` |
| `trace_id` | Опционально |

### `UnsubscribeMessage` — `unsubscribe`

| Поле | Описание |
|------|----------|
| `streams` | Имена отключаемых потоков |
| `trace_id` | Опционально |

### `PongMessage` — `pong`

Ответ на серверный `ping`; поле `trace_id` по желанию.
