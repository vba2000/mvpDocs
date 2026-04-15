# Админ — `/admin/*`

[← Прочие приватные эндпоинты](../other.md)

Требуется аутентификация; в описаниях API — роль **OPERATOR**.

## `/admin/sessions`

| Метод и путь | Назначение |
|--------------|------------|
| GET `/admin/sessions/listSessions/{userId}` | Все сессии пользователя |
| POST `/admin/sessions/revoke-all/{userId}` | Отозвать все сессии пользователя |
| DELETE `/admin/sessions/{userId}/{sessionId}` | Отозвать одну сессию |

## `/admin/users/{userId}`

| Метод и путь | Назначение |
|--------------|------------|
| GET `.../spot/trades/history` | Сделки пользователя (фильтры + `Pageable`) |
| GET `.../accounts/transfers/history` | Переводы |
| GET `.../accounts/deposits/history` | Депозиты |
