# Engines API


Base path: `/v2/engines`

### Create Engine

```bash
curl -X POST http://localhost:8000/v2/engines \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "claude-code-v1",
    "name": "Claude Code Engine",
    "type": "code_agent",
    "provider": "anthropic",
    "handler_config": {
      "model": "claude-sonnet-4-6",
      "max_turns": 15,
      "timeout_minutes": 20
    },
    "description": "Claude Code CLI for repository-based work",
    "is_default": true
  }'
```

**Request Body** (`EngineCreateRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique engine identifier |
| `name` | string | Yes | Human-readable name |
| `type` | string | Yes | `code_agent`, `managed_agent`, `direct_ops`, or `custom` |
| `provider` | string | No (default: "anthropic") | Provider name |
| `handler_config` | object | No | Engine-specific configuration |
| `description` | string | No | Description |
| `is_default` | boolean | No (default: false) | Whether this is the default engine |

**Response** `201 Created`:

```json
{
  "id": "claude-code-v1",
  "name": "Claude Code Engine",
  "type": "code_agent",
  "provider": "anthropic",
  "handler_config": {
    "model": "claude-sonnet-4-6",
    "max_turns": 15,
    "timeout_minutes": 20
  },
  "description": "Claude Code CLI for repository-based work",
  "is_default": true,
  "is_active": true
}
```

### List Engines

```bash
curl http://localhost:8000/v2/engines \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "id": "claude-code-v1",
    "name": "Claude Code Engine",
    "type": "code_agent",
    "provider": "anthropic",
    "handler_config": { "model": "claude-sonnet-4-6" },
    "description": "Claude Code CLI for repository-based work",
    "is_default": true,
    "is_active": true
  }
]
```

### Get Engine

```bash
curl http://localhost:8000/v2/engines/claude-code-v1 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns `EngineResponse`.

### Update Engine

```bash
curl -X PUT http://localhost:8000/v2/engines/claude-code-v1 \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "claude-code-v1",
    "name": "Claude Code Engine v2",
    "type": "code_agent",
    "provider": "anthropic",
    "handler_config": {
      "model": "claude-opus-4-6",
      "max_turns": 20
    },
    "description": "Updated to use Opus model",
    "is_default": true
  }'
```

**Response** `200 OK`: Returns updated `EngineResponse`.

### Delete Engine

Soft delete.

```bash
curl -X DELETE http://localhost:8000/v2/engines/claude-code-v1 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "claude-code-v1",
  "message": "Engine deleted"
}
```

---

