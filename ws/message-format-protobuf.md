# WebSocket protobuf / binary format

## Назначение

`format=binary` это основной transport format по умолчанию. Он использует protobuf-сообщения и подходит для высоконагруженной интеграции.

## Когда использовать

- нужен компактный и стабильный контракт;
- требуется минимизировать overhead сериализации;
- клиент уже умеет работать с protobuf;
- важны массовые public market streams.

## Общая модель

На wire-уровне клиент и сервер обмениваются protobuf frames. Логика сообщений соответствует тем же действиям, что и в JSON-режиме:

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

## Актуальные proto-файлы

В репозитории документации лежат два самостоятельных контракта:

- `protos/ws_public.proto` для public WebSocket;
- `protos/ws_private.proto` для private WebSocket.

Оба файла уже содержат общий envelope и базовые сообщения, поэтому их можно использовать независимо друг от друга.

## Рекомендация

Если задача команды это production market data feed или большой поток событий, выбирайте `binary`. Если важнее простота запуска и ручная отладка, используйте `json`.

## См. также

- [message-format-json.md](message-format-json.md)
- [overview.md](overview.md)
