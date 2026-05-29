# WebSocket protobuf / binary format

## Назначение

`format=binary` это основной transport format по умолчанию. Он использует protobuf-сообщения и подходит для высоконагруженной интеграции.

## Когда использовать

- нужен компактный и стабильный контракт;
- требуется минимизировать overhead сериализации;
- клиент уже умеет работать с protobuf;
- важны массовые public market streams.

## Общая модель

На wire-уровне клиент и сервер обмениваются protobuf frames.

Модель разделена на три файла:

- `protos/ws_common.proto`
- `protos/ws_public.proto`
- `protos/ws_private.proto`

Роли файлов:

| Файл | Назначение |
|------|------------|
| `ws_common.proto` | общий server envelope, control messages, error enum, price level |
| `ws_public.proto` | public client message и market payloads |
| `ws_private.proto` | private client message и user/trading/RFQ payloads |

## Wire model

### Client -> Server

- public socket принимает `PublicWsClientMessage`;
- private socket принимает `PrivateWsClientMessage`.

Это два разных wrapper type. Клиент не должен пытаться отправлять один и тот же бинарный frame на оба сокета.

### Server -> Client

Оба сокета отдают `WsEnvelope`, внутри которого лежит:

- `subscribed`
- `unsubscribed`
- `ping`
- `data`

`DataEvent.data` это opaque `bytes`, которые клиент декодирует уже как конкретный payload по типу стрима.

Именно поэтому binary-клиенту нужны одновременно:

1. knowledge о stream name;
2. mapping stream -> protobuf payload type;
3. общий parser для `WsEnvelope`.

## Общая логика действий

Логика действий совпадает с JSON-режимом:

- `subscribe`
- `unsubscribe`
- `pong`
- private commands для торгового контура

## Что остается одинаковым между binary и json

- те же public/private URL;
- те же stream names;
- те же права доступа;
- та же бизнес-семантика команд и событий;
- тот же heartbeat.

## Что меняется

- payload кодируется protobuf-сообщениями;
- не нужно вручную учитывать stringified numbers в JSON;
- для декодирования нужны `.proto` контракты и сгенерированные модели клиента.

## Codegen expectations

- `ws_common.proto` должен генерироваться вместе с `ws_public.proto` и `ws_private.proto`;
- `ws_public.proto` и `ws_private.proto` импортируют общий файл, поэтому isolated generation только одного файла без import resolution приведет к проблемам;
- для server-to-client decoding сначала разбирается `WsEnvelope`, а затем его `data` поле.

## Public payloads

Основные типы public data:

- `OrderbookSnapshotProto`
- `OrderbookDeltaProto`
- `OrderbookDeltaBatchProto`
- `TradeProto`
- `TradeBatchProto`
- `TickerProto`
- `CandleProto`
- `AggregatePriceProto`
- `MarkPriceProto`

Для orderbook в актуальном контракте присутствует `matcher_seq`, а trades в live-stream чаще приходят как `TradeBatchProto`.

## Private payloads

Основные типы private data:

- `UserEventBatchProto`
- `UserNotificationBatchProto`
- `RfqEventProto`
- `MarketsAvailableProto`

В private сокете есть и client commands:

- `create_order`
- `cancel_order`
- `batch_create_order`
- `batch_cancel_order`
- `cancel_all_orders`
- `create_rfq`

## Что важно для клиента

- Public и private client frames это разные wrapper types.
- Общий envelope вынесен в `ws_common.proto`.
- Для private binary-интеграции нужно поддержать не только `user`, но и `notifications`, если клиент хочет live-статусы депозитов.

## Рекомендация

Если задача команды это production market data feed или большой поток событий, выбирайте `binary`. Если важнее простота запуска и ручная отладка, используйте `json`.

## См. также

- [message-format-json.md](message-format-json.md)
- [overview.md](overview.md)
