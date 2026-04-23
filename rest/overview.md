# REST API: обзор

## Назначение

REST Gateway делится на две группы контрактов:

- публичные market-методы без аутентификации;
- приватные методы, в которые внешний клиент входит по `API key` через заголовки `X-API-Key`, `X-Timestamp`, `X-Signature`.

Базовый URL тестовой среды: `https://cex-test.web3tech.ru/api/v1/gateway`

## Общая модель ответа

Большинство успешных и ошибочных ответов приходят в единой обертке:

```json
{
  "code": 0,
  "msg": "OK",
  "result": {},
  "extInfo": {},
  "time": 1713200000123
}
```

### Поля обертки

| Поле | Тип | Назначение |
|------|-----|------------|
| `code` | integer | Код результата. `0` обычно означает успешную обработку |
| `msg` | string | Краткий текстовый статус |
| `result` | object/array/scalar | Полезная нагрузка метода |
| `extInfo` | object | Дополнительные детали ошибки или обработки |
| `time` | integer | Время формирования ответа на стороне Gateway |

## Группы методов в этой документации

| Группа | Аутентификация | Раздел |
|--------|----------------|--------|
| `/market/*` | не требуется | [market.md](market.md) |
| `/spot/*` | `API key` | [spot.md](spot.md) |
| `/accounts/*` | `API key` | [accounts.md](accounts.md) |
| `/rfq/*` taker-side | `API key` | [rfq.md](rfq.md) |
| `/aggregate-price/*` | `API key` | [aggregate-price.md](aggregate-price.md) |
| `/auth/api-keys*` | сессия пользователя | [../auth/api-key-rest.md](../auth/api-key-rest.md) |

## Что не входит

Внешняя интеграционная документация не описывает:

- `/users/*`;
- `/admin/*`;
- `POST /accounts/deposit`;
- market maker-only RFQ endpoints;
- browser/session-only auth flow вроде `/auth/login`, `/auth/callback`, `/auth/me`, `/auth/sessions`.

## Общие правила для приватных методов

### 1. Подпись обязательна

Для всех приватных REST-методов строка подписи строится так:

```text
{timestamp}{METHOD}{pathWithQuery}{body}
```

Где:

- `timestamp` совпадает с `X-Timestamp`;
- `METHOD` всегда в верхнем регистре;
- `pathWithQuery` включает путь после хоста и query string;
- `body` это точная UTF-8 строка тела запроса, для `GET` обычно пустая строка.

Подробности и примеры приведены в [../auth/api-key-rest.md](../auth/api-key-rest.md).

### 2. Денежные и количественные поля

Практически все цены, количества, суммы и идентификаторы торгового контура передаются как строки. Это важно для точности и для побайтного совпадения тела с HMAC-подписью.

### 3. Параметр `userId`

В OpenAPI у части методов встречается query-параметр `userId`. Он предназначен для операторских сценариев и не нужен обычной внешней интеграции по API key. В этой документации такие параметры не рассматриваются как целевой сценарий.

### 4. Идемпотентность

Там, где метод поддерживает `clientOrderId` или `idempotencyKey`, эти поля нужно считать обязательными для production-интеграции, даже если сервер не везде делает их формально required.

## Типовой порядок интеграции

1. Получить и сохранить `API key` и `secret`.
2. Синхронизировать часы клиента через `GET /market/time`.
3. Получить market metadata через `GET /market/exchange-info` и при необходимости `GET /market/symbols`.
4. Выполнять приватные запросы с HMAC-подписью.
5. Для событий и статусов использовать private WebSocket параллельно с REST.

## См. также

- [../auth/api-key-rest.md](../auth/api-key-rest.md)
- [../ws/overview.md](../ws/overview.md)
- [../flows/quickstart.md](../flows/quickstart.md)
