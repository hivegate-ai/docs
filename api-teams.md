# Teams API


Base path: `/v2/teams`

### List All Teams

```bash
curl http://localhost:8000/v2/teams \
  -H "X-API-Key: agk_abc123def456"
```

**Query Parameters**:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_inactive` | boolean | false | Include soft-deleted teams |

**Response** `200 OK`:

```json
[
  {
    "id": "support-team",
    "name": "Support Team",
    "description": "Multi-agent support team",
    "version": "2.0",
    "mode": "coordinate",
    "agents": [
      { "agent_id": "customer-support", "role": "lead", "order_index": 0 },
      { "agent_id": "technical-support", "role": "specialist", "order_index": 1 }
    ],
    "updated_at": "2025-01-15T10:30:00",
    "created_at": "2025-01-10T08:00:00"
  }
]
```

### Get Team by ID

```bash
curl http://localhost:8000/v2/teams/support-team \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "support-team",
  "name": "Support Team",
  "description": "Multi-agent support team",
  "version": "2.0",
  "mode": "coordinate",
  "agents": [
    { "agent_id": "customer-support", "role": "lead", "order_index": 0 },
    { "agent_id": "technical-support", "role": "specialist", "order_index": 1 }
  ],
  "updated_at": "2025-01-15T10:30:00",
  "created_at": "2025-01-10T08:00:00"
}
```

### Create Team

```bash
curl -X POST http://localhost:8000/v2/teams \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "dev-team",
    "name": "Development Team",
    "description": "Multi-agent team for development tasks",
    "mode": "coordinate",
    "agents": [
      { "agent_id": "code-reviewer", "role": "reviewer", "order_index": 0 },
      { "agent_id": "test-writer", "role": "tester", "order_index": 1 }
    ]
  }'
```

**Request Body** (`CreateTeamRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique team identifier |
| `name` | string | Yes | Human-readable name |
| `description` | string | No | Team description |
| `mode` | string | No (default: "coordinate") | Team mode: `coordinate` or `supervisor` |
| `agents` | TeamAgentAssignment[] | No | List of agents with roles and order |

**Response** `201 Created`:

```json
{
  "id": "dev-team",
  "name": "Development Team",
  "description": "Multi-agent team for development tasks",
  "version": "2.0",
  "message": "Team dev-team created successfully"
}
```

### Delete Team

```bash
# Hard delete (permanent)
curl -X DELETE http://localhost:8000/v2/teams/dev-team \
  -H "X-API-Key: agk_abc123def456"

# Soft delete (archive)
curl -X DELETE "http://localhost:8000/v2/teams/dev-team?soft=true" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "message": "Team dev-team deleted permanently"
}
```

### Run Team

```bash
curl -X POST http://localhost:8000/v2/teams/support-team/runs \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "A customer reports they cannot log in after changing their email address",
    "stream": false,
    "model": "gemini-2.5-pro",
    "user_id": "user-42",
    "session_id": "team-sess-001",
    "stream_verbosity": "events",
    "user_profile": {
      "profile_id": "prof-42",
      "email": "operator@acme.com",
      "full_name": "Operator",
      "role": "Support Lead",
      "tenant_id": "tenant-acme"
    },
    "tenant_profile": {
      "tenant_id": "tenant-acme",
      "name": "Acme Corp",
      "website": "https://acme.com"
    },
    "timezone": "America/New_York",
    "locale": "en-US"
  }'
```

