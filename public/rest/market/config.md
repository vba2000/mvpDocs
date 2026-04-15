# GET `/market/config`

[← Каталог `/market`](../market.md)

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
