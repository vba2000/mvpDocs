# Приватный REST: прочие эндпоинты

## Aggregate Price — `/aggregate-price/*`

Аутентификация требуется (не входит в публичный список Security).

### GET `/aggregate-price/latest`

**Назначение:** последние агрегированные цены по списку рынков.

| Параметр | Обяз. | Описание |
|----------|-------|----------|
| `markets` | да | Повторяющийся query или список ID, напр. `BTC_USDT` |

**Ответ `result`:** массив `AggregatePriceDto`

| Поле | Описание |
|------|----------|
| `marketId` | Рынок |
| `aggregatePrice` | Значение индекса |
| `change24` | Изменение за 24ч |
| `timestamp` | Метка времени (`Instant`) |

---

### GET `/aggregate-price/history`

**Назначение:** история агрегата по рынкам и интервалу.

| Параметр | Обяз. | Описание |
|----------|-------|----------|
| `markets` | да | Список рынков |
| `from`, `to` | да | Epoch **миллисекунды** |
| `interval` | да | `1min`, `10min`, `1h`, `1d`, `1w` |
| `limit` | нет | Макс. точек на рынок (до 1000) |

**Ответ `result`:** `List<AggregatePriceDto>`.

---

## Auth — остальные `/auth/*` (приватные)

Требуют cookie **`GATEWAY_SESSION`** (или иной успешной аутентификации, если расширено).

| Метод и путь | Назначение |
|--------------|------------|
| POST `/auth/logout` | Завершить сессию |
| POST `/auth/change-password` | Смена пароля |
| GET `/auth/me` | Профиль пользователя (`UserInfoResponse`) |
| POST `/auth/api-keys` | Создать API-ключ (`GatewayCreateApiKeyResponse`, секрет один раз) |
| GET `/auth/api-keys` | Список ключей |
| GET `/auth/api-keys/{apiKeyId}` | Один ключ |
| DELETE `/auth/api-keys/{apiKeyId}` | Отозвать ключ |
| GET `/auth/sessions` | Список сессий пользователя |
| POST `/auth/sessions/revoke-others` | Отозвать другие сессии |
| DELETE `/auth/sessions/{targetSessionId}` | Отозвать конкретную сессию |

Для операций с ключами у OPERATOR доступен query **`userId`**.

Детали полей — в OpenAPI.

---

## Пользователи — `/users/*` (OPERATOR)

Все маршруты требуют авторити **`OPERATOR`**.

| Метод и путь | Назначение |
|--------------|------------|
| POST `/users` | Создать пользователя → ID |
| GET `/users` | Список с пагинацией |
| GET `/users/{userId}` | Карточка пользователя |
| POST `/users/{userId}/block` | Заблокировать |
| POST `/users/{userId}/unblock` | Разблокировать |
| POST `/users/{userId}/psw-reset` | Сброс пароля |
| GET `/users/{userId}/status` | Статус (строка) |
| PUT `/users/{userId}/roles` | Обновить роли (`GatewayUpdateRolesRequest`) |

---

## Админ — `/admin/*`

Требуется аутентификация; в описаниях API указана роль **OPERATOR**.

### `/admin/sessions`

| Метод и путь | Назначение |
|--------------|------------|
| GET `/admin/sessions/listSessions/{userId}` | Все сессии пользователя |
| POST `/admin/sessions/revoke-all/{userId}` | Отозвать все сессии пользователя |
| DELETE `/admin/sessions/{userId}/{sessionId}` | Отозвать одну сессию |

### `/admin/users/{userId}`

| Метод и путь | Назначение |
|--------------|------------|
| GET `.../spot/trades/history` | Сделки пользователя (фильтры + `Pageable`) |
| GET `.../accounts/transfers/history` | Переводы |
| GET `.../accounts/deposits/history` | Депозиты |

---

## См. также

- [Spot](spot.md) · [Accounts](accounts.md) · [RFQ](rfq.md)
- [Публичный WS: aggregate price](../../public/ws/stream-aggregate-price.md)
- [Обзор приватного REST](README.md)
