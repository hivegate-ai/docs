# Admin API


Base path: `/admin`

All admin routes require the `X-Admin-Secret` header.

### Cache Management

#### Get Cache Stats

```bash
curl http://localhost:8000/admin/cache/stats \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "prompt_cache": {
    "size": 42,
    "maxsize": 256,
    "ttl": 300,
    "currsize": 42
  }
}
```

#### Get Cache Status

```bash
curl http://localhost:8000/admin/cache/status \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`: Returns detailed cache initialization state and statistics.

#### Invalidate Prompt Cache

```bash
# Invalidate all
curl -X POST http://localhost:8000/admin/cache/invalidate/prompts \
  -H "X-Admin-Secret: my-admin-secret"

# Invalidate specific prompt
curl -X POST "http://localhost:8000/admin/cache/invalidate/prompts?prompt_id=code-review-prompt" \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "status": "success",
  "message": "All prompt caches invalidated"
}
```

#### Reload Cache

```bash
curl -X POST http://localhost:8000/admin/cache/reload \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "status": "success",
  "message": "Cache reload initiated in background"
}
```

#### Refresh Cache

```bash
curl -X POST http://localhost:8000/admin/cache/refresh \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "status": "success",
  "message": "Cache refresh initiated in background"
}
```

### Background Tasks

#### List Background Tasks

```bash
curl http://localhost:8000/admin/background-tasks \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "tasks": {
    "prompt_cache_refresh": {
      "status": "running",
      "started_at": "2025-01-15T10:00:00"
    }
  },
  "total_tasks": 1
}
```

#### Cancel Background Task

```bash
curl -X POST http://localhost:8000/admin/background-tasks/prompt_cache_refresh/cancel \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "status": "success",
  "message": "Task 'prompt_cache_refresh' cancelled successfully"
}
```

### API Key Management

#### Create API Key

The raw API key is only returned once at creation time.

```bash
curl -X POST http://localhost:8000/admin/api-keys \
  -H "X-Admin-Secret: my-admin-secret" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Frontend",
    "owner_id": "team-frontend",
    "scopes": ["agents:read", "agents:chat"],
    "rate_limit": 5000,
    "expires_in_days": 90
  }'
```

**Response** `201 Created`:

```json
{
  "id": 1,
  "name": "Production Frontend",
  "api_key": "agk_7f3a9b2c4d5e6f1a8b9c0d1e2f3a4b5c",
  "owner_id": "team-frontend",
  "scopes": ["agents:read", "agents:chat"],
  "rate_limit": 5000,
  "expires_at": "2025-04-15T10:00:00",
  "created_at": "2025-01-15T10:00:00"
}
```

#### List API Keys

```bash
# All keys
curl http://localhost:8000/admin/api-keys \
  -H "X-Admin-Secret: my-admin-secret"

# By owner
curl "http://localhost:8000/admin/api-keys?owner_id=team-frontend" \
  -H "X-Admin-Secret: my-admin-secret"

# Include inactive
curl "http://localhost:8000/admin/api-keys?include_inactive=true" \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "api_keys": [
    {
      "id": 1,
      "name": "Production Frontend",
      "owner_id": "team-frontend",
      "scopes": ["agents:read", "agents:chat"],
      "rate_limit": 5000,
      "expires_at": "2025-04-15T10:00:00",
      "last_used_at": "2025-01-15T11:00:00",
      "created_at": "2025-01-15T10:00:00",
      "is_active": true
    }
  ],
  "total": 1
}
```

#### Get API Key

```bash
curl http://localhost:8000/admin/api-keys/1 \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`: Returns API key metadata (never the raw key).

#### Update API Key

```bash
curl -X PATCH http://localhost:8000/admin/api-keys/1 \
  -H "X-Admin-Secret: my-admin-secret" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Frontend v2",
    "rate_limit": 10000,
    "expires_in_days": 180
  }'
```

**Response** `200 OK`: Returns updated API key metadata.

#### Delete API Key

```bash
# Soft delete (deactivate)
curl -X DELETE http://localhost:8000/admin/api-keys/1 \
  -H "X-Admin-Secret: my-admin-secret"

# Permanent delete
curl -X DELETE "http://localhost:8000/admin/api-keys/1?permanent=true" \
  -H "X-Admin-Secret: my-admin-secret"
```

**Response** `200 OK`:

```json
{
  "status": "success",
  "message": "API key 1 deactivated"
}
```

---

