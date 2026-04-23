# Интеграционные сценарии

## 1. Первый приватный REST-вызов

Цель: проверить, что `API key` работает.

1. Вызвать `GET /market/time` без авторизации.
2. Вычислить `X-Timestamp`.
3. Подписать `GET /api/v1/gateway/accounts/balance`.
4. Отправить запрос с `X-API-Key`, `X-Timestamp`, `X-Signature`.
5. Убедиться, что сервер вернул `balances[]`.

## 2. Первый ордер

Цель: безопасно пройти путь от справочника рынков до create order.

1. Вызвать `GET /market/exchange-info`.
2. Выбрать `marketId` и проверить допустимый `orderType`.
3. Вызвать `GET /accounts/balance`.
4. Подготовить `POST /spot/orders/create` с `clientOrderId`.
5. Подписать запрос и отправить его.
6. Проверить `CommandResponseDto`.
7. Дальше отслеживать состояние через:
   - `GET /spot/orders/active`
   - `GET /spot/orders/history`
   - private WebSocket `user`

## 3. Первый public WebSocket

Цель: получить рыночные обновления без polling.

1. Открыть `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public?format=json`.
2. Отправить:

```json
{
  "action": "subscribe",
  "streams": ["spot/ticker.BTC_USDT", "spot/trades.BTC_USDT"]
}
```

3. Принимать `event=data`.
4. Отвечать `pong` на heartbeat сервера.

## 4. Первый private WebSocket

Цель: получать пользовательские события и команды через live-канал.

1. Сформировать `timestamp`.
2. Посчитать HMAC для `"{timestamp}WEBSOCKET/wsWS_CONNECT"`.
3. Открыть `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private?format=json`.
4. Передать `X-API-Key`, `X-Timestamp`, `X-Signature`.
5. Дождаться успешного подключения и auto-subscribe на `user`.
6. Отправлять private commands по мере необходимости.

## 5. RFQ taker flow

1. Получить рынок через `GET /market/rfq-exchange-info`.
2. Запросить `GET /rfq/indicative-price`.
3. Сформировать `POST /rfq/create`.
4. Читать `GET /rfq/{rfqId}` или `user` events.
5. При необходимости отменить `POST /rfq/{rfqId}/cancel`.

## 6. Recovery flow после разрыва WebSocket

1. Переподключить private WebSocket.
2. Повторно авторизоваться по API key.
3. Получить snapshots через REST:
   - `/spot/orders/active`
   - `/accounts/balance`
   - нужные history endpoints
4. Сверить восстановленное состояние с локальным кэшем.

## См. также

- [../auth/api-key-rest.md](../auth/api-key-rest.md)
- [../auth/api-key-websocket.md](../auth/api-key-websocket.md)
- [../rest/spot.md](../rest/spot.md)
- [../ws/private.md](../ws/private.md)