**Request Body** (`TeamRunRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `message` | string | Yes | User message |
| `stream` | boolean | No (default: true) | Enable SSE streaming |
| `stream_verbosity` | string | No (default: "events") | Verbosity: `full`, `events`, `result` |
| `model` | Model enum | No (default: gemini-2.5-pro) | LLM model |
| `user_id` | string | No | User identifier |
| `session_id` | string | No | Session identifier |
| `user_profile` | UserProfile | No | User profile |
| `tenant_profile` | TenantProfile | No | Tenant profile |
| `timezone` | string | No | User timezone |
| `locale` | string | No | User locale |
| `temperature` | float | No | Model temperature |
| `max_tokens` | integer | No | Max output tokens |
| `images` | object[] | No | Multimodal images |

**Response** `200 OK`:

```json
{
  "content": "Based on our investigation, the customer needs to verify their new email...",
  "team_id": "support-team",
  "session_id": "team-sess-001",
  "model": "gemini-2.5-pro",
  "token_usage": {
    "input_tokens": 1250,
    "output_tokens": 340,
    "total_tokens": 1590
  },
  "status": "completed",
  "run_id": "run-team-abc",
  "tools": null
}
```

### Commit Team Run

Resume a paused team run with confirmed/edited tools.

```bash
curl -X POST http://localhost:8000/v2/teams/support-team/runs/commit \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "run_id": "run-team-abc",
    "stream": false,
    "model": "gemini-2.5-pro",
    "user_id": "user-42",
    "session_id": "team-sess-001",
    "updated_tools": [
      {
        "tool_call_id": "tc-002",
        "confirmed": true
      }
    ]
  }'
```

**Response** `200 OK`:

```json
{
  "content": "The email update has been confirmed and the customer's account is now accessible.",
  "team_id": "support-team",
  "session_id": "team-sess-001",
  "model": "gemini-2.5-pro",
  "status": "completed"
}
```

### List Team Sessions

```bash
curl http://localhost:8000/v2/teams/support-team/sessions \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "id": "team-sess-001",
    "team_id": "support-team",
    "created_at": "2025-01-15T10:00:00",
    "updated_at": "2025-01-15T10:30:00",
    "message_count": 12
  }
]
```

### Get Team Session

```bash
curl http://localhost:8000/v2/teams/support-team/sessions/team-sess-001 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "team-sess-001",
  "team_id": "support-team",
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T10:30:00",
  "message_count": 12
}
```

### Delete Team Session

```bash
curl -X DELETE http://localhost:8000/v2/teams/support-team/sessions/team-sess-001 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `204 No Content`

### List Team Memories

```bash
curl http://localhost:8000/v2/teams/support-team/memories \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "id": "mem-001",
    "team_id": "support-team",
    "session_id": null,
    "content": "Customer prefers email communication over phone calls",
    "created_at": "2025-01-15T10:25:00"
  }
]
```

### List Team Members

```bash
curl http://localhost:8000/v2/teams/support-team/members \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "agent_id": "customer-support",
    "role": "lead",
    "order_index": 0,
    "created_at": "2025-01-10T08:00:00"
  },
  {
    "agent_id": "technical-support",
    "role": "specialist",
    "order_index": 1,
    "created_at": "2025-01-10T08:00:00"
  }
]
```

### Add Team Member

```bash
curl -X POST http://localhost:8000/v2/teams/support-team/members \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "escalation-agent",
    "role": "escalation",
    "order_index": 2
  }'
```

**Response** `201 Created`:

```json
{
  "message": "Agent escalation-agent added to team support-team successfully",
  "member": {
    "agent_id": "escalation-agent",
    "role": "escalation",
    "order_index": 2,
    "created_at": "2025-01-15T11:00:00"
  }
}
```

### Update Team Member

```bash
curl -X PUT http://localhost:8000/v2/teams/support-team/members/escalation-agent \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "role": "senior-escalation",
    "order_index": 1
  }'
```

**Response** `200 OK`:

```json
{
  "agent_id": "escalation-agent",
  "role": "senior-escalation",
  "order_index": 1,
  "created_at": "2025-01-15T11:00:00"
}
```

### Remove Team Member

```bash
curl -X DELETE http://localhost:8000/v2/teams/support-team/members/escalation-agent \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "message": "Agent escalation-agent removed from team support-team successfully"
}
```

---

