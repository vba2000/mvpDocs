# Приватный REST: обзор

Все перечисленные ниже префиксы требуют **аутентификации**: API-ключ ([auth/rest-api-key.md](../../auth/rest-api-key.md)) или сессия ([auth/rest-session.md](../../auth/rest-session.md)).

Исключения — только если явно указано (например публичные `/market/*`).

## Префиксы

| Префикс | Индекс | Описание |
|---------|--------|----------|
| `/spot/*` | [spot.md](spot.md) | Ордера и история; подробно: [spot/orders-create.md](spot/orders-create.md) |
| `/accounts/*` | [accounts.md](accounts.md) | Балансы, комиссии, переводы, депозиты, история |
| `/rfq/*` | [rfq.md](rfq.md) | RFQ и котировки (часть — **MARKET_MAKER**) |
| `/aggregate-price/*` и др. | [other.md](other.md) | Агрегат, приватный `/auth`, `/users`, `/admin` — см. [other/](other/) |

Параметр **`userId`** в query там, где он указан, действителен **только для OPERATOR**; обычные пользователи его игнорируют.

## OpenAPI

- UI: [Swagger UI (dev)](https://cex-dev.web3tech.ru/api/v1/gateway/swagger-ui/index.html).
- JSON: `GET {REST_BASE}/v3/api-docs`.

## См. также

- [Публичный REST](../../public/rest/README.md)
- [← Оглавление](../../README.md)
