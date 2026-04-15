# Публичный REST: обзор

Эндпоинты ниже **не требуют** аутентификации (список `permitAll` в Security Gateway: `/market/**` и выбранные `/auth/*`, плюс swagger/actuator).

## Разделы

- [Рынок: `/market/*`](market.md) — котировки, свечи, символы, тикеры.
- [Публичная часть `/auth/*`](auth-endpoints.md) — старт логина, callback, обновление сессии, статус сессии.

## Полная машиночитаемая схема

OpenAPI: `GET {REST_BASE}/v3/api-docs` (например `https://cex-dev.web3tech.ru/api/v1/gateway/v3/api-docs`).

## Приватный REST

Всё остальное на Gateway — с API-ключом или сессией: [private/rest/README.md](../../private/rest/README.md).

## См. также

- [Авторизация](../../auth/README.md)
- [← Оглавление](../../README.md)
