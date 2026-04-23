# REST: accounts

## Назначение

Группа `/accounts/*` описывает состояние активов пользователя и внутренние перемещения между счетами. Для внешней интеграции по API key документируются только пользовательские методы. Операторский депозит не входит в этот раздел.

## Методы

| Метод | Путь | Назначение |
|------|------|------------|
| `GET` | `/accounts/balance` | Балансы по активам и account types |
| `GET` | `/accounts/fee-tier` | Текущий maker/taker fee tier |
| `POST` | `/accounts/transfer` | Внутренний перевод между счетами |
| `GET` | `/accounts/transfers/history` | История внутренних переводов |
| `GET` | `/accounts/deposits/history` | История депозитных операций в учете платформы |

Не документируется:

- `POST /accounts/deposit` — operator-only метод.

## `GET /accounts/balance`

Возвращает список балансов пользователя.

Ответ `BalancesResponseDto`:

| Поле | Назначение |
|------|------------|
| `assetId` | Код актива |
| `available` | Доступный остаток |
| `reserved` | Зарезервированный остаток |
| `accountType` | Тип счета |
| `version` | Версия записи баланса |

Бизнес-применение:

- показывать доступный баланс перед выставлением ордера;
- сверять post-trade остатки;
- контролировать перемещения между `SPOT`, `RFQ`, `FUTURES`, `FUNDING`.

## `GET /accounts/fee-tier`

Возвращает комиссии клиента:

- `makerFeeBps`
- `takerFeeBps`

Используется для предрасчета комиссий, PnL и котирования.

## `POST /accounts/transfer`

Внутренний перевод между счетами пользователя.

`TransferRequestDto`:

| Поле | Тип | Назначение |
|------|-----|------------|
| `assetId` | string | Перемещаемый актив |
| `amount` | string | Сумма перевода |
| `fromAccountType` | enum | Источник |
| `toAccountType` | enum | Получатель |
| `idempotencyKey` | string | Ключ идемпотентности |

Поддерживаемые account types:

- `SPOT`
- `RFQ`
- `FUTURES`
- `FUNDING`

### Flow перевода

1. Клиент читает текущие балансы.
2. Клиент принимает бизнес-решение о переводе.
3. Клиент формирует `TransferRequestDto`.
4. Клиент подписывает запрос HMAC.
5. Сервер выполняет перевод и возвращает обновленные состояния исходного и целевого баланса.

`TransferResponseDto`:

- `success`
- `transferId`
- `message`
- `fromBalance`
- `toBalance`

## `GET /accounts/transfers/history`

История внутренних переводов.

Фильтры:

| Параметр | Тип | Назначение |
|----------|-----|------------|
| `status` | string | Статус операции |
| `assetId` | string | Фильтр по активу |
| `dateFromMs` | integer | Начало периода |
| `dateToMs` | integer | Конец периода |
| `limit` | integer | Размер страницы |
| `offset` | integer | Смещение |

Используется для reconciliation и отчетности.

## `GET /accounts/deposits/history`

Возвращает историю зачислений, попавших в учетный контур платформы.

С точки зрения внешней интеграции этот метод полезен не для инициирования депозита, а для чтения истории уже отраженных операций.

Фильтры аналогичны transfer history:

- `status`
- `assetId`
- `dateFromMs`
- `dateToMs`
- `limit`
- `offset`

## Что важно учитывать

- Денежные суммы передаются строками.
- Для `transfer` нужно всегда задавать `idempotencyKey`.
- Истории по `transfers` и `deposits` нужны для аналитики и reconciliation, а не для low-latency статусов.

## См. также

- [overview.md](overview.md)
- [spot.md](spot.md)
- [../ws/private.md](../ws/private.md)
