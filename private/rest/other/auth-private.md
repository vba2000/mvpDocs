# Auth — приватные `/auth/*`

[← Прочие приватные эндпоинты](../other.md)

Требуют cookie **`GATEWAY_SESSION`** (или иную успешную аутентификацию, если расширено).

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

Для операций с ключами у **OPERATOR** доступен query **`userId`**.

Поля тел и ответов — в [OpenAPI / Swagger](https://cex-dev.web3tech.ru/api/v1/gateway/swagger-ui/index.html).

## См. также

- [Публичные auth-эндпоинты](../../../public/rest/auth-endpoints.md)
- [REST: сессия](../../../auth/rest-session.md)
