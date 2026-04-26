# Case 02 – proxy_pass trailing slash bug

## 🧩 Problem

Nginx reverse proxy works, but routes behave unexpectedly.

Example:

- `/dashboard` works
- `/dashboard/static/...` returns incorrect paths or 404

---

## 🧠 Root Cause

The behavior of `proxy_pass` in Nginx depends on whether the upstream URL ends with a trailing slash.

This small difference changes how request URIs are forwarded.

---

## ❌ Broken configuration

```nginx
location /dashboard/ {
    proxy_pass http://127.0.0.1:5000;
}
```

### What happens

Request:
```
/dashboard/static/style.css
```

Forwarded upstream as:
```
/dashboard/static/style.css
```

👉 The `/dashboard` prefix is NOT stripped  
👉 Backend does not expect this → leads to 404

---

## ✅ Fixed configuration

```nginx
location /dashboard/ {
    proxy_pass http://127.0.0.1:5000/;
}
```

### What happens

Request:
```
/dashboard/static/style.css
```

Forwarded upstream as:
```
/static/style.css
```

👉 `/dashboard` prefix is removed  
👉 Matches backend routes correctly

---

## 🔍 Debugging process

Steps to identify the issue:

1. Compare working vs failing routes
2. Use browser DevTools → Network tab
3. Check upstream logs (Flask / app logs)
4. Inspect Nginx config:
   ```bash
   nginx -T
   ```

---

## 📊 Broken vs Fixed

| Config                        | Upstream path                  | Result |
|-----------------------------|-------------------------------|--------|
| `proxy_pass http://app`     | `/dashboard/static/...`       | ❌ 404 |
| `proxy_pass http://app/`    | `/static/...`                 | ✅ OK  |

---

## 📌 Key Takeaways

- `proxy_pass` trailing slash changes request rewriting behavior
- Without `/` → full path is forwarded
- With `/` → matching location prefix is stripped
- This is one of the most common Nginx misconfigurations

---

## 🎯 Summary

A single missing `/` in `proxy_pass` can break routing entirely.

Always verify:

- expected upstream paths
- how Nginx rewrites URIs
- consistency between proxy and backend routes
