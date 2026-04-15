# REST: API-ключ и HMAC-подпись

## Назначение

Позволяет серверным и клиентским приложениям вызывать защищённые методы REST **без** интерактивного логина: передаются идентификатор ключа и доказательство владения секретом (HMAC).

## Заголовки

| Заголовок | Обязательность | Описание |
|-----------|----------------|----------|
| `X-API-Key` | Да (для режима ключа) | Публичный идентификатор ключа |
| `X-Signature` | Да | Подпись, **Base64** (HMAC-SHA256) |
| `X-Timestamp` | Да | Unix-время в **миллисекундах** |

Если `X-API-Key` **нет**, Gateway может аутентифицировать запрос по **сессии** (см. [rest-session.md](rest-session.md)).

## Строка для подписи

Секрет ключа никогда не передаётся в запросе. Подпись строится так же, как в `HmacSignature` authorization-provider:

```
message = "{timestamp}{METHOD}{path}{body}"
signature = Base64( HMAC_SHA256(secret, UTF8(message)) )
```

Где:

- **`timestamp`** — то же целое число, что в `X-Timestamp` (строка при конкатенации — десятичное представление без пробелов).
- **`METHOD`** — HTTP-метод **заглавными** буквами (`GET`, `POST`, …).
- **`path`** — `requestURI` **включая** query string: если query есть, формат `path?query` (как в сервлетном API после нормализации Gateway).
- **`body`** — **сырое** тело запроса как строка UTF-8; для GET/DELETE без тела — **пустая строка** `""`.

Пример для `GET` без query:

- `timestamp = 1713200000123`
- `message = "1713200000123GET/market/time"`

Пример для `GET` с query `symbol=BTC%2FUSDT`:

- `path = "/market/symbols?symbol=BTC%2FUSDT"` (как приходит в подписи на стороне сервера)
- `body = ""`

**Важно про `path`:** в подпись входит **точно тот путь с query**, который уходит в HTTP-запросе после хоста (включая префикс приложения, если он есть в URI, например `/api/v1/gateway/...`). Убедитесь, что кодировка query (`%2F` и т.д.) совпадает с реальным запросом.

## Примеры кода: формирование подписи

### Python (REST)

```python
import base64
import hashlib
import hmac
import time
import urllib.request


def cex_hmac_signature(secret: str, timestamp_ms: int, method: str, path_with_query: str, body: str = "") -> str:
    """path_with_query — путь + ?query как в URL; body — сырая строка тела (для GET пустая)."""
    msg = f"{timestamp_ms}{method.upper()}{path_with_query}{body}"
    digest = hmac.new(secret.encode("utf-8"), msg.encode("utf-8"), hashlib.sha256).digest()
    return base64.b64encode(digest).decode("ascii")


# --- GET: /market/time (подставьте свой префикс в path, если он есть в URI) ---
api_key = "YOUR_API_KEY_ID"
secret = "YOUR_API_SECRET"
base = "https://cex-dev.web3tech.ru"
path = "/api/v1/gateway/market/time"  # как в запросе после хоста
ts = int(time.time() * 1000)
sig = cex_hmac_signature(secret, ts, "GET", path, "")

req = urllib.request.Request(
    f"{base}{path}",
    headers={
        "X-API-Key": api_key,
        "X-Timestamp": str(ts),
        "X-Signature": sig,
    },
    method="GET",
)
# urllib.request.urlopen(req)

# --- POST: JSON-тело; строка для подписи == ровно те байты, что в теле запроса ---
import json

path_post = "/api/v1/gateway/spot/orders/create"
payload = {
    "marketId": "BTC_USDT",
    "baseAssetId": "BTC",
    "quoteAssetId": "USDT",
    "side": "BUY",
    "orderType": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.001",
    "price": "50000",
}
body_str = json.dumps(payload, separators=(",", ":"), ensure_ascii=False)
ts2 = int(time.time() * 1000)
sig2 = cex_hmac_signature(secret, ts2, "POST", path_post, body_str)

req2 = urllib.request.Request(
    f"{base}{path_post}",
    data=body_str.encode("utf-8"),
    headers={
        "X-API-Key": api_key,
        "X-Timestamp": str(ts2),
        "X-Signature": sig2,
        "Content-Type": "application/json",
    },
    method="POST",
)
# urllib.request.urlopen(req2)
```

