# Targets API


Base path: `/v2/targets`

### Create Target

```bash
curl -X POST http://localhost:8000/v2/targets \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "linux-pool-1",
    "name": "Linux Worker Pool",
    "type": "ssh",
    "connection_config": {
      "host": "worker-1.internal.acme.com",
      "port": 22,
      "ssh_key_path": "/secrets/worker-key.pem",
      "user": "worker"
    },
    "capacity": {
      "max_concurrent_jobs": 4,
      "memory_mb": 16384,
      "cpus": 8
    },
    "worker_pool": "linux_worker_pool"
  }'
```

**Request Body** (`TargetCreateRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique target identifier |
| `name` | string | Yes | Human-readable name |
| `type` | string | Yes | `local`, `ssh`, `remote_service`, or `managed_agents` |
| `connection_config` | object | No | Connection details (host, port, SSH key, API URL) |
| `capacity` | object | No | Resource capacity |
| `worker_pool` | string | No (default: "linux_worker_pool") | Worker pool name |

**Response** `201 Created`:

```json
{
  "id": "linux-pool-1",
  "name": "Linux Worker Pool",
  "type": "ssh",
  "connection_config": {
    "host": "worker-1.internal.acme.com",
    "port": 22
  },
  "capacity": { "max_concurrent_jobs": 4, "memory_mb": 16384 },
  "worker_pool": "linux_worker_pool",
  "is_active": true
}
```

### List Targets

```bash
curl http://localhost:8000/v2/targets \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns `TargetResponse[]`.

### Get Target

```bash
curl http://localhost:8000/v2/targets/linux-pool-1 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns `TargetResponse`.

### Update Target

```bash
curl -X PUT http://localhost:8000/v2/targets/linux-pool-1 \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "linux-pool-1",
    "name": "Linux Worker Pool (Updated)",
    "type": "ssh",
    "connection_config": { "host": "worker-2.internal.acme.com", "port": 22 },
    "capacity": { "max_concurrent_jobs": 8, "memory_mb": 32768 },
    "worker_pool": "linux_worker_pool"
  }'
```

**Response** `200 OK`: Returns updated `TargetResponse`.

### Delete Target

```bash
curl -X DELETE http://localhost:8000/v2/targets/linux-pool-1 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "linux-pool-1",
  "message": "Target deleted"
}
```

### Target Health Check

```bash
curl http://localhost:8000/v2/targets/linux-pool-1/health \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "target_id": "linux-pool-1",
  "status": "unknown",
  "message": "Health check not yet implemented for this target type"
}
```

---

