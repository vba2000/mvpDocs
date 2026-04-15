# Публичный REST: `/market/*`

Базовый префикс: **`/market`**. Методы: **GET**. Аутентификация не требуется.

Ответы обёрнуты в общую структуру (`code`, `msg`, `result`, …); ниже описано поле **`result`**.

---

## GET `/market/config`

**Назначение:** конфигурация графика (совместимость с TradingView UDF): доступные разрешения, таймзона, флаги возможностей.

**Параметры:** нет.

**Ответ `result`:** `ConfigDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `supportedResolutions` | string[] | Допустимые интервалы свечей |
| `supportsGroupRequest` | boolean | Групповые запросы |
| `supportsMarks` | boolean | Метки на графике |
| `supportsSearch` | boolean | Поиск символов |
| `supportsTimescaleMarks` | boolean | Метки шкалы времени |
| `supportsTime` | boolean | Поддержка времени сервера |

---

## GET `/market/symbols`

**Назначение:** метаданные торговой пары для графика.

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `symbol` | да | string | Символ, напр. `BTC/USDT` |

**Ответ `result`:** `SymbolInfoDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `name`, `ticker`, `description`, `type`, `session`, `timezone`, `exchange` | string | Идентификация и отображение |
| `minmov`, `pricescale` | int | Шаг цены (UDF) |
| `hasIntraday`, `hasWeeklyAndMonthly` | boolean | Доступность интервалов |
| `supportedResolutions` | string[] | Разрешения для символа |
| `volumePrecision` | int | Точность объёма |
| `dataStatus` | string | Статус данных |

---

## GET `/market/search`

**Назначение:** поиск пар для автодополнения.

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `query` | да | string | Строка поиска, напр. `BTC` |
| `type` | нет | string | Фильтр типа символа |
| `exchange` | нет | string | Фильтр биржи |
| `limit` | нет | int | Макс. число результатов |

**Ответ `result`:** `SearchSymbolsDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `result` | array | Список `SymbolSearchResultDto` |

`SymbolSearchResultDto`: `symbol`, `fullName`, `description`, `exchange`, `ticker`, `type`.

---

## GET `/market/history`

**Назначение:** история OHLCV (свечи).

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `symbol` | да | string | Пара, напр. `BTC/USDT` |
| `resolution` | да | string | Интервал: `1`, `5`, `15`, `60`, `1D`, `1W`, … |
| `from` | да | long | Начало, Unix **секунды** |
| `to` | да | long | Конец, Unix **секунды** |
| `limit` | нет | int | Макс. свечей (по умолчанию до 5000) |

**Ответ `result`:** `HistoryDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `status` | string | Статус выборки (UDF) |
| `t` | long[] | Время открытия свечи, Unix сек |
| `o`, `h`, `l`, `c` | double[] | Open, high, low, close |
| `v` | double[] | Объём |

---

## GET `/market/time`

**Назначение:** время сервера для синхронизации часов.

**Параметры:** нет.

**Ответ `result`:** `ServerTimeDto` — поле `time` (long, обычно epoch ms или согласно реализации контроллера; уточнить по OpenAPI при интеграции).

---

## GET `/market/exchange-info`

**Назначение:** список спотовых рынков в стиле Binance `exchangeInfo`: точность, статус, типы ордеров.

**Параметры:** нет.

**Ответ `result`:** `SpotExchangeInfoDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `markets` | array | `SpotMarketDto` |

`SpotMarketDto`: `symbol`, `status`, `baseAsset`, `quoteAsset`, `baseAssetPrecision`, `quoteAssetPrecision`, `orderTypes`.

---

## GET `/market/rfq-exchange-info`

**Назначение:** список RFQ-рынков.

**Параметры:** нет.

**Ответ `result`:** `RfqExchangeInfoDto` — `markets[]` как `RfqMarketDto` (`symbol`, `status`, `baseAsset`, `quoteAsset`, точности).

---

## GET `/market/recent-trade`

**Назначение:** последние публичные сделки (до 100).

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `symbol` | нет | string | Фильтр пары, напр. `BTC_USDT` |
| `limit` | нет | int | 1…100, по умолчанию 60 |

**Ответ `result`:** `RecentTradesDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `list` | array | `RecentTradeItemDto` |

Элемент: `execId`, `symbol`, `price`, `size`, `side`, `time`.

---

## GET `/market/tickers`

**Назначение:** агрегированная 24h-статистика по спотовым рынкам (кэшируется).

| Параметр | Обяз. | Тип | Описание |
|----------|-------|-----|----------|
| `category` | нет | string | Категория, напр. `spot` |
| `symbol` | нет | string | Одна пара, напр. `BTC_USDT` |
| `baseCoin` | нет | string | Фильтр базового актива, напр. `USDT` |

**Ответ `result`:** `TickersResponseDto`

| Поле | Тип | Описание |
|------|-----|----------|
| `category` | string | Категория выборки |
| `list` | array | `TickerItemDto` |

`TickerItemDto`: `symbol`, `bid1Price`, `bid1Size`, `ask1Price`, `ask1Size`, `lastPrice`, `prevPrice24h`, `price24hPcnt`, `highPrice24h`, `lowPrice24h`, `turnover24h`, `volume24h`.

---

## См. также

- [Обзор публичного REST](README.md)
- [WebSocket: те же рынки в потоках](../ws/README.md)
- [← Оглавление](../../README.md)
