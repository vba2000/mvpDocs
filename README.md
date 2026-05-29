# Документация внешней интеграции CEX

Документация описывает внешний интеграционный контракт для `api/v1/gateway` и public/private WebSocket. В центре внимания только сценарии внешнего клиента: публичный read-only REST, пользовательский REST по `API key` и streaming через public/private WS.

## Тестовая среда

| Назначение | URL |
|------------|-----|
| REST Gateway | `https://cex-test.web3tech.ru/api/v1/gateway` |
| Test stand | [https://cex-test.web3tech.ru/](https://cex-test.web3tech.ru/) |
| WebSocket public | `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public` |
| WebSocket private | `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private` |

Все REST пути в документации указаны относительно базы `https://cex-test.web3tech.ru/api/v1/gateway`. Для HMAC-подписи в REST используется путь после хоста, включая префикс `/api/v1/gateway` и query string.

## Что входит в документацию

- Публичные read-only REST-методы: `market`, `network`, публичный snapshot `spot/orderbook`.
- Приватные REST-методы, доступные внешнему клиенту по `API key`: `spot`, `accounts`, `deposit-addresses`, `rfq` taker-side, `aggregate-price`.
- Аутентификация по `API key` и HMAC для REST и private WebSocket.
- Публичный и приватный WebSocket, включая режим `format=json`.
- Интеграционные флоу с пошаговым описанием запроса, ответа и бизнес-смысла.

## Что исключено

- Back-office и внутренние сервисы.
- `admin`, `users`, `internal` и browser onboarding.
- Операторские impersonation-сценарии.
- Market maker-only сценарии, которые не относятся к обычной внешней интеграции.

## Структура документации

1. [Основные понятия](concepts.md)
2. [Каталог ошибок](errors.md)
3. [Аутентификация REST по API key](auth/api-key-rest.md)
4. [Аутентификация private WebSocket](auth/api-key-websocket.md)
5. [Примеры подписи и запросов](auth/signing-examples.md)
6. [Обзор REST API](rest/overview.md)
7. [Market data](rest/market.md)
8. [Spot trading](rest/spot.md)
9. [Accounts](rest/accounts.md)
10. [RFQ taker API](rest/rfq.md)
11. [Aggregate price](rest/aggregate-price.md)
12. [Обзор WebSocket](ws/overview.md)
13. [Public WebSocket](ws/public.md)
14. [Private WebSocket](ws/private.md)
15. [JSON-формат сообщений](ws/message-format-json.md)
16. [Protobuf/binary-формат](ws/message-format-protobuf.md)
17. [Интеграционные сценарии](flows/quickstart.md)

## Принципы чтения

- Источник правды для этой документации: актуальные backend contracts, DTO, handlers, security rules и runtime validation в коде.
- Эта документация описывает не только поля запросов, но и бизнес-флоу, правила подписи, side effects, ограничения и реальные ограничения исполнения.
- Если описания интерфейсов и фактическое поведение gateway/service слоя расходятся, в тексте приоритет отдается коду исполнения и это отмечается явно.
- В спецификации используются оба формата идентификаторов рынков: `BTC/USDT` и `BTC_USDT`. Разница и правила применения описаны в [concepts.md](concepts.md).
