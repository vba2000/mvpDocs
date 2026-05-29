# REST: market and public reference data

## Назначение

Этот раздел покрывает публичный read-only слой, который нужен до первой приватной интеграции:

- market data и справочники рынков;
- данные по blockchain networks;
- публичный snapshot orderbook;
- mark prices и RFQ market universe.

Эти методы не требуют `API key`.

## Набор методов

| Метод | Путь | Назначение |
|------|------|------------|
| `GET` | `/market/time` | Синхронизация часов клиента |
| `GET` | `/market/config` | Конфигурация charting |
| `GET` | `/market/symbols` | Карточка одного символа |
| `GET` | `/market/search` | Поиск рынков |
| `GET` | `/market/history` | OHLCV история |
| `GET` | `/market/exchange-info` | Каталог spot markets и торговых правил |
| `GET` | `/market/rfq-exchange-info` | Каталог RFQ markets |
| `GET` | `/market/recent-trade` | Последние публичные сделки |
| `GET` | `/market/tickers` | 24h snapshot по рынкам |
| `GET` | `/market/mark-prices` | Текущие mark prices |
| `GET` | `/network` | Публичный список сетей и активов |
| `GET` | `/spot/orderbook` | Публичный snapshot стакана по рынку |

## Рекомендуемый порядок использования

1. `GET /market/time`
2. `GET /market/exchange-info`
3. `GET /market/symbols`
4. `GET /network`
5. `GET /market/tickers` и public WebSocket

## Ключевые методы

### `GET /market/time`

Используется для выравнивания часов клиента перед приватными REST и private WebSocket запросами.

Фактический ответ содержит три поля:

- `timeSec`
- `timeMs`
- `timeNano`

Для HMAC-интеграции обычно используется `timeMs`.

### `GET /market/config`

Возвращает конфигурацию charting-совместимого market data слоя.

Ключевые поля `ConfigDto`:

- `supportedResolutions`
- `supportsGroupRequest`
- `supportsMarks`
- `supportsSearch`
- `supportsTimescaleMarks`
- `supportsTime`

Даже если клиент не строит UI-графики напрямую, этот метод полезен как справочник допустимых resolution для history/charting use cases.

### `GET /market/exchange-info`

Главная точка входа для spot-интеграции.

Метод возвращает по рынкам:

- `symbol`, `status`, `baseAsset`, `quoteAsset`;
- precision и шаги цены/объема;
- `tickSize`, `lotSize`, `minQty`, `maxQty`, `minNotional`;
- поддерживаемые `orderTypes`;
- поддерживаемые `timeInForceModes`;
- `executionFlags`;
- защитные ограничения по price protection.

Именно этот метод нужен, чтобы валидировать ордера до отправки.

Как читать поля рынка:

| Поле | Значение для интегратора |
|------|--------------------------|
| `symbol` | market id вида `BTC_USDT` |
| `status` | текущая торговая доступность рынка |
| `baseAssetPrecision` / `quoteAssetPrecision` | точность отображения активов |
| `pricePrecision` / `qtyPrecision` | точность торговых полей |
| `tickSize` / `lotSize` | минимальные шаги цены и количества |
| `minQty` / `maxQty` | границы количества |
| `minNotional` | нижняя граница стоимости ордера |
| `orderTypes` | допустимые типы ордеров |
| `timeInForceModes` | допустимые режимы исполнения |
| `executionFlags` | специальные execution-возможности рынка |
| `limit*PercentProtection` / `market*PercentProtection` | защитные price bounds |

### `GET /market/rfq-exchange-info`

Справочник RFQ рынков. Нужен до вызовов `/rfq/*`, чтобы проверить:

- есть ли рынок в RFQ universe;
- какие precision и статусы действуют для него.

### `GET /market/symbols`

Query:

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | да | Символ вида `BTC/USDT` |

Возвращает карточку инструмента для chart-интеграции.

Важно: этот endpoint использует формат символа вида `BTC/USDT`, а большая часть торговых и WS-сценариев работает с `BTC_USDT`.

### `GET /market/search`

Используется в symbol picker и поисковых формах.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `query` | string | да | Строка поиска |
| `type` | string | нет | Фильтр типа |
| `exchange` | string | нет | Фильтр площадки |
| `limit` | integer | нет | Максимум результатов |

### `GET /market/history`

Backfill OHLCV свечей.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | да | Символ вида `BTC/USDT` |
| `resolution` | string | да | Таймфрейм |
| `from` | integer | да | Начало диапазона в Unix seconds |
| `to` | integer | да | Конец диапазона в Unix seconds |
| `limit` | integer | нет | Максимум свечей |

Важно: `from` и `to` задаются в секундах, не в миллисекундах.

### `GET /market/tickers`

24h snapshot для UI, market lists и initial market state.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `category` | string | нет | Категория рынка |
| `symbol` | string | нет | Один рынок |
| `baseCoin` | string | нет | Фильтр по базовому активу |

### `GET /market/recent-trade`

Начальная подкачка recent trades перед переходом на public WebSocket.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | нет | Например `BTC_USDT` |
| `limit` | integer | нет | До 100 записей |

### `GET /market/mark-prices`

Возвращает текущие mark prices.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `market` | string | нет | Фильтр по одному рынку |

Метод полезен для аналитики и риск-контроля; это отдельный слой данных, не заменяющий торговый ticker.

### `GET /network`

Публичный список blockchain networks и их активов.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `network` | string | нет | Фильтр по shortname сети |
| `asset` | string | нет | Фильтр по активу |

Этот метод нужен до депозитных и funding-сценариев.

### `GET /spot/orderbook`

Публичный snapshot стакана по рынку.

| Параметр | Тип | Обязателен | Назначение |
|----------|-----|------------|------------|
| `symbol` | string | да | Рынок, например `BTC_USDT` |
| `step` | integer | нет | Агрегация цены по `10^step` |
| `size` | integer | нет | Количество уровней на сторону, `0` для полного cached snapshot |

Это REST snapshot, а не поток. Он нужен:

- для initial book load;
- для восстановления состояния после разрыва public WS;
- для low-frequency UI use cases.

## Что важно бизнес-аналитику

- `exchange-info` и `rfq-exchange-info` это справочники доступных рынков и правил, а не просто каталог символов.
- Символ `BTC/USDT` и market id `BTC_USDT` не взаимозаменяемы без учета контекста endpoint.
- `tickers` и public WebSocket решают разные задачи: snapshot против live feed.
- `spot/orderbook` нужен как точка восстановления, а не как замена WS.
- `mark-prices` и торговые tickers нельзя считать одним и тем же источником цены.

## См. также

- [overview.md](overview.md)
- [../concepts.md](../concepts.md)
- [../ws/public.md](../ws/public.md)
- [../flows/quickstart.md](../flows/quickstart.md)
