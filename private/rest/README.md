# Приватный REST: обзор

Все перечисленные ниже префиксы требуют **аутентификации**: API-ключ ([auth/rest-api-key.md](../../auth/rest-api-key.md)) или сессия ([auth/rest-session.md](../../auth/rest-session.md)).

Исключения — только если явно указано (например публичные `/market/*`).

## Префиксы

| Префикс | Документ | Описание |
|---------|----------|----------|
| `/spot/*` | [spot.md](spot.md) | Спотовые ордера и история |
| `/accounts/*` | [accounts.md](accounts.md) | Балансы, комиссии, переводы, депозиты, история |
| `/rfq/*` | [rfq.md](rfq.md) | RFQ, котировки; часть методов — роль MARKET_MAKER |
| `/aggregate-price/*` | [other.md](other.md) | Агрегированные цены |
| `/auth/*` (кроме публичных) | [other.md](other.md) | Профиль, API-ключи, сессии |
| `/users/*` | [other.md](other.md) | Только **OPERATOR** |
| `/admin/*` | [other.md](other.md) | Админ-сессии и история по userId |

Параметр **`userId`** в query там, где он указан, действителен **только для OPERATOR**; обычные пользователи его игнорируют.

## OpenAPI

- UI: [Swagger UI (dev)](https://cex-dev.web3tech.ru/api/v1/gateway/swagger-ui/index.html).
- JSON: `GET {REST_BASE}/v3/api-docs`.

## См. также

- [Публичный REST](../../public/rest/README.md)
- [← Оглавление](../../README.md)
