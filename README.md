# Документация внешнего API биржи

Материалы для команд интеграции: REST Gateway, WebSocket-шлюз, авторизация по API-ключу и сессии.

## Базовые URL (пример окружения dev)

| Назначение | URL |
|------------|-----|
| REST Gateway | `https://cex-dev.web3tech.ru/api/v1/gateway/` |
| Swagger UI (dev) | [swagger-ui/index.html](https://cex-dev.web3tech.ru/api/v1/gateway/swagger-ui/index.html) |
| WebSocket (публичный) | `wss://cex-dev.web3tech.ru/api/v1/ws-gw/ws/public` |
| WebSocket (приватный) | `wss://cex-dev.web3tech.ru/api/v1/ws-private/ws/private` |

Пути API ниже указаны **относительно префикса REST** (без хоста). У продакшена префиксы могут совпадать по смыслу, но домен и версия пути задаются деплоем.

## Оглавление

1. **[Основные понятия](concepts.md)** — публичное/приватное, форматы ответов, роли.
2. **Авторизация** — [обзор](auth/README.md), [REST: API-ключ](auth/rest-api-key.md), [REST: сессия](auth/rest-session.md), [WebSocket](auth/websocket.md).
3. **Публичные данные**
   - REST: [обзор](public/rest/README.md), [рынок `/market` (индекс + подстраницы)](public/rest/market.md), [публичные `/auth/*`](public/rest/auth-endpoints.md).
   - WebSocket: [подключение и протокол](public/ws/README.md), потоки: [стакан](public/ws/stream-orderbook.md), [сделки](public/ws/stream-trades.md), [тикер](public/ws/stream-ticker.md), [свечи](public/ws/stream-candle.md), [агрегированная цена](public/ws/stream-aggregate-price.md).
4. **Приватные данные**
   - REST: [обзор](private/rest/README.md), [Spot](private/rest/spot.md) ([создание ордера](private/rest/spot/orders-create.md)), [счета](private/rest/accounts.md), [RFQ](private/rest/rfq.md), [прочее / админка](private/rest/other.md).
   - WebSocket: [обзор](private/ws/README.md), [поток `user`](private/ws/stream-user.md), [RFQ провайдера](private/ws/stream-rfq-provider.md), [`provider.markets`](private/ws/stream-provider-markets.md), [сообщения клиента](private/ws/client-messages.md) ([спот](private/ws/messages/spot-trading.md), [RFQ](private/ws/messages/rfq-mm.md)).
5. **Protobuf** — [каталог `protos/`](protos/README.md), файл [`protos/ws.proto`](protos/ws.proto).

## Машиночитаемые схемы

- REST: интерактивная документация — [Swagger UI (dev)](https://cex-dev.web3tech.ru/api/v1/gateway/swagger-ui/index.html). Сырой OpenAPI JSON: `GET /v3/api-docs` относительно базы Gateway (dev: `https://cex-dev.web3tech.ru/api/v1/gateway/v3/api-docs`).
- WebSocket (JSON-обёртка сообщений): в репозитории бэкенда — `websocket-gateway-app/src/main/resources/openapi/ws-json-openapi.yaml`; на развёрнутом WS-сервисе путь задаётся конфигом (см. `springdoc.ws-json-api-docs` в `application.yml`).
