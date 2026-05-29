# Интеграционные сценарии

## 1. Onboarding prerequisites

Перед первым боевым вызовом внешнему интегратору нужны:

1. `API key` и `secret`;
2. список разрешенных IP, если используется whitelist;
3. синхронизация времени через `GET /market/time`;
4. понимание, какие market ids и networks реально поддерживаются;
5. стратегия idempotency для trading и transfers.

## 2. Первый приватный REST-вызов

Цель: проверить, что `API key` работает.

1. Вызвать `GET /market/time` без авторизации.
2. Вычислить `X-Timestamp`.
3. Подписать `GET /api/v1/gateway/accounts/balance`.
4. Отправить запрос с `X-API-Key`, `X-Timestamp`, `X-Signature`.
5. Убедиться, что сервер вернул `balances[]`.

## 3. Первый ордер

Цель: безопасно пройти путь от справочника рынков до create order.

1. Вызвать `GET /market/exchange-info`.
2. Выбрать `marketId` и проверить допустимый `orderType`.
3. Вызвать `GET /accounts/balance`.
4. Подготовить `POST /spot/orders/create` с `clientOrderId`.
5. Подписать запрос и отправить его.
6. Проверить `CommandResponseDto`.
7. Дальше отслеживать состояние через:
   - `GET /spot/orders/active`
   - `GET /spot/orders/{orderId}`
   - `GET /spot/orders/history`
   - private WebSocket `user`

## 4. Первый депозитный flow

1. Прочитать `GET /deposit-addresses/settings`.
2. Выбрать asset/network и проверить лимиты адресов.
3. Создать адрес через `POST /deposit-addresses`.
4. Сохранить `id`, `network`, `address`.
5. После внешней on-chain отправки отслеживать:
   - `GET /accounts/deposit-wallet`
   - `GET /accounts/deposit-operations`
   - private WS `notifications`
6. При финальном статусе выполнить reconciliation в собственной учетной системе.

## 5. Первый public WebSocket

Цель: получить рыночные обновления без polling.

1. Открыть `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public?format=json`.
2. Отправить:

```json
{
  "action": "subscribe",
  "streams": ["spot/ticker.BTC_USDT", "spot/trades.BTC_USDT"]
}
```

1. Принимать `event=data`.
2. Отвечать `pong` на heartbeat сервера.

## 6. Первый private WebSocket

Цель: получать пользовательские события и команды через live-канал.

1. Сформировать `timestamp`.
2. Посчитать HMAC для `"{timestamp}WEBSOCKET/wsWS_CONNECT"`.
3. Открыть `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private?format=json`.
4. Передать `X-API-Key`, `X-Timestamp`, `X-Signature`.
5. Дождаться успешного подключения и auto-subscribe на `user` и `notifications`.
6. Отправлять private commands по мере необходимости.

## 7. RFQ taker flow

1. Получить рынок через `GET /market/rfq-exchange-info`.
2. Запросить `GET /rfq/indicative-price`.
3. Сформировать `POST /rfq/create`.
4. Читать `GET /rfq/{rfqId}` или `user` events.
5. При необходимости отменить `POST /rfq/{rfqId}/cancel`.

## 8. Recovery flow после разрыва WebSocket

1. Зафиксировать локальный момент разрыва и считать поток после него потенциально неполным.
2. Переподключить private WebSocket.
3. Повторно авторизоваться по API key.
4. Получить snapshots через REST:
   - `/spot/orders/active`
   - `/spot/orders/history`
   - `/accounts/balance`
   - `/accounts/transfers/history`
   - `/accounts/deposit-operations`, если клиент ведет депозитный контур
   - нужные history endpoints
5. Сверить восстановленное состояние с локальным кэшем.
6. Для market data отдельно переподключить public WS и заново взять `/spot/orderbook`, если клиент поддерживает локальную книгу.

## См. также

- [../auth/api-key-rest.md](../auth/api-key-rest.md)
- [../auth/api-key-websocket.md](../auth/api-key-websocket.md)
- [../rest/spot.md](../rest/spot.md)
- [../ws/private.md](../ws/private.md)
