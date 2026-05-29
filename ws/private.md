# Private WebSocket

## Назначение

Private WebSocket используется для двух задач:

- получать пользовательские события без polling;
- отправлять торговые команды и сразу видеть реакцию торгового контура.

URL тестовой среды:

`wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private`

## Подключение

Private socket требует аутентификации. Для внешней интеграции основной сценарий это `API key + HMAC`.

Подробности handshake: [../auth/api-key-websocket.md](../auth/api-key-websocket.md)

## Жизненный цикл private соединения

1. Клиент открывает `wss://.../ws/private?format=json` или `?format=binary`.
2. Передает `X-API-Key`, `X-Timestamp`, `X-Signature`.
3. Сервер валидирует подпись и права.
4. После успешного входа сервер автоматически подписывает сессию на `user` и `notifications`.
5. Клиент получает служебные подтверждения и затем данные по этим потокам.
6. Клиент может отправлять private commands.

## Private streams

Для обычной внешней интеграции ключевыми потоками являются `user` и `notifications`.

| Stream | Статус в этой документации | Назначение |
|--------|----------------------------|------------|
| `user` | описывается | Пользовательские события |
| `notifications` | описывается | Уведомления по депозитным операциям |

## Поток `user`

Главный интеграционный stream для:

- жизненного цикла ордеров;
- execution/fill событий;
- обновлений балансов;
- внутренних переводов;
- RFQ user events.

Поток нужен как online-дополнение к REST history endpoints.

Типы payload внутри `user` для внешнего клиента:

| Тип события | Для чего нужен |
|-------------|----------------|
| `orders` | Изменения жизненного цикла ордеров |
| `fill` | Исполнения и trade-level факты |
| `balance` | Точечное обновление одного баланса |
| `balances` | Пакетное обновление нескольких балансов |
| `transfer` | Факт внутреннего перевода |
| `rfq` | Изменение статуса RFQ |

Практическое правило: `user` не заменяет historical REST, а дает live-изменения между snapshot/reconciliation циклами.

## Поток `notifications`

Этот поток нужен для live-уведомлений по депозитному контуру.

Типовой payload включает deposit notification batch, внутри которого есть события с полями:

- `deposit_id`
- `amount`
- `token`
- `status`
- `updated_at`
- `event_type`

Практически это значит, что клиент может получать депозитные изменения в live-режиме без polling `deposit-operations`.

Для внешней интеграции `notifications` надо трактовать как быстрый канал оповещения, а `GET /accounts/deposit-operations` как источник подтвержденного списка и фильтруемого history view.

## Client actions в JSON-режиме

Private сокет поддерживает:

- `subscribe`
- `unsubscribe`
- `pong`
- `create_order`
- `cancel_order`
- `batch_create_order`
- `batch_cancel_order`
- `cancel_all_orders`
- `create_rfq`

## Пример команды создания ордера

```json
{
  "action": "create_order",
  "market_id": "BTC_USDT",
  "side": "BUY",
  "order_type": "LIMIT",
  "time_in_force": "GTC",
  "quantity": "0.001",
  "price": "50000",
  "client_order_id": "bot-order-0001"
}
```

## Пример `user` event в JSON

```json
{
  "code": "0",
  "msg": "OK",
  "time": "1713200000123",
  "event": "data",
  "stream": "user",
  "data": {
    "type": "orders",
    "orders": [
      {
        "order_id": "123456",
        "client_order_id": "bot-order-0001",
        "market_id": "BTC_USDT",
        "status": "PARTIALLY_FILLED",
        "side": "BUY",
        "order_type": "LIMIT",
        "price": "50000",
        "quantity": "0.001",
        "executed_quantity": "0.0004"
      }
    ]
  }
}
```

## Пример `notifications` event в JSON

```json
{
  "code": "0",
  "msg": "OK",
  "time": "1713200000999",
  "event": "data",
  "stream": "notifications",
  "data": {
    "type": "deposit_operation",
    "notifications": [
      {
        "deposit_id": "dep-10001",
        "status": "CHECK",
        "amount": "125.50",
        "token": "USDT",
        "network": "TRON",
        "updated_at": "2026-04-22T10:15:30Z"
      }
    ]
  }
}
```

## Права доступа

| Возможность | Требование |
|-------------|------------|
| Подключение к private WS | `READ` на private streams |
| Получение `user` events | `READ` |
| Получение `notifications` | `READ` |
| Торговые команды | `TRADE` |

## Что private socket не делает

- не используется для публичных market streams;
- не заменяет REST полностью, потому что initial snapshot и история удобнее через REST;
- не заменяет остальные внутренние сервисные каналы платформы.

## Практический интеграционный паттерн

1. Получить начальный snapshot через REST.
2. Поднять private WebSocket.
3. Держать `user` и `notifications` как live-источник изменений.
4. В случае разрыва:
   - переподключиться;
   - заново синхронизировать состояние через REST history methods;
   - считать пропущенным любой период между disconnect и новым snapshot.

## Recovery pattern

1. Переподключить private WS и заново пройти handshake.
2. Дождаться auto-subscribe на `user` и `notifications`.
3. Прочитать через REST:
   - `/spot/orders/active`
   - `/spot/orders/history`
   - `/accounts/balance`
   - `/accounts/transfers/history`
   - `/accounts/deposit-operations`, если используется депозитный контур
4. Пересобрать локальное состояние, а затем снова применять live events.

## См. также

- [overview.md](overview.md)
- [message-format-json.md](message-format-json.md)
- [../rest/spot.md](../rest/spot.md)
- [../rest/rfq.md](../rest/rfq.md)
