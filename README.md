# Документация внешней интеграции CEX

Документация описывает только те REST API и WebSocket-сценарии, которые пригодны для интеграции по `API key` и HMAC-подписи. Browser login, session-only методы, operator-only и admin-only API в этой версии не документируются как внешние интеграционные контракты.

## Тестовая среда

| Назначение | URL |
|------------|-----|
| REST Gateway | `https://cex-test.web3tech.ru/api/v1/gateway` |
| Swagger UI | [https://cex-test.web3tech.ru/api/v1/gateway/swagger-ui/index.html#](https://cex-test.web3tech.ru/api/v1/gateway/swagger-ui/index.html#) |
| OpenAPI JSON | [https://cex-test.web3tech.ru/api/v1/gateway/v3/api-docs](https://cex-test.web3tech.ru/api/v1/gateway/v3/api-docs) |
| Test stand | [https://cex-test.web3tech.ru/](https://cex-test.web3tech.ru/) |
| WebSocket public | `wss://cex-test.web3tech.ru/api/v1/ws-gw/ws/public` |
| WebSocket private | `wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private` |

Все REST пути в документации указаны относительно базы `https://cex-test.web3tech.ru/api/v1/gateway`. Для HMAC-подписи в REST используется путь после хоста, включая префикс `/api/v1/gateway` и query string.

## Что входит в документацию

- Публичные REST market-методы.
- Приватные REST-методы, доступные внешнему клиенту по `API key`: `spot`, `accounts`, `rfq` taker-side, `aggregate-price`.
- Управление API keys.
- Публичный и приватный WebSocket, включая режим `format=json`.
- Интеграционные флоу с пошаговым описанием запроса, ответа и бизнес-смысла.

## Что исключено

- Session-only `/auth/*`, кроме операций управления `API key`.
- Browser/OAuth flow.
- `admin` и `users` API.
- Operator-only параметры и сценарии, если они не нужны обычной внешней интеграции.
- Market maker-only RFQ API и MM WebSocket-потоки как часть общей интеграции по API key.

## Структура документации

1. [Аутентификация REST по API key](auth/api-key-rest.md)
2. [Аутентификация private WebSocket](auth/api-key-websocket.md)
3. [Примеры подписи и запросов](auth/signing-examples.md)
4. [Обзор REST API](rest/overview.md)
5. [Market data](rest/market.md)
6. [Spot trading](rest/spot.md)
7. [Accounts](rest/accounts.md)
8. [RFQ taker API](rest/rfq.md)
9. [Aggregate price](rest/aggregate-price.md)
10. [Обзор WebSocket](ws/overview.md)
11. [Public WebSocket](ws/public.md)
12. [Private WebSocket](ws/private.md)
13. [JSON-формат сообщений](ws/message-format-json.md)
14. [Protobuf/binary-формат](ws/message-format-protobuf.md)
15. [Интеграционные сценарии](flows/quickstart.md)

## Принципы чтения

- Swagger и OpenAPI остаются основным источником формальных схем DTO и enum.
- Эта документация дополняет Swagger: объясняет бизнес-флоу, порядок вызовов, правила подписи, ограничения, side effects и типовые ошибки.
- Если OpenAPI и фактическое поведение сервера расходятся, в тексте это отмечается отдельно.
