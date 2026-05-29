# Основные понятия

## Назначение

Этот раздел фиксирует базовые термины и соглашения внешней спецификации, чтобы REST и WebSocket читались одинаково всеми командами интеграции.

## Базовые URL

- REST Gateway: `https://cex-test.web3tech.ru/api/v1/gateway`
- Public WebSocket: `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public`
- Private WebSocket: `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private`

## Форматы идентификаторов рынков

В системе используются два распространенных представления символа:

| Формат | Пример | Где обычно используется |
|--------|--------|-------------------------|
| Trading/chart symbol | `BTC/USDT` | `GET /market/symbols`, `GET /market/history`, search/charting-compatible endpoints |
| Market id | `BTC_USDT` | `spot`, `rfq`, `tickers`, `mark-prices`, WebSocket streams и торговые payload |

Главное правило: всегда использовать тот формат, который ожидает конкретный endpoint или stream.

## Типы счетов

Во внешней спецификации чаще всего встречаются:

- `SPOT`
- `FUNDING`

Хотя backend enum шире, в пользовательском transfer-сценарии документируются только направления `FUNDING -> SPOT` и `SPOT -> FUNDING`.

## REST response wrapper

Большинство REST-ответов приходят в общей оболочке:

```json
{
  "code": 0,
  "msg": "OK",
  "result": {},
  "extInfo": {},
  "time": 1713200000123
}
```

`code = 0` обычно означает успех. Полезная нагрузка лежит в `result`.

## WebSocket envelope

В WebSocket сервер отправляет envelope с:

- `code`
- `msg`
- `time`
- `event`
- `stream`
- `data`

В JSON-режиме многие числа приходят строками.

## Жизненный цикл spot-ордера

Упрощенная модель для интегратора:

1. Клиент читает `exchange-info`.
2. Клиент валидирует order fields и рыночные ограничения.
3. Клиент отправляет `create order`.
4. Gateway отвечает `202 Accepted`.
5. Финальное состояние приходит через:
   - `GET /spot/orders/{orderId}`
   - `GET /spot/orders/history`
   - private WebSocket `user`

`202 Accepted` не означает финальное исполнение, а только успешный прием команды в обработку.

## Жизненный цикл RFQ

Типовой taker flow:

1. Получить RFQ market universe.
2. Получить indicative price.
3. Создать RFQ.
4. Отслеживать статусы через REST и `user` events.
5. Либо получить исполнение, либо отмену/истечение.

Типовые статусы:

- `PENDING`
- `QUOTING`
- `QUOTED`
- `EXECUTING`
- `EXECUTED`
- `EXPIRED`
- `CANCELLED`
- `FAILED`

## Жизненный цикл депозита

Типовой flow:

1. Получить `deposit-addresses/settings`.
2. Создать адрес через `POST /deposit-addresses`.
3. Читать список адресов через `GET /accounts/deposit-wallet`.
4. После on-chain транзакции отслеживать статусы через:
   - `GET /accounts/deposit-operations`
   - private WebSocket stream `notifications`

## Private WebSocket streams

Для обычного внешнего клиента основными являются:

- `user` — ордера, fills, балансы, transfers, RFQ события
- `notifications` — депозитные уведомления

## См. также

- [errors.md](errors.md)
- [rest/overview.md](rest/overview.md)
- [ws/overview.md](ws/overview.md)
