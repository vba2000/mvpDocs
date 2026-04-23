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
4. После успешного входа сервер автоматически подписывает сессию на `user`.
5. Клиент получает служебное подтверждение и затем данные по `user`.
6. Клиент может отправлять private commands.

## Private streams

Для обычной внешней интеграции главным потоком является `user`.

| Stream | Статус в этой документации | Назначение |
|--------|----------------------------|------------|
| `user` | описывается | Пользовательские события |
| `provider.markets` | не целевой | MM-only сценарий |
| `rfq/provider` | не целевой | MM-only сценарий |

## Поток `user`

Главный интеграционный stream для:

- жизненного цикла ордеров;
- execution/fill событий;
- обновлений балансов;
- внутренних переводов;
- RFQ user events.

Поток нужен как online-дополнение к REST history endpoints.

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

Также в коде есть MM-specific actions вроде `submit_firm_quote`, но они не относятся к общей внешней интеграции.

## Пример команды создания ордера

```json
{
  "action": "create_order",
  "market_id": "BTC_USDT",
  "base_asset_id": "BTC",
  "quote_asset_id": "USDT",
  "side": "BUY",
  "order_type": "LIMIT",
  "time_in_force": "GTC",
  "quantity": "0.001",
  "price": "50000",
  "client_order_id": "bot-order-0001"
}
```

## Права доступа

| Возможность | Требование |
|-------------|------------|
| Подключение к private WS | `READ` на поток `user` |
| Получение `user` events | `READ` |
| Торговые команды | `TRADE` |

## Что private socket не делает

- не используется для публичных market streams;
- не заменяет REST полностью, потому что initial snapshot и история удобнее через REST;
- не документирует operator/MM-only сценарии в этом наборе docs.

## Практический интеграционный паттерн

1. Получить начальный snapshot через REST.
2. Поднять private WebSocket.
3. Держать `user` stream как live-источник изменений.
4. В случае разрыва:
   - переподключиться;
   - заново синхронизировать состояние через REST history methods.

## См. также

- [overview.md](overview.md)
- [message-format-json.md](message-format-json.md)
- [../rest/spot.md](../rest/spot.md)
- [../rest/rfq.md](../rest/rfq.md)
