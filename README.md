# Real-World Nginx Debugging

Practical debugging cases from real self-hosted environments.

Focus:

* Nginx reverse proxy
* Flask / uWSGI apps
* Subpath deployments
* Static file issues
* Production-like troubleshooting

---

## Case 1 – Flask static files returning 404 behind Nginx subpath

### 🧩 Problem

Flask application works correctly when accessed locally:

```
http://127.0.0.1:5002/static/style.css
```

But fails when accessed through Nginx reverse proxy under a subpath:

```
https://example.com/dashboard/static/style.css → 404
```

---

### 🧠 Root Cause

Mismatch between:

* Flask static file configuration
* Nginx reverse proxy configuration
* Deployment under subpath (`/dashboard`)

Flask serves static files relative to root (`/static/...`), but the app is exposed under `/dashboard`, causing incorrect path resolution.

---

### 🔍 Debugging Process

Steps used to identify the issue:

1. **Test local app directly**

   ```bash
   curl http://127.0.0.1:5002/static/style.css
   ```

2. **Test via Nginx**

   ```bash
   curl https://example.com/dashboard/static/style.css
   ```

3. **Inspect Nginx configuration**

   ```bash
   nginx -T
   ```

4. **Check browser DevTools**

   * Network tab → 404 responses
   * Requested paths mismatch

---

### ⚙️ Solution

Ensure consistency between:

* Flask static path
* Nginx location blocks
* Subpath routing

#### Option A – Fix Flask config

```python
app = Flask(
    __name__,
    static_url_path="/dashboard/static"
)
```

#### Option B – Fix Nginx config

Example:

```nginx
location /dashboard/ {
    proxy_pass http://127.0.0.1:5002/;
}

location /dashboard/static/ {
    proxy_pass http://127.0.0.1:5002/static/;
}
```

---

### ✅ Result

Static files load correctly:

```
https://example.com/dashboard/static/style.css → 200 OK
```

---

### 📌 Key Takeaways

* Subpath deployments require explicit path alignment
* Flask defaults assume root (`/`), not subpaths
* Nginx `proxy_pass` behavior (trailing slash!) is critical
* Always test both:

  * upstream (localhost)
  * external (through proxy)

---

## Cases

- [Case 01 – Flask static files returning 404 behind Nginx subpath](./cases/case-01-flask-static-404)
- [Case 02 – proxy_pass trailing slash bug](./cases/case-02-proxy-pass-trailing-slash)
- [Case 03 – WebSocket reverse proxy issue](./cases/case-03-websocket-reverse-proxy)
- [Case 04 – Host header misconfiguration](./cases/case-04-host-header-misconfiguration)

---

## 🚀 Future Cases

Planned additions:

* Trailing slash issues in `proxy_pass`
* WebSocket proxy problems
* Host header misconfiguration
* uWSGI / socket vs HTTP mismatch
* Caching & stale assets

---

## 🧱 Environment

Typical stack used in these cases (tested in real self-hosted setups):

* Nginx
* Flask
* uWSGI
* Raspberry Pi / Ubuntu server
* Self-hosted setup

---

## 🎯 Purpose

This repository is not a tutorial.

It is a collection of **real debugging scenarios**, focusing on:

* identifying root causes
* understanding system behavior
* fixing production-like issues

---

## 📄 License

MIT (or your choice)
