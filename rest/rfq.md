# REST: RFQ taker API

## Назначение

RFQ-раздел предназначен для клиента, который хочет запросить котировку у платформы и затем отслеживать жизненный цикл RFQ. В этой документации описывается только taker-side интеграция, доступная обычному внешнему API key клиенту.

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

Фактические статусы RFQ:

- `PENDING`
- `QUOTING`
- `QUOTED`
- `EXECUTING`
- `EXECUTED`
- `EXPIRED`
- `CANCELLED`
- `FAILED`

## State machine RFQ

Упрощенно для taker-интегратора:

1. `PENDING` — RFQ зарегистрирован.
2. `QUOTING` — идет сбор/обработка котировок.
3. `QUOTED` — появилась подходящая котировка.
4. `EXECUTING` — система пытается исполнить RFQ.
5. Финал:
   - `EXECUTED`
   - `EXPIRED`
   - `CANCELLED`
   - `FAILED`

REST и private WS `user` должны рассматриваться как два способа чтения одного жизненного цикла.

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

Содержимое элементов повторяет `RfqResponseDto`, поэтому список можно использовать и как текущую ленту, и как исторический RFQ журнал.

## Рекомендуемый бизнес-флоу

1. Получить рынок из `/market/rfq-exchange-info`.
2. Запросить `GET /rfq/indicative-price`.
3. Создать `POST /rfq/create`.
4. Читать статус через `GET /rfq/{rfqId}` и private WebSocket `user`.
5. При необходимости отменить `POST /rfq/{rfqId}/cancel`.

## Что делает taker на каждом этапе

| Этап | Действие клиента |
|------|------------------|
| До создания RFQ | Проверить market universe и indicative price |
| Во время quoting | Не повторять create blindly, а читать статус |
| При quoted/executing | Отслеживать progression через REST и WS |
| При expired/cancelled/failed | Принимать решение о повторном запросе |
| При executed | Сверять итог через RFQ state и user events |

## Типичные ошибки

- `marketId` не входит в RFQ universe.
- `quantity` меньше минимального доступного объема.
- `worstPrice` делает RFQ невыполнимым.
- TTL слишком короткий для выполнения бизнес-сценария.
- Клиент не различает `QUOTED` и `EXECUTING`, из-за чего преждевременно считает RFQ завершенным.

## См. также

- [market.md](market.md)
- [../ws/private.md](../ws/private.md)
- [../flows/quickstart.md](../flows/quickstart.md)
