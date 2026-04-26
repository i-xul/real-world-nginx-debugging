# Case 03 – WebSocket reverse proxy issue

## 🧩 Problem

Application works locally, but WebSocket connections fail when routed through Nginx.

Symptoms:

- WebSocket connection fails in browser
- Console error:
  ```
  WebSocket connection to 'ws://...' failed
  ```
- HTTP endpoints work normally

---

## 🧠 Root Cause

Nginx does not forward WebSocket upgrade headers by default.

WebSockets require:

- `Upgrade` header
- `Connection: upgrade`
- HTTP/1.1

Without these, the connection is treated as a normal HTTP request → handshake fails.

---

## ❌ Broken configuration

```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:5000;
}
```

### What happens

- Nginx forwards request as standard HTTP
- Upgrade headers are missing
- WebSocket handshake fails

---

## ✅ Fixed configuration

```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:5000;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_set_header Host $host;
}
```

---

## 🔍 Debugging process

Steps to identify the issue:

1. Check browser DevTools
   - Console → WebSocket errors
   - Network → failed WS connection

2. Compare:
   - direct connection (works)
   - proxied connection (fails)

3. Inspect headers
   - missing `Upgrade`
   - missing `Connection`

4. Review Nginx config:
   ```bash
   nginx -T
   ```

---

## 📊 Broken vs Fixed

| Config                    | Upgrade headers | Result |
|--------------------------|----------------|--------|
| default proxy            | ❌ missing      | ❌ fail |
| with WS headers          | ✅ present      | ✅ OK   |

---

## 📌 Key Takeaways

- WebSockets are not standard HTTP requests
- Nginx requires explicit upgrade handling
- Missing headers = silent failure
- Always verify protocol (HTTP vs WS)

---

## 🎯 Summary

WebSocket proxying in Nginx requires explicit configuration.

Without proper headers and HTTP/1.1 support, connections will fail even if everything else works.

This is a common issue when moving from local development to reverse proxy setups.
