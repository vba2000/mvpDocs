# REST: market data

## Назначение

Группа `/market/*` дает публичные рыночные данные. Эти методы не требуют `API key`, используются для инициализации интеграции, синхронизации времени, построения UI/аналитики и предварительной валидации торговых запросов.

Базовый префикс: `GET https://cex-test.web3tech.ru/api/v1/gateway/market/*`

## Набор методов

| Метод | Путь | Бизнес-смысл |
|------|------|--------------|
| `GET` | `/market/time` | Серверное время для синхронизации часов клиента |
| `GET` | `/market/config` | Конфигурация charting/market data |
| `GET` | `/market/symbols` | Карточка одного символа |
| `GET` | `/market/search` | Поиск рынков |
| `GET` | `/market/history` | OHLCV история |
| `GET` | `/market/exchange-info` | Каталог spot markets и торговых ограничений |
| `GET` | `/market/rfq-exchange-info` | Каталог RFQ markets |
| `GET` | `/market/recent-trade` | Последние публичные сделки |
| `GET` | `/market/tickers` | 24h ticker snapshot |

## Рекомендуемый порядок использования

1. `GET /market/time`
2. `GET /market/exchange-info`
3. `GET /market/symbols` для конкретного инструмента
4. `GET /market/tickers` и/или public WebSocket для рыночных обновлений
5. `GET /market/history` для построения свечей

## Детали по методам

### `GET /market/time`

Используется для выравнивания часов клиента перед приватными REST и private WS запросами.

Ответ:

- `result.time` — время сервера в миллисекундах.

### `GET /market/exchange-info`

Главная точка входа для spot-интеграции.

Что получает клиент:

- список активных рынков;
- `baseAsset` и `quoteAsset`;
- точность по активам;
- список поддерживаемых `orderTypes`;
- текущий статус рынка.

Зачем вызывать:

- построить каталог инструментов;
- валидировать типы ордеров до отправки;
- понять доступность рынка для торговли.

### `GET /market/symbols`

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | да | Символ вида `BTC/USDT` |

Возвращает детальную карточку символа: scale, resolutions, exchange metadata, intraday capability.

### `GET /market/search`

Подходит для поисковых строк и symbol picker.

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `query` | string | да | Строка поиска |
| `type` | string | нет | Фильтр типа инструмента |
| `exchange` | string | нет | Фильтр биржи/источника |
| `limit` | integer | нет | Максимум результатов |

### `GET /market/history`

Нужен для свечей и backfill market data.

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | да | Символ вида `BTC/USDT` |
| `resolution` | string | да | Таймфрейм, например `1`, `5`, `15`, `60`, `1D`, `1W` |
| `from` | integer | да | Начало интервала, Unix timestamp в секундах |
| `to` | integer | да | Конец интервала, Unix timestamp в секундах |
| `limit` | integer | нет | Максимум свечей |

Важно: `from` и `to` в OpenAPI заданы в секундах, а не в миллисекундах.

### `GET /market/tickers`

Используется для 24h snapshot рынка.

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `category` | string | нет | Категория рынка |
| `symbol` | string | нет | Фильтр по одному рынку |
| `baseCoin` | string | нет | Фильтр по базовому активу |

Возвращает `lastPrice`, `bid1Price`, `ask1Price`, `volume24h`, `turnover24h`, `highPrice24h`, `lowPrice24h`.

### `GET /market/recent-trade`

Нужен для блока recent trades и начальной подкачки перед переходом на WebSocket.

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | нет | Символ вида `BTC_USDT` |
| `limit` | integer | нет | Число записей, максимум 100 |

### `GET /market/config`

Служебный метод для chart- и widget-интеграций. Возвращает поддерживаемые resolutions и capability flags.

### `GET /market/rfq-exchange-info`

Каталог RFQ-рынков. Используется до вызовов `/rfq/*`, чтобы понять допустимые пары и precision.

## Типичные ошибки

- Некорректный `symbol` или `resolution` -> бизнес-ошибка в `result`/`extInfo`.
- Неверный диапазон времени -> пустой набор или ошибка валидации.
- Использование `/` и `_` в разных методах: проверяйте формат символа по конкретному endpoint.

## Что важно бизнес-аналитику

- `exchange-info` и `rfq-exchange-info` это справочники доступных рынков.
- `tickers` и public WebSocket закрывают разные задачи: snapshot против потоковых обновлений.
- `history` это исторический backfill, а не low-latency feed.

## См. также

- [overview.md](overview.md)
- [../ws/public.md](../ws/public.md)
- [../flows/quickstart.md](../flows/quickstart.md)
