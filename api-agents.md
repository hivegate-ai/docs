# Agents API


Base path: `/v2/agents`

### List All Agents

```bash
curl http://localhost:8000/v2/agents \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "id": "customer-support",
    "name": "Customer Support Agent",
    "description": "Handles customer inquiries and troubleshooting",
    "version": "2.0",
    "template": "You are a customer support agent for Acme Corp...",
    "tags": ["support", "customer-facing"],
    "config": {
      "enable_memory": true,
      "enable_history": true,
      "num_history_runs": 3,
      "enable_reasoning": false,
      "reasoning_min_steps": 1,
      "reasoning_max_steps": 10
    }
  }
]
```

### Get Agent by ID

```bash
curl http://localhost:8000/v2/agents/customer-support \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "customer-support",
  "name": "Customer Support Agent",
  "description": "Handles customer inquiries and troubleshooting",
  "version": "2.0",
  "template": "You are a customer support agent for Acme Corp...",
  "tags": ["support", "customer-facing"],
  "config": {
    "enable_memory": true,
    "enable_history": true,
    "num_history_runs": 3,
    "enable_reasoning": false,
    "reasoning_min_steps": 1,
    "reasoning_max_steps": 10
  }
}
```

### Get Agents by IDs (Batch)

```bash
curl -X POST http://localhost:8000/v2/agents/batch \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_ids": ["customer-support", "sales-assistant"]
  }'
```

**Response** `200 OK`:

```json
[
  {
    "id": "customer-support",
    "name": "Customer Support Agent",
    "description": "Handles customer inquiries",
    "version": "2.0",
    "template": "You are a customer support agent...",
    "tags": ["support"],
    "config": {
      "enable_memory": true,
      "enable_history": true,
      "num_history_runs": 3,
      "enable_reasoning": false,
      "reasoning_min_steps": 1,
      "reasoning_max_steps": 10
    }
  }
]
```

### Create Agent

```bash
curl -X POST http://localhost:8000/v2/agents \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "code-reviewer",
    "name": "Code Reviewer Agent",
    "template": "You are an expert code reviewer. Review code for bugs, security issues, and best practices.",
    "description": "Reviews pull requests and code changes",
    "tags": ["engineering", "code-review"],
    "config": {
      "enable_memory": true,
      "enable_history": true,
      "num_history_runs": 5,
      "enable_reasoning": true,
      "reasoning_min_steps": 2,
      "reasoning_max_steps": 8
    }
  }'
```

