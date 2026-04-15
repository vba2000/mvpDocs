# Приватный REST: `/spot/*`

Префикс **`/spot`**. Аутентификация обязательна. Для ряда GET допускается query **`userId`** только у **OPERATOR** (см. подстраницы и OpenAPI).

Обёртка ответа: стандартная (`result` содержит DTO).

## Каталог методов

### Ордера (изменение состояния)

| Метод | Путь | Кратко | Подробно |
|-------|------|--------|----------|
| POST | `/spot/orders/create` | Новый ордер, тело `OrderCreateRequest` → `CommandResponseDto` | **[orders-create.md](spot/orders-create.md)** (поля, enum, правила) |
| POST | `/spot/orders/cancel` | Отмена: query `orderId` **или** `clientOrderId` → `CommandResponseDto` | — |
| POST | `/spot/orders/cancel-and-replace` | Тело `CancelAndReplaceRequest` → `CommandResponseDto` | OpenAPI |
| POST | `/spot/orders/batch/create` | Тело `BatchOrderCreateRequest` → `BatchCommandResponseDto` | — |
| POST | `/spot/orders/batch/cancel` | Тело `BatchOrderCancelRequest` → `BatchCommandResponseDto` | — |
| POST | `/spot/orders/cancel-all` | Тело `CancelAllOrdersRequest` (опц. фильтр рынка) → `BatchCommandResponseDto` | OpenAPI |

### Ордера и сделки (чтение)

| Метод | Путь | Кратко |
|-------|------|--------|
| GET | `/spot/orders/active` | Активные ордера; фильтры `symbol`, `orderId`, `clientOrderId`, `orderTypeFilter`, `userId` (OPERATOR) → `SpotActiveOrdersResponseDto` |
| GET | `/spot/orders/history` | История ордеров, курсор: фильтры по символу/типу/датам, `includeTrades`, `limit`≤100, `cursor`, `userId` → `SpotOrdersHistoryResponseDto` |
| GET | `/spot/trades/history` | Исполненные сделки, Spring `Pageable` + фильтры рынка/дат/`userId` → `TradesHistoryResponseDto` |
| GET | `/spot/executions/history` | История исполнений (fills), курсор → `ExecutionsHistoryResponseDto` |
| GET | `/spot/deals/history` | Сделки матчера в разрезе ордера, `Pageable` → `DealsHistoryResponseDto` |

Полные схемы полей ответов — [Swagger / OpenAPI](https://cex-dev.web3tech.ru/api/v1/gateway/swagger-ui/index.html).

## См. также

- [Счета](accounts.md)
- [Приватный WS: сообщения клиента](../ws/client-messages.md)
- [Обзор приватного REST](README.md)
