# Пользователи — `/users/*` (OPERATOR)

[← Прочие приватные эндпоинты](../other.md)

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

Детали — OpenAPI.
