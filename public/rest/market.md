# Публичный REST: `/market/*`

Базовый префикс **`/market`**, методы **GET**, аутентификация не нужна. Ответы в обёртке `code` / `msg` / `result` / …; в подстраницах описано поле **`result`**.

## Каталог эндпоинтов

| Путь | Назначение | Документ |
|------|------------|----------|
| `/market/config` | Конфиг графика (TradingView UDF) | [market/config.md](market/config.md) |
| `/market/symbols` | Метаданные пары | [market/symbols.md](market/symbols.md) |
| `/market/search` | Поиск символов | [market/search.md](market/search.md) |
| `/market/history` | OHLCV свечи | [market/history.md](market/history.md) |
| `/market/time` | Время сервера | [market/time.md](market/time.md) |
| `/market/exchange-info` | Спотовые рынки (exchangeInfo) | [market/exchange-info.md](market/exchange-info.md) |
| `/market/rfq-exchange-info` | RFQ-рынки | [market/rfq-exchange-info.md](market/rfq-exchange-info.md) |
| `/market/recent-trade` | Последние публичные сделки | [market/recent-trade.md](market/recent-trade.md) |
| `/market/tickers` | 24h тикеры | [market/tickers.md](market/tickers.md) |

## См. также

- [Обзор публичного REST](README.md)
- [WebSocket: рыночные потоки](../ws/README.md)
- [← Оглавление](../../README.md)
