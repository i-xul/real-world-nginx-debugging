# Case 04 – Host header misconfiguration

## Header flow

Client (Host: example.com)
  ↓
Nginx
  ↓
App

Broken:
Host: 127.0.0.1 ❌

Fixed:
Host: example.com ✅

## 🧩 Problem

Application behaves differently when accessed through Nginx reverse proxy.

Symptoms:

- redirects point to wrong domain (e.g. `127.0.0.1` instead of real hostname)
- absolute URLs are incorrect
- authentication / callbacks may fail
- app works locally but breaks behind proxy

---

## 🧠 Root Cause

Nginx does not automatically forward the original `Host` header.

By default, the upstream application may receive:

```
Host: 127.0.0.1:5000
```

instead of:

```
Host: example.com
```

Many applications rely on the Host header to:

- generate URLs
- validate requests
- handle redirects
- enforce security rules

---

## ❌ Broken configuration

```nginx
location / {
    proxy_pass http://127.0.0.1:5000;
}
```

### What happens

- upstream sees incorrect Host
- app generates wrong URLs
- redirects point to internal address

---

## ✅ Fixed configuration

```nginx
location / {
    proxy_pass http://127.0.0.1:5000;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

---

## 🔍 Debugging process

Steps to identify the issue:

1. Inspect redirects
   - look for wrong domain / IP in Location headers

2. Check application logs
   - what Host does backend receive?

3. Use curl:
   ```bash
   curl -I http://example.com
   ```

4. Compare:
   - direct app access (correct)
   - proxied access (incorrect)

5. Review Nginx config:
   ```bash
   nginx -T
   ```

---

## 📊 Broken vs Fixed

| Header                  | Broken           | Fixed          |
|------------------------|------------------|----------------|
| Host                   | 127.0.0.1        | example.com    |
| X-Forwarded-For        | ❌ missing        | ✅ present      |
| X-Forwarded-Proto      | ❌ missing        | ✅ present      |

---

## 📌 Key Takeaways

- backend applications rely on request headers
- reverse proxies must forward critical headers
- missing Host header causes subtle and confusing bugs
- X-Forwarded-* headers are essential in real deployments

---

## 🎯 Summary

Incorrect or missing headers in reverse proxy setups can break application behavior in non-obvious ways.

Always ensure that:

- Host header is preserved
- client IP is forwarded
- protocol (HTTP/HTTPS) is correctly indicated

This is a common issue in production environments and affects many frameworks and applications.
