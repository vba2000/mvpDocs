# REST: accounts

## Назначение

Группа `/accounts/*` описывает состояние активов пользователя, funding/spot перемещения, депозитные адреса и операции депозита. Для внешней интеграции документируются только пользовательские методы. Операторский `POST /accounts/deposit` в этот раздел не входит.

## Методы

| Метод | Путь | Назначение |
|------|------|------------|
| `GET` | `/accounts/balance` | Балансы по активам и account types |
| `GET` | `/accounts/fee-tier` | Текущий maker/taker fee tier |
| `POST` | `/accounts/transfer` | Внутренний перевод между счетами |
| `GET` | `/accounts/transfers/history` | История внутренних переводов |
| `GET` | `/accounts/deposits/history` | История депозитных операций в учете платформы |
| `GET` | `/accounts/deposit-wallet` | Список депозитных адресов пользователя |
| `GET` | `/accounts/deposit-operations` | Список депозитных операций пользователя |
| `GET` | `/deposit-addresses/settings` | Поддерживаемые сети, активы и лимиты адресов |
| `POST` | `/deposit-addresses` | Создание нового депозитного адреса |

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
- контролировать funding/spot перемещения и post-trade состояние.

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

Фактически поддерживаемое направление в gateway:

- `FUNDING -> SPOT`
- `SPOT -> FUNDING`

Другие направления нужно считать невалидными для внешней интеграции, даже если enum `AccountType` шире.

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

Типовые причины отказа:

- неподдерживаемое направление перевода;
- недостаточный баланс;
- невалидный `amount`;
- повторный вызов без корректной idempotency strategy.

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

## `GET /accounts/deposit-wallet`

Возвращает список депозитных адресов пользователя.

Основные фильтры:

- `status`
- `network`
- `searchQuery`
- `pageable`

Этот метод нужен для UI и reconciliation адресов, уже выданных пользователю.

Ключевые поля `DepositAddressItemDto`:

- `id`
- `custodyWalletId`
- `network`
- `address`
- `comment`
- `status`
- `created`
- `statusDate`

Статусы адресов:

- `ACTIVE`
- `ARCHIVED`

## `GET /accounts/deposit-operations`

Возвращает список депозитных операций пользователя.

Основные фильтры:

- `status`
- `network`
- `riskLevel`
- `searchQuery`
- `pageable`

Этот метод нужен для контроля жизненного цикла депозитов и сверки статусов.

Ключевые поля `DepositOperationItemDto`:

- `id`
- `txHash`
- `amount`
- `address`
- `assetId`
- `network`
- `status`
- `riskLevel`
- `confirmations`
- `requiredConfirmations`
- `createdAt`
- `updatedAt`

Типовые статусы депозитной операции:

- `PENDING`
- `FAILED`
- `CHECK`
- `DEPOSITED`
- `REJECTED`
- `REFUNDED`

Типовые уровни риска:

- `NONE`
- `LOW`
- `MEDIUM`
- `HIGH`
- `SEVERE`

## `GET /deposit-addresses/settings`

Возвращает:

- поддерживаемые активы;
- поддерживаемые сети;
- ограничения на выпуск депозитных адресов.

Это стартовый метод перед `POST /deposit-addresses`.

Ответ `DepositAddressSettingsResponseDto` важен как источник лимитов:

- `userMaxDepositWallets`
- список `assets[]`
- для каждого asset список `networks[]` с `network`, `code`, `fullName`, `standard`, `maxDepositWallets`

## `POST /deposit-addresses`

Создает новый депозитный адрес для текущего пользователя и выбранной сети.

Типовой payload:

| Поле | Тип | Назначение |
|------|-----|------------|
| `network` | string | Сеть адреса |
| `comment` | string | Необязательный комментарий |

Типовой ответ `DepositAddressResponseDto`:

- `id`
- `network`
- `address`
- `comment`
- `createdAt`

## End-to-end deposit flow

1. Клиент читает `GET /deposit-addresses/settings`.
2. Клиент выбирает asset и network с учетом лимитов адресов.
3. Клиент вызывает `POST /deposit-addresses`.
4. Клиент хранит адрес и показывает его пользователю или upstream-системе.
5. После on-chain перевода клиент отслеживает:
   - `GET /accounts/deposit-wallet`
   - `GET /accounts/deposit-operations`
   - private WS `notifications`

Такой flow позволяет совместить synchronous onboarding через REST и live tracking через WebSocket.

## Что важно учитывать

- Денежные суммы передаются строками.
- Для `transfer` нужно всегда задавать `idempotencyKey`.
- Для transfer документируйте только `FUNDING <-> SPOT`.
- Истории по `transfers` и `deposits` нужны для аналитики и reconciliation, а не для low-latency статусов.
- Для deposit integration лучший production pattern: REST для bootstrap и private WS `notifications` для live updates.

## См. также

- [overview.md](overview.md)
- [spot.md](spot.md)
- [../ws/private.md](../ws/private.md)
