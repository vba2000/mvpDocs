# REST API: обзор

## Назначение

REST Gateway для внешней интеграции делится на два контура:

- публичный read-only слой без аутентификации;
- приватный слой, который вызывается по `API key` и HMAC-подписи.

Базовый URL тестовой среды: `https://cex-test.web3tech.ru/api/v1/gateway`

## Общая модель ответа

Большинство ответов приходят в единой обертке:

```json
{
  "code": 0,
  "msg": "OK",
  "result": {},
  "extInfo": {},
  "time": 1713200000123
}
```

| Поле | Тип | Назначение |
|------|-----|------------|
| `code` | integer | Код результата; `0` обычно означает успешную обработку |
| `msg` | string | Краткий текстовый статус |
| `result` | object/array/scalar | Полезная нагрузка метода |
| `extInfo` | object | Дополнительные технические детали |
| `time` | integer | Время формирования ответа на стороне Gateway |

## Группы методов

| Группа | Доступ | Раздел |
|--------|--------|--------|
| `/market/*` | публично | [market.md](market.md) |
| `/network` | публично | [market.md](market.md) |
| `GET /spot/orderbook` | публично | [market.md](market.md) |
| `/spot/*` кроме `orderbook` | `API key` | [spot.md](spot.md) |
| `/accounts/*` | `API key` | [accounts.md](accounts.md) |
| `/deposit-addresses/*` | `API key` | [accounts.md](accounts.md) |
| `/rfq/*` taker-side | `API key` | [rfq.md](rfq.md) |
| `/aggregate-price/*` | `API key` | [aggregate-price.md](aggregate-price.md) |

## Что не входит

Эта документация не покрывает как внешний интеграционный контракт:

- `/users/*`;
- `/admin/*`;
- `POST /accounts/deposit`;
- browser onboarding и account management flow;
- market maker-only сценарии;
- операторские impersonation-сценарии.

## Общие правила для приватных методов

### HMAC-подпись обязательна

Для приватных REST-методов строка подписи строится так:

```text
{timestamp}{METHOD}{pathWithQuery}{body}
```

Где:

- `timestamp` совпадает с `X-Timestamp`;
- `METHOD` всегда в верхнем регистре;
- `pathWithQuery` включает путь после хоста и query string;
- `body` это точная UTF-8 строка тела запроса.

Подробности и примеры: [../auth/api-key-rest.md](../auth/api-key-rest.md).

### Денежные и количественные поля

Цены, количества, суммы, quote budgets и многие торговые идентификаторы передаются строками. Это важно и для точности, и для побайтного совпадения тела запроса с HMAC-подписью.

### Идемпотентность и retry-safe вызовы

Для production-интеграции нужно считать обязательными:

- `clientOrderId` в торговых командах;
- `idempotencyKey` в account transfer;
- собственный client-side correlation id в retry сценариях.

## Типовой порядок интеграции

1. Получить и сохранить `API key` и `secret`.
2. Синхронизировать часы через `GET /market/time`.
3. Получить справочники рынков и сетей.
4. Выполнять приватные запросы с HMAC-подписью.
5. Держать private WebSocket как live-источник пользовательских изменений.

## См. также

- [../auth/api-key-rest.md](../auth/api-key-rest.md)
- [../concepts.md](../concepts.md)
- [../errors.md](../errors.md)
- [market.md](market.md)
- [spot.md](spot.md)
- [accounts.md](accounts.md)
- [../ws/overview.md](../ws/overview.md)
