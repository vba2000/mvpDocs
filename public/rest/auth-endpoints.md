# Публичный REST: фрагмент `/auth/*`

Следующие маршруты **разрешены без аутентификации** на уровне Spring Security. Остальные `/auth/*` (например `/auth/me`, `/auth/api-keys`) требуют cookie сессии — см. [private/rest/README.md](../../private/rest/README.md).

---

## POST `/auth/login`

**Назначение:** старт OAuth2 PKCE — в ответе URL для перенаправления пользователя на провайдера.

**Тело:** JSON `Map` — поля задаёт фронт/BFF согласно реализации контроллера (PKCE verifier/challenge и т.д.).

**Ответ `result`:** `LoginInitResponse`

| Поле | Тип | Описание |
|------|-----|----------|
| `authorizationUrl` | string | URL для редиректа браузера |

---

## GET `/auth/callback`

**Назначение:** обработка редиректа от OAuth2; обмен кода на токены, создание сессии, **HTTP 302** на фронтенд.

**Query:** стандартные `code`, `state`, при ошибке — `error`, `error_description`.

**Ответ:** редирект, не JSON-тело обёртки в успешном сценарии.

---

## POST `/auth/refresh`

**Назначение:** обновление access-токена по refresh; cookie `GATEWAY_SESSION` сохраняется.

**Заголовки / cookie:** `GATEWAY_SESSION` (опционально, если нет — 401).

**Ответ:** `ResponseEntity<RefreshResult>` — см. OpenAPI для полей `RefreshResult`.

---

## GET `/auth/session`

**Назначение:** проверить, есть ли валидная сессия, срок и флаг принудительной смены пароля. **Работает без обязательной аутентификации** (cookie опциональна).

**Cookie:** `GATEWAY_SESSION` (опционально).

**Ответ:** `SessionStatusResponse` внутри стандартной обёртки (через `ResponseEntity`):

| Поле | Тип | Описание |
|------|-----|----------|
| `authenticated` | boolean | Есть ли активная сессия |
| `expiresIn` | long \| null | Секунд до истечения (если применимо) |
| `requiresPasswordChange` | boolean | Нужно сменить пароль |

---

## См. также

- [Сессия и cookie](../../auth/rest-session.md)
- [Приватные auth-операции](../../private/rest/README.md)
- [Обзор публичного REST](README.md)