**Request Body** (`CreateAgentRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string (max 255) | Yes | Unique agent identifier (alphanumeric, hyphens, underscores) |
| `name` | string (max 255) | Yes | Human-readable name |
| `template` | string | Yes | Prompt template text |
| `description` | string (max 1000) | No | Agent description |
| `tags` | string[] | No | Categorization tags |
| `config` | AgentConfigRequest | No | Agent configuration (memory, history, reasoning) |

**AgentConfigRequest**:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enable_memory` | boolean | true | Enable MemoryManager for long-term memory |
| `enable_history` | boolean | true | Include chat history in context |
| `num_history_runs` | integer | 3 | Number of history runs to include |
| `enable_reasoning` | boolean | false | Enable step-by-step reasoning |
| `reasoning_min_steps` | integer | 1 | Minimum reasoning steps |
| `reasoning_max_steps` | integer | 10 | Maximum reasoning steps |
| `worker_config` | object | null | Supervisor worker configuration (MCP servers, hooks, permissions) |

**Response** `201 Created`:

```json
{
  "id": "code-reviewer",
  "name": "Code Reviewer Agent",
  "description": "Reviews pull requests and code changes",
  "version": "2.0",
  "message": "Agent Code Reviewer Agent created successfully",
  "tags": ["engineering", "code-review"]
}
```

### Delete Agent

Soft-deletes the agent from the database and removes the prompt from storage.

```bash
curl -X DELETE http://localhost:8000/v2/agents/code-reviewer \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "code-reviewer",
  "message": "Agent code-reviewer deleted successfully"
}
```

### Chat with Agent

```bash
curl -X POST http://localhost:8000/v2/agents/customer-support/chat \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "I need help resetting my password",
    "stream": false,
    "model": "gemini-2.5-pro",
    "user_id": "user-42",
    "session_id": "sess-abc123",
    "timezone": "America/New_York",
    "locale": "en-US",
    "user_profile": {
      "profile_id": "prof-42",
      "email": "jane@acme.com",
      "full_name": "Jane Smith",
      "role": "Engineering Manager",
      "tenant_id": "tenant-acme"
    },
    "tenant_profile": {
      "tenant_id": "tenant-acme",
      "name": "Acme Corp",
      "website": "https://acme.com"
    }
  }'
```

**Request Body** (`ChatRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `message` | string | Yes | User message |
| `stream` | boolean | No (default: true) | Enable SSE streaming |
| `model` | Model enum | No (default: gemini-2.5-pro) | LLM model to use |
| `user_id` | string | Yes | User identifier |
| `session_id` | string | Yes | Session identifier for conversation continuity |
| `timezone` | string | Yes | User timezone (e.g., "America/New_York") |
| `locale` | string | Yes | User locale (e.g., "en-US") |
| `temperature` | float | No | Model temperature |
| `max_tokens` | integer | No | Max output tokens |
| `user_profile` | UserProfile | No | User profile with email, name, role |
| `tenant_profile` | TenantProfile | No | Tenant/organization profile |
| `images` | object[] | No | Base64-encoded images for multimodal input |
| `system_prompt` | string | No | Custom system prompt override |
| `tools` | string[] | No | Tool identifiers for cache key |

**Non-streaming Response** `200 OK`:

```json
{
  "content": "I can help you reset your password. Please go to Settings > Security > Reset Password...",
  "agent_id": "customer-support",
  "session_id": "sess-abc123",
  "model": "gemini-2.5-pro",
  "token_usage": null,
  "status": "completed",
  "run_id": "run-f47ac10b",
  "tools": []
}
```

**Streaming Response**: Returns `text/event-stream` with SSE events:

```
event: message
data: {"content": "I can help ", "status": "running", "run_id": "run-f47ac10b"}

event: message
data: {"content": "you reset your password.", "status": "completed", "run_id": "run-f47ac10b"}
```

**Paused Response** (when a tool requires confirmation):

```json
{
  "content": null,
  "agent_id": "customer-support",
  "session_id": "sess-abc123",
  "model": "gemini-2.5-pro",
  "status": "paused",
  "run_id": "run-f47ac10b",
  "tools": [
    {
      "tool_call_id": "tc-001",
      "tool_name": "send_email",
      "requires_confirmation": true,
      "tool_args": {
        "to": "jane@acme.com",
        "subject": "Password Reset"
      },
      "result": null
    }
  ]
}
```

### Commit (Resume Paused Run)

After inspecting/editing tool args from a paused run, send confirmed tools to resume execution.

```bash
curl -X POST http://localhost:8000/v2/agents/customer-support/chat/commit \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "run_id": "run-f47ac10b",
    "stream": false,
    "model": "gemini-2.5-pro",
    "user_id": "user-42",
    "session_id": "sess-abc123",
    "updated_tools": [
      {
        "tool_call_id": "tc-001",
        "confirmed": true,
        "tool_args": {
          "to": "jane@acme.com",
          "subject": "Password Reset Instructions"
        }
      }
    ]
  }'
```

**Request Body** (`CommitRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `run_id` | string | Yes | Run ID from the paused response |
| `stream` | boolean | No (default: true) | Enable SSE streaming |
| `model` | Model enum | No | Model to use |
| `user_id` | string | Yes | User identifier |
| `session_id` | string | Yes | Session identifier |
| `updated_tools` | object[] | Yes | Tools with confirmed/edited args |

Set `confirmed: false` on all tools to cancel execution:

```json
{
  "run_id": "run-f47ac10b",
  "user_id": "user-42",
  "session_id": "sess-abc123",
  "updated_tools": [
    { "tool_call_id": "tc-001", "confirmed": false }
  ]
}
```

**Response** `200 OK`:

```json
{
  "content": "Tool execution cancelled by user.",
  "agent_id": "customer-support",
  "session_id": "sess-abc123",
  "model": "gemini-2.5-pro",
  "status": "cancelled"
}
```

### Direct Toolkit Execution

Execute a toolkit method directly without going through the chat flow.

```bash
curl "http://localhost:8000/v2/agents/calendar-agent/toolkit/run?toolkit_name=CalendarToolkit&method_name=list_events&user_id=user-42&session_id=sess-1&organizer_email=jane@acme.com&max_results=5" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "status": "success",
  "message": "Toolkit method executed successfully",
  "result": {
    "events": [
      {
        "id": "evt-001",
        "summary": "Team Standup",
        "start": "2025-01-15T09:00:00Z"
      }
    ]
  }
}
```

---

