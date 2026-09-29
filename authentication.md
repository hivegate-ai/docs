# Authentication


### V2 Routes (`/v2/*`)

All V2 routes require an `X-API-Key` header. API keys are created via the Admin API, stored as SHA-256 hashes in PostgreSQL, and validated on each request.

```bash
curl http://localhost:8000/v2/agents \
  -H "X-API-Key: agk_abc123def456"
```

### Admin Routes (`/admin/*`)

Admin routes require an `X-Admin-Secret` header matching the `ADMIN_SECRET` environment variable.

```bash
curl http://localhost:8000/admin/cache/stats \
  -H "X-Admin-Secret: my-secret-value"
```

### Health Routes (`/health`, `/status`, `/version`, `/_/version`, `/`)

No authentication required.

### Development Mode

Set `AUTH_DISABLED=true` to bypass all authentication:

```bash
export AUTH_DISABLED=true
./scripts/start_server.sh
```

---

