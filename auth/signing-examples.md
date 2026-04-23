# Примеры подписи и запросов

## Python: REST GET

```python
import base64
import hashlib
import hmac
import time
import urllib.request


def cex_signature(secret: str, timestamp_ms: int, method: str, path_with_query: str, body: str = "") -> str:
    payload = f"{timestamp_ms}{method.upper()}{path_with_query}{body}"
    digest = hmac.new(secret.encode("utf-8"), payload.encode("utf-8"), hashlib.sha256).digest()
    return base64.b64encode(digest).decode("ascii")


api_key = "YOUR_API_KEY"
secret = "YOUR_SECRET"
base_url = "https://cex-test.web3tech.ru"
path = "/api/v1/gateway/market/time"
timestamp_ms = int(time.time() * 1000)

signature = cex_signature(secret, timestamp_ms, "GET", path, "")

request = urllib.request.Request(
    f"{base_url}{path}",
    method="GET",
    headers={
        "X-API-Key": api_key,
        "X-Timestamp": str(timestamp_ms),
        "X-Signature": signature,
    },
)
```

## Python: REST POST

```python
import json

path = "/api/v1/gateway/spot/orders/create"
body = json.dumps(
    {
        "marketId": "BTC_USDT",
        "baseAssetId": "BTC",
        "quoteAssetId": "USDT",
        "side": "BUY",
        "orderType": "LIMIT",
        "timeInForce": "GTC",
        "quantity": "0.001",
        "price": "50000",
        "clientOrderId": "bot-order-0001",
    },
    separators=(",", ":"),
)

timestamp_ms = int(time.time() * 1000)
signature = cex_signature(secret, timestamp_ms, "POST", path, body)

request = urllib.request.Request(
    f"{base_url}{path}",
    method="POST",
    data=body.encode("utf-8"),
    headers={
        "X-API-Key": api_key,
        "X-Timestamp": str(timestamp_ms),
        "X-Signature": signature,
        "Content-Type": "application/json",
    },
)
```

## JavaScript: REST

```javascript
import crypto from "crypto";

function cexSignature(secret, timestampMs, method, pathWithQuery, body = "") {
  const payload = `${timestampMs}${method.toUpperCase()}${pathWithQuery}${body}`;
  return crypto.createHmac("sha256", secret).update(payload, "utf8").digest("base64");
}

const apiKey = "YOUR_API_KEY";
const secret = "YOUR_SECRET";
const baseUrl = "https://cex-test.web3tech.ru";
const path = "/api/v1/gateway/accounts/balance";
const timestampMs = Date.now();

const signature = cexSignature(secret, timestampMs, "GET", path, "");

await fetch(`${baseUrl}${path}`, {
  method: "GET",
  headers: {
    "X-API-Key": apiKey,
    "X-Timestamp": String(timestampMs),
    "X-Signature": signature,
  },
});
```

## JavaScript: private WebSocket

```javascript
import crypto from "crypto";
import WebSocket from "ws";

const apiKey = "YOUR_API_KEY";
const secret = "YOUR_SECRET";
const timestampMs = Date.now();
const payload = `${timestampMs}WEBSOCKET/wsWS_CONNECT`;
const signature = crypto.createHmac("sha256", secret).update(payload, "utf8").digest("base64");

const ws = new WebSocket(
  "wss://cex-test.web3tech.ru/api/v1/ws-private/ws/private?format=json",
  {
    headers: {
      "X-API-Key": apiKey,
      "X-Timestamp": String(timestampMs),
      "X-Signature": signature,
    },
  }
);

ws.on("message", (data) => {
  console.log(data.toString());
});
```

## JSON subscribe после подключения

```json
{
  "action": "subscribe",
  "streams": ["user"]
}
```

## Практические правила

- Для REST всегда подписывайте путь с `/api/v1/gateway`.
- Для private WebSocket никогда не подписывайте `/ws/private`.
- Все цены и количества сериализуйте как строки.
- Один и тот же JSON должен использоваться и для HMAC, и для HTTP body.
