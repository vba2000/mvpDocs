# REST: RFQ taker API

## Назначение

RFQ-раздел предназначен для клиента, который хочет запросить котировку у платформы и затем отслеживать жизненный цикл RFQ. В этой документации описывается только taker-side интеграция, доступная обычному внешнему API key клиенту.

Не документируются:

- `POST /rfq/providers/quote`
- `POST /rfq/quote/indicative`
- `POST /rfq/quote/firm`
- `GET /rfq/provider/quotes`

Эти методы относятся к market maker-роли.

## Методы taker-side

| Метод | Путь | Назначение |
|------|------|------------|
| `POST` | `/rfq/create` | Создать RFQ |
| `GET` | `/rfq/{rfqId}` | Получить RFQ по id |
| `POST` | `/rfq/{rfqId}/cancel` | Отменить RFQ |
| `GET` | `/rfq/list` | История и список RFQ |
| `GET` | `/rfq/indicative-price` | Получить предварительную агрегированную цену |

## `GET /rfq/indicative-price`

Лучший стартовый метод для предпросмотра RFQ-сценария.

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `marketId` | string | да | Рынок, например `BTC_USDT` |
| `side` | string | да | `BUY` или `SELL` |
| `quantity` | string | да | Объем заявки |

Ответ `IndicativePriceResponseDto`:

- `price`
- `userQuantity`
- `availableQuantity`
- `minQuantity`
- `validUntilMs`

Этот метод нужен бизнесу для pre-check: показать пользователю ориентир до создания полноценного RFQ.

## `POST /rfq/create`

Создает RFQ.

`RfqCreateRequest`:

| Поле | Тип | Назначение |
|------|-----|------------|
| `marketId` | string | RFQ рынок |
| `side` | enum | `BUY` или `SELL` |
| `quantity` | string | Объем заявки |
| `worstPrice` | string | Предельная худшая цена исполнения |
| `ttlMs` | integer | Время жизни RFQ |

### Бизнес-смысл полей

- `worstPrice` ограничивает максимально допустимое ухудшение цены;
- `ttlMs` ограничивает окно, в котором RFQ может быть обработан;
- `quantity` должна соответствовать ограничениям рынка и доступной ликвидности.

## `GET /rfq/{rfqId}`

Возвращает текущее состояние RFQ.

Ключевые поля `RfqResponseDto`:

| Поле | Назначение |
|------|------------|
| `rfqId` | Идентификатор RFQ |
| `marketId` | Рынок |
| `side` | Направление |
| `quantity` | Запрошенный объем |
| `status` | Статус RFQ |
| `bestQuotePrice` | Лучшая котировка |
| `executionPrice` | Цена исполнения |
| `executedBaseAmount` | Исполненный объем в base asset |
| `executedQuoteAmount` | Исполненный объем в quote asset |
| `executionDeadline` | Дедлайн исполнения |
| `createdAt` | Время создания |

Статусы по OpenAPI:

- `PENDING`
- `QUOTING`
- `QUOTED`
- `EXECUTING`
- `EXECUTED`
- `EXPIRED`
- `CANCELLED`
- `FAILED`

## `POST /rfq/{rfqId}/cancel`

Останавливает активный RFQ до его исполнения или истечения TTL.

## `GET /rfq/list`

История и список RFQ пользователя.

Фильтры:

- `status`
- `pageable`

Ответ:

- `rfqRequests[]`
- `total`
- `limit`
- `offset`

## Рекомендуемый бизнес-флоу

1. Получить рынок из `/market/rfq-exchange-info`.
2. Запросить `GET /rfq/indicative-price`.
3. Создать `POST /rfq/create`.
4. Читать статус через `GET /rfq/{rfqId}` или private WebSocket события пользователя.
5. При необходимости отменить `POST /rfq/{rfqId}/cancel`.

## Типичные ошибки

- `marketId` не входит в RFQ universe.
- `quantity` меньше минимального доступного объема.
- `worstPrice` делает RFQ невыполнимым.
- TTL слишком короткий для выполнения бизнес-сценария.

## См. также

- [market.md](market.md)
- [../ws/private.md](../ws/private.md)
- [../flows/quickstart.md](../flows/quickstart.md)