Для `GET` с query используйте тот же `path_with_query`, что в URL (например `"/api/v1/gateway/market/symbols?symbol=BTC%2FUSDT"`).

### JavaScript (REST, Node.js)

Секрет не храните во фронтенде; пример для сервера или скрипта.

```javascript
import crypto from "crypto";

function cexHmacSignature(secret, timestampMs, method, pathWithQuery, body = "") {
  const msg = `${timestampMs}${method.toUpperCase()}${pathWithQuery}${body}`;
  return crypto.createHmac("sha256", secret).update(msg, "utf8").digest("base64");
}

const apiKey = "YOUR_API_KEY_ID";
const secret = "YOUR_API_SECRET";
const base = "https://cex-dev.web3tech.ru";

// GET
const pathGet = "/api/v1/gateway/market/time";
const ts = Date.now();
const sigGet = cexHmacSignature(secret, ts, "GET", pathGet, "");

await fetch(`${base}${pathGet}`, {
  method: "GET",
  headers: {
    "X-API-Key": apiKey,
    "X-Timestamp": String(ts),
    "X-Signature": sigGet,
  },
});

// POST: body для подписи и для fetch должен совпадать побайтно
const pathPost = "/api/v1/gateway/spot/orders/create";
const bodyStr = JSON.stringify({
  marketId: "BTC_USDT",
  baseAssetId: "BTC",
  quoteAssetId: "USDT",
  side: "BUY",
  orderType: "LIMIT",
  timeInForce: "GTC",
  quantity: "0.001",
  price: "50000",
});
const ts2 = Date.now();
const sigPost = cexHmacSignature(secret, ts2, "POST", pathPost, bodyStr);

await fetch(`${base}${pathPost}`, {
  method: "POST",
  headers: {
    "X-API-Key": apiKey,
    "X-Timestamp": String(ts2),
    "X-Signature": sigPost,
    "Content-Type": "application/json",
  },
  body: bodyStr,
});
```

### WebSocket handshake (API key)

Строка **фиксирована**: литералы склеиваются как `WEBSOCKET` + `/ws` + `WS_CONNECT` без слэша между `/ws` и `WS_CONNECT`, итого подстановка в шаблон: `{timestamp}WEBSOCKET/wsWS_CONNECT`.

**Python:**

```python
ts = int(time.time() * 1000)
ws_msg = f"{ts}WEBSOCKET/wsWS_CONNECT"
ws_sig = base64.b64encode(
    hmac.new(secret.encode("utf-8"), ws_msg.encode("utf-8"), hashlib.sha256).digest()
).decode("ascii")
# Заголовки рукопожатия: X-API-Key, X-Timestamp=str(ts), X-Signature=ws_sig
```

**JavaScript (Node.js):**

```javascript
const tsWs = Date.now();
const wsMsg = `${tsWs}WEBSOCKET/wsWS_CONNECT`;
const wsSig = crypto.createHmac("sha256", secret).update(wsMsg, "utf8").digest("base64");
// передать в заголовках WebSocket-клиента при connect
```

Подробнее про WS: [websocket.md](websocket.md).

## IP клиента

На сторону проверки ключа передаётся IP: первый элемент из `X-Forwarded-For`, иначе адрес соединения. Это используется для опционального whitelist в настройках ключа.

## Публичные маршруты и API-ключ

Для путей из списка публичных (`/market/**`, часть `/auth/*`, swagger и т.д.) фильтр API-ключа **не** обрабатывает запрос как обязательную подпись: ключ можно не слать. Если ключ передан на публичный путь, поведение зависит от реализации фильтра (в коде такие пути **исключены** из фильтра API-ключа — запрос идёт без проверки ключа).

## Ошибки

При неверной подписи, просроченном времени или отклонении ключа Gateway отвечает **401** с JSON в стиле обёртки API (`extInfo` с деталями).

## Создание и отзыв ключей

Выполняется через сессию: `POST/GET/DELETE /auth/api-keys` (требуется аутентификация пользователя). Секрет показывается только при создании.

## См. также

- [WebSocket: подпись отличается](websocket.md)
- [Сессия REST](rest-session.md)
- [← Обзор авторизации](README.md)
