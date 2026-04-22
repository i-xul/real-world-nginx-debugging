## How to run

```bash
docker-compose up --build
```

Then open:

http://localhost:8080/dashboard/

## Lessons learned

- Flask assumes root (`/`) by default
- Subpath deployments require explicit static path handling
- Nginx `proxy_pass` behavior depends on trailing slash
- Always test both upstream and proxy paths
