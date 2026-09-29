# HiveGate API Documentation

## 1. Overview

HiveGate is a FastAPI-based platform for managing, orchestrating, and executing LLM-powered agents. It provides a unified REST API for agent CRUD, multi-agent teams, knowledge management, token storage, prompt templates, skills, and a supervisor/worker execution platform.

### Architecture

```
                                    +-------------------+
                                    |   Client / UI     |
                                    +--------+----------+
                                             |
                                   X-API-Key | (V2 routes)
                                             |
                              +--------------v--------------+
                              |      hivegate         |
                              |    FastAPI  (port 8000)     |
                              |                             |
                              |  /v2/agents   /v2/teams     |
                              |  /v2/prompts  /v2/knowledge |
                              |  /v2/tokens   /v2/skills    |
                              |  /v2/engines  /v2/targets   |
                              |  /v2/approvals              |
                              |  /admin       /health       |
                              +-+----------+----------+-----+
                                |          |          |
                     +----------v--+  +----v----+  +--v--------+
                     | PostgreSQL  |  | Qdrant  |  |  Prompts  |
                     | (metadata,  |  | (vector |  |  Storage  |
                     |  sessions,  |  |  KB)    |  | (postgres)|
                     |  tokens)    |  +---------+  +-----------+
                     +-------------+
```

### Key Concepts

- **Agent**: An LLM-powered assistant defined by a prompt template, model, and configuration. Stored in PostgreSQL with templates in the prompts storage backend.
- **Team**: A group of agents that collaborate. Supports `coordinate` mode (round-robin collaboration) and `supervisor` mode (leader classifies, workers execute).
- **Prompt**: A reusable prompt template stored in the prompts storage backend and referenced by agents.
- **Knowledge**: Document-based knowledge bases with dual-level isolation (tenant and collection) backed by Qdrant vector search.
- **Token**: Encrypted OAuth2/API key credentials stored per user per integration (Google, Slack, OpenAI, etc.).
- **Skill**: A reusable instruction set with references and scripts that agents can leverage.
- **Engine**: An execution engine registry entry (e.g., `code_agent` via Claude Code, `managed_agent` via API, `direct_ops`).
- **Target**: An execution target (local machine, SSH host, remote service, managed agent API).
- **Approval**: A human-in-the-loop approval request for tool calls that require confirmation.
- **Pack**: A YAML-defined bundle of agents, prompts, and team configuration that can be applied as a unit.

---

## 2. Quick Start

### Prerequisites

- Python 3.11+
- Docker (for PostgreSQL and Qdrant)
- `uv` package manager

### Setup

```bash
# Clone and setup
cd agent-api
./scripts/dev_setup.sh && source .venv/bin/activate

# Start infrastructure
docker compose up -d

# Start API server (port 8000)
./scripts/start_server.sh
```

### First API Call

```bash
# Check health (no auth required)
curl http://localhost:8000/health

# With auth disabled (development mode)
export AUTH_DISABLED=true

# List agents
curl http://localhost:8000/v2/agents

# Create an agent
curl -X POST http://localhost:8000/v2/agents \
  -H "Content-Type: application/json" \
  -d '{
    "id": "my-first-agent",
    "name": "My First Agent",
    "template": "You are a helpful assistant that answers questions concisely.",
    "description": "A simple Q&A agent",
    "tags": ["qa", "general"]
  }'

# Chat with the agent
curl -X POST http://localhost:8000/v2/agents/my-first-agent/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What is the capital of France?",
    "stream": false,
    "model": "gemini-2.5-pro",
    "user_id": "user-1",
    "session_id": "session-1",
    "timezone": "UTC",
    "locale": "en-US"
  }'
```

---

## 3. Authentication

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

### Health Routes (`/health`, `/status`, `/version`, `/`)

No authentication required.

### Development Mode

Set `AUTH_DISABLED=true` to bypass all authentication:

```bash
export AUTH_DISABLED=true
./scripts/start_server.sh
```

---

## 4. Agents API

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

## 5. Teams API

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

## 6. Prompts API

Base path: `/v2/prompts`

### Create Prompt

```bash
curl -X POST http://localhost:8000/v2/prompts \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "code-review-prompt",
    "name": "Code Review Prompt",
    "template": "You are a senior code reviewer. Review the following code for:\n1. Bugs and logic errors\n2. Security vulnerabilities\n3. Performance issues\n4. Code style violations",
    "description": "Standard code review prompt template",
    "tags": ["engineering", "code-review"],
    "tools": []
  }'
```

**Request Body** (`PromptCreate`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique prompt identifier |
| `name` | string | Yes | Human-readable name |
| `template` | string | Yes | Prompt template text |
| `description` | string | No | Description |
| `tags` | string[] | No | Categorization tags |
| `tools` | object[] | No | Tool configurations |

**Response** `201 Created`:

```json
{
  "id": "code-review-prompt",
  "name": "Code Review Prompt",
  "template": "You are a senior code reviewer...",
  "description": "Standard code review prompt template",
  "tags": ["engineering", "code-review"],
  "tools": [],
  "version": 1,
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T10:00:00",
  "is_active": true
}
```

### List Prompts

```bash
curl http://localhost:8000/v2/prompts \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "prompts": [
    {
      "id": "code-review-prompt",
      "name": "Code Review Prompt",
      "template": "You are a senior code reviewer...",
      "description": "Standard code review prompt template",
      "tags": ["engineering", "code-review"],
      "tools": [],
      "version": 1,
      "created_at": "2025-01-15T10:00:00",
      "updated_at": "2025-01-15T10:00:00",
      "is_active": true
    }
  ],
  "total": 1
}
```

### Get Prompt

```bash
curl http://localhost:8000/v2/prompts/code-review-prompt \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "id": "code-review-prompt",
  "name": "Code Review Prompt",
  "template": "You are a senior code reviewer...",
  "description": "Standard code review prompt template",
  "tags": ["engineering", "code-review"],
  "tools": [],
  "version": 1,
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T10:00:00",
  "is_active": true
}
```

### Update Prompt

```bash
curl -X PUT http://localhost:8000/v2/prompts/code-review-prompt \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "template": "You are an expert code reviewer. Focus on security, performance, and maintainability.",
    "tags": ["engineering", "code-review", "security"]
  }'
```

**Request Body** (`PromptUpdate`): All fields optional.

**Response** `200 OK`:

```json
{
  "id": "code-review-prompt",
  "name": "Code Review Prompt",
  "template": "You are an expert code reviewer. Focus on security, performance, and maintainability.",
  "description": "Standard code review prompt template",
  "tags": ["engineering", "code-review", "security"],
  "tools": [],
  "version": 2,
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T11:00:00",
  "is_active": true
}
```

### Delete Prompt

Soft delete (sets `is_active=false`).

```bash
curl -X DELETE http://localhost:8000/v2/prompts/code-review-prompt \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "message": "Prompt deleted successfully",
  "id": "code-review-prompt"
}
```

---

## 7. Knowledge API

Base path: `/v2/knowledge`

Knowledge entries support dual-level isolation:
- **Tenant-level**: Shared across all collections for an organization (`/v2/knowledge/{tenant_id}`)
- **Collection-level**: Scoped to a specific project or collection (`/v2/knowledge/{tenant_id}/{collection_id}`)

### Create Tenant Knowledge Entry

```bash
curl -X POST http://localhost:8000/v2/knowledge/tenant-acme \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "file_id": "550e8400-e29b-41d4-a716-446655440001",
    "original_filename": "company-handbook.pdf",
    "file_type": "company",
    "content_type": "application/pdf",
    "gcs_path": "gs://acme-docs/company-handbook.pdf",
    "status": "active",
    "metadata": { "department": "HR", "year": 2025 }
  }'
```

**Request Body** (`KnowledgeEntryCreate`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file_id` | string (UUID) | Yes | UUID of the source file |
| `original_filename` | string | Yes | Original filename |
| `file_type` | string | Yes | `company` or `project` |
| `content_type` | string | No | MIME type |
| `gcs_path` | string | No | Google Cloud Storage path |
| `status` | string | No (default: "active") | File status |
| `metadata` | object | No | Additional metadata |

**Response** `201 Created`:

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "tenant_id": "tenant-acme",
  "collection_id": null,
  "file_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "active",
  "knowledge_status": "indexing",
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T10:00:00",
  "message": "File added to tenant knowledge base. Knowledge indexing is processing asynchronously."
}
```

### List Tenant Knowledge Entries

```bash
curl "http://localhost:8000/v2/knowledge/tenant-acme?limit=50&offset=0" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "files": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "tenant_id": "tenant-acme",
      "collection_id": null,
      "file_id": "550e8400-e29b-41d4-a716-446655440001",
      "original_filename": "company-handbook.pdf",
      "file_type": "company",
      "content_type": "application/pdf",
      "gcs_path": "gs://acme-docs/company-handbook.pdf",
      "status": "active",
      "knowledge_status": "indexed",
      "metadata": { "department": "HR", "year": 2025 },
      "created_at": "2025-01-15T10:00:00",
      "updated_at": "2025-01-15T10:05:00"
    }
  ],
  "pagination": {
    "total": 1,
    "limit": 50,
    "offset": 0,
    "has_more": false
  }
}
```

### Get Tenant Knowledge Entry

```bash
curl http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns `KnowledgeEntryResponse` (same schema as list items).

### Update Tenant Knowledge Entry

```bash
curl -X PATCH http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "archived",
    "metadata": { "department": "HR", "year": 2025, "archived_reason": "superseded" }
  }'
```

**Request Body** (`KnowledgeEntryUpdate`): All fields optional (`status`, `knowledge_status`, `metadata`).

**Response** `200 OK`: Returns updated `KnowledgeEntryResponse`.

### Delete Tenant Knowledge Entry

```bash
# Soft delete
curl -X DELETE http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: agk_abc123def456"

# Hard delete
curl -X DELETE "http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001?hard_delete=true" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `204 No Content`

### Collection-Level Knowledge Endpoints

All tenant-level endpoints have collection-level equivalents at `/v2/knowledge/{tenant_id}/{collection_id}`:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/{tenant_id}/{collection_id}` | POST | Create collection knowledge entry |
| `/{tenant_id}/{collection_id}` | GET | List collection knowledge entries |
| `/{tenant_id}/{collection_id}/files/{file_id}` | GET | Get collection knowledge entry |
| `/{tenant_id}/{collection_id}/files/{file_id}` | PATCH | Update collection knowledge entry |
| `/{tenant_id}/{collection_id}/files/{file_id}` | DELETE | Delete collection knowledge entry |

Request and response models are identical to the tenant-level endpoints.

---

## 8. Tokens API

Base path: `/v2/users/{user_id}/tokens`

Manages encrypted OAuth2, API key, and JWT credentials per user per integration. All token data is encrypted with Fernet using the `SECRET_TOKEN_ENC_KEY` environment variable.

### Store Token

```bash
curl -X POST http://localhost:8000/v2/users/user-42/tokens \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "integration_key": "google",
    "provider": "google",
    "token_type": "oauth2",
    "token_data": {
      "access_token": "ya29.a0AfH6SM...",
      "refresh_token": "1//0dx2Xj...",
      "token_type": "Bearer",
      "expires_in": 3600,
      "scope": "https://www.googleapis.com/auth/calendar",
      "client_id": "123456.apps.googleusercontent.com",
      "client_secret": "GOCSPX-..."
    },
    "scopes": ["calendar", "email"],
    "expires_at": "2025-01-16T10:00:00Z"
  }'
```

**Request Body** (`StoreTokenRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `integration_key` | string | Yes | Integration identifier (e.g., "google", "slack", "openai") |
| `provider` | string | Yes | Provider name (whitelisted: google, slack, openai, anthropic, github, microsoft, zoom, dropbox, notion) |
| `token_type` | string | Yes | `oauth2`, `api_key`, or `jwt` |
| `token_data` | TokenData | Yes | Token credentials (access_token, refresh_token, api_key, etc.) |
| `scopes` | string[] | No | Permission scopes |
| `expires_at` | datetime | No | Expiration time |

**Response** `201 Created`:

```json
{
  "integration_key": "google",
  "provider": "google",
  "token_type": "oauth2",
  "message": "Token stored successfully for google",
  "created_at": "2025-01-15T10:00:00"
}
```

### Get Token (with Auto-Refresh)

Retrieves the token and automatically refreshes OAuth2 tokens if expired.

```bash
curl http://localhost:8000/v2/users/user-42/tokens/google \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "integration_key": "google",
  "provider": "google",
  "token_type": "oauth2",
  "token_data": {
    "access_token": "ya29.a0AfH6SM...",
    "refresh_token": "1//0dx2Xj...",
    "token_type": "Bearer"
  },
  "scopes": ["calendar", "email"],
  "expires_at": "2025-01-16T10:00:00",
  "is_expired": false,
  "refreshed": false
}
```

### List User Tokens

Returns metadata only (no token data).

```bash
curl http://localhost:8000/v2/users/user-42/tokens \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "integration_key": "google",
    "provider": "google",
    "token_type": "oauth2",
    "scopes": ["calendar", "email"],
    "expires_at": "2025-01-16T10:00:00",
    "created_at": "2025-01-15T10:00:00",
    "updated_at": "2025-01-15T10:00:00",
    "is_expired": false
  },
  {
    "integration_key": "openai",
    "provider": "openai",
    "token_type": "api_key",
    "scopes": null,
    "expires_at": null,
    "created_at": "2025-01-14T08:00:00",
    "updated_at": "2025-01-14T08:00:00",
    "is_expired": false
  }
]
```

### Refresh Token

Manually trigger an OAuth2 token refresh.

```bash
curl -X POST http://localhost:8000/v2/users/user-42/tokens/google/refresh \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "integration_key": "google",
  "success": true,
  "message": "Token refreshed successfully",
  "refreshed_at": "2025-01-15T11:00:00"
}
```

### Delete Token

```bash
curl -X DELETE http://localhost:8000/v2/users/user-42/tokens/google \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "integration_key": "google",
  "message": "Token for google deleted successfully"
}
```

---

## 9. Skills API

Base path: `/v2/skills`

### Create Skill

```bash
curl -X POST http://localhost:8000/v2/skills \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "terraform-deploy",
    "name": "Terraform Deployment",
    "instructions": "Follow these steps to deploy infrastructure:\n1. Run terraform init\n2. Run terraform plan\n3. Review the plan output\n4. Apply only if the plan is safe",
    "description": "Standard Terraform deployment workflow",
    "category": "infrastructure",
    "references": [
      { "name": "Terraform Best Practices", "content": "https://docs.terraform.io/best-practices" }
    ],
    "scripts": [
      { "name": "validate", "content": "terraform validate && terraform fmt -check" }
    ],
    "allowed_tools": ["bash", "read_file", "write_file"],
    "tags": ["terraform", "infrastructure", "deployment"]
  }'
```

**Request Body** (`SkillCreate`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string (max 255) | Yes | Unique skill identifier |
| `name` | string (max 255) | Yes | Human-readable name |
| `instructions` | string | Yes | Skill instructions |
| `description` | string (max 1000) | No | Description |
| `category` | string (max 255) | No | Category for filtering |
| `references` | object[] | No | Reference materials `[{name, content}]` |
| `scripts` | object[] | No | Executable scripts `[{name, content}]` |
| `allowed_tools` | string[] | No | Tools this skill can use |
| `tags` | string[] | No | Tags |

**Response** `201 Created`:

```json
{
  "id": "terraform-deploy",
  "name": "Terraform Deployment",
  "instructions": "Follow these steps to deploy infrastructure...",
  "description": "Standard Terraform deployment workflow",
  "category": "infrastructure",
  "references": [{ "name": "Terraform Best Practices", "content": "https://docs.terraform.io/best-practices" }],
  "scripts": [{ "name": "validate", "content": "terraform validate && terraform fmt -check" }],
  "allowed_tools": ["bash", "read_file", "write_file"],
  "tags": ["terraform", "infrastructure", "deployment"],
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T10:00:00",
  "is_active": true
}
```

### List Skills

```bash
# All active skills
curl http://localhost:8000/v2/skills \
  -H "X-API-Key: agk_abc123def456"

# Filter by category
curl "http://localhost:8000/v2/skills?category=infrastructure" \
  -H "X-API-Key: agk_abc123def456"

# Include inactive
curl "http://localhost:8000/v2/skills?include_inactive=true" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "id": "terraform-deploy",
    "name": "Terraform Deployment",
    "description": "Standard Terraform deployment workflow",
    "category": "infrastructure",
    "tags": ["terraform", "infrastructure", "deployment"],
    "is_active": true
  }
]
```

### Get Skill

```bash
curl http://localhost:8000/v2/skills/terraform-deploy \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns full `SkillResponse` (includes instructions, references, scripts).

### Update Skill

```bash
curl -X PUT http://localhost:8000/v2/skills/terraform-deploy \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "instructions": "Updated deployment workflow with additional safety checks...",
    "tags": ["terraform", "infrastructure", "deployment", "safety"]
  }'
```

**Response** `200 OK`: Returns updated `SkillResponse`.

### Delete Skill

Soft delete (sets `is_active=false`).

```bash
curl -X DELETE http://localhost:8000/v2/skills/terraform-deploy \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `204 No Content`

---

## 10. Supervisor Platform

The supervisor platform enables a multi-agent architecture where a supervisor agent classifies incoming requests and delegates work to specialized worker agents.

### Architecture

```
                   User Message
                        |
                        v
               +--------+--------+
               | Supervisor Agent |  (role: leader)
               |  - Classifies   |
               |  - Routes       |
               |  - Plans        |
               +-+------+------+-+
                 |      |      |
          +------+  +---+---+ +------+
          |         |       |        |
   +------v---+ +--v----+ +v-------+ +----------+
   | Coding   | |Infra  | | Ops    | | Verifier |
   | Worker   | |Worker | | Worker | | Worker   |
   +----+-----+ +---+---+ +---+----+ +----+-----+
        |            |         |           |
   +----v-----+ +---v----+ +--v-----+ +---v-----+
   |ClaudeCode| |Claude  | |Direct  | |Claude   |
   | Toolkit  | |Code TK | |Ops     | |Code TK  |
   +----------+ +--------+ +--------+ +---------+
```

### Classification

The supervisor classifies each request into one of these categories:

| Classification | Description |
|---------------|-------------|
| `no_action` | No action needed |
| `answer_only` | Can answer directly without worker |
| `read_only_analysis` | Read-only code/data analysis |
| `code_fix` | Bug fix or small code change |
| `feature_small` | Small feature (1-2 files) |
| `feature_medium` | Medium feature (3-5 files) |
| `feature_large` | Large feature (6+ files) |
| `refactor_scoped` | Scoped refactoring |
| `test_generation` | Generate tests |
| `documentation_update` | Update documentation |
| `infrastructure_change` | Infrastructure/IaC changes |
| `noc_operation` | Runtime operations (kubectl, monitoring) |
| `high_risk_escalation` | Requires human review |

### Worker Types

| Worker | Engine | Description |
|--------|--------|-------------|
| `coding` | claude_code | Repository-based code changes |
| `planning` | claude_code | Read-only analysis and planning |
| `infrastructure` | claude_code | Terraform, Helm, CDK changes |
| `operations` | direct_ops | Runtime ops (kubectl, AWS, GCP) |
| `documentation` | claude_code | Documentation updates |
| `verifier` | claude_code | Test execution and verification |
| `data_platform` | claude_code | Data pipeline/DAG work |

### Execution Engines

| Engine Type | Description |
|-------------|-------------|
| `code_agent` | Claude Code CLI with MCP plugins |
| `managed_agent` | API-based agents (Anthropic, OpenAI, Google) |
| `direct_ops` | Direct tool execution (kubectl, AWS CLI) |
| `custom` | Custom execution handler |

### Execution Targets

| Target Type | Description |
|-------------|-------------|
| `local` | Local machine execution |
| `ssh` | Remote execution via SSH |
| `remote_service` | Remote API-based execution |
| `managed_agents` | Managed agent API endpoints |

### Execution Limits

Each job has configurable limits:

```json
{
  "network_access": false,
  "allow_dependency_install": false,
  "allow_git_push": false,
  "allow_merge": false,
  "allow_delete_files": false,
  "allow_migrations": false,
  "allow_apply_or_deploy": false,
  "allow_production_change": false,
  "max_runtime_minutes": 15,
  "max_attempts": 3,
  "max_memory_mb": 4096,
  "max_cpus": 2.0
}
```

### MCP Plugin Support

Workers can use MCP (Model Context Protocol) servers configured per worker:

```yaml
mcp_servers:
  - name: filesystem
    type: stdio
    command: npx
    args: ["-y", "@anthropic-ai/mcp-filesystem"]
    description: File read/write
  - name: github
    type: http
    url: https://api.githubcopilot.com/mcp/
    description: GitHub PRs and issues
```

### Permission Rules

Workers have granular permission rules:

```yaml
permissions:
  allow:
    - "Read(/src/**)"
    - "Bash(git diff *)"
    - "Bash(npm run *)"
  deny:
    - "Bash(rm -rf *)"
    - "Bash(git push --force *)"
    - "Edit(.env)"
  ask:
    - "Bash(git push *)"
```

### Streaming

Supervisor runs support SSE streaming with three verbosity levels:

| Verbosity | Events Included |
|-----------|----------------|
| `full` | All events (tool calls, thinking, results, status) |
| `events` | Tool calls, messages, status changes, approval requests |
| `result` | Messages, errors, and final status only |

### OOM Recovery

The platform handles out-of-memory failures with automatic retry:

1. **Retry**: If under `max_retries`, retry with `memory_multiplier * current_memory`
2. **Re-plan**: If at retry limit and `enable_supervisor_replan=true`, supervisor re-classifies
3. **Circuit breaker**: After `circuit_breaker_threshold` failures, mark `failed_circuit_open` and escalate

Default retry policy:

```json
{
  "max_retries": 2,
  "memory_multiplier": 2.0,
  "enable_supervisor_replan": true,
  "circuit_breaker_threshold": 3
}
```

---

## 11. Engines API

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

## 12. Targets API

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

## 13. Approvals API

Base path: `/v2/approvals`

The approvals API provides polling-based human-in-the-loop approval for tool calls that require confirmation. When a worker agent attempts a tool call marked as requiring confirmation, the run pauses and an approval request is registered.

### List Pending Approvals

```bash
curl http://localhost:8000/v2/approvals \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "job_id": "run-abc:tc-001",
    "worker_name": "Coding Worker",
    "tool_name": "run_claude_code",
    "tool_args": {
      "prompt": "Fix the login validation bug in auth.py",
      "repo_path": "/src/app",
      "model": "claude-sonnet-4-6"
    },
    "risk_level": "medium",
    "reason": "Tool requires human confirmation before execution",
    "timestamp": "2025-01-15T10:30:00"
  }
]
```

### Get Pending Approval

```bash
curl http://localhost:8000/v2/approvals/run-abc:tc-001 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns `ApprovalResponse`.

### Submit Decision (Approve or Deny)

```bash
curl -X POST http://localhost:8000/v2/approvals/run-abc:tc-001/decide \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "approved": true,
    "reason": "Reviewed and safe to proceed",
    "modified_args": {
      "prompt": "Fix the login validation bug in auth.py. Do not modify tests.",
      "repo_path": "/src/app"
    }
  }'
```

**Request Body** (`ApprovalDecisionRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `approved` | boolean | Yes | Whether to approve or deny |
| `reason` | string | No | Reason for the decision |
| `modified_args` | object | No | Modified tool arguments (override originals) |

**Response** `200 OK`:

```json
{
  "job_id": "run-abc:tc-001",
  "status": "approved",
  "message": "Tool call approved"
}
```

### Quick Approve

```bash
curl -X POST http://localhost:8000/v2/approvals/run-abc:tc-001/approve \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "job_id": "run-abc:tc-001",
  "status": "approved",
  "message": "Tool call approved"
}
```

### Quick Deny

```bash
curl -X POST http://localhost:8000/v2/approvals/run-abc:tc-001/deny \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "job_id": "run-abc:tc-001",
  "status": "denied",
  "message": "Tool call denied"
}
```

### List Notification Plugins

```bash
curl http://localhost:8000/v2/approvals/plugins/list \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
["webhook", "telegram", "slack", "discord", "whatsapp"]
```

---

## 14. Admin API

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

## 15. Agent Packs

Packs are YAML-defined bundles that provision a complete supervisor/worker team with a single operation.

### Pack Format

A pack is a directory containing:

```
packs/my-pack/
  pack.yaml              # Manifest file
  prompts/
    supervisor.txt       # Supervisor prompt
    coding_worker.txt    # Worker prompt
    ...
  extensions/
    data_platform.txt    # Domain extension prompt
```

### pack.yaml Schema

```yaml
name: My Supervisor Pack
version: "1.0"
description: >
  Multi-agent supervisor with coding and operations workers.

agents:
  - id: agent-supervisor
    name: Supervisor Agent
    prompt_file: prompts/supervisor.txt
    role: leader          # "leader" or "worker"
    order_index: 0
    engine: claude_code   # Execution engine preference

  - id: worker-coding
    name: Coding Worker
    prompt_file: prompts/coding_worker.txt
    role: worker
    order_index: 1
    engine: claude_code
    target: linux-pool
    worker_config:
      execution_engine_preference: claude_code
      worker_pool: linux_worker_pool
      mcp_servers:
        - name: filesystem
          type: stdio
          command: npx
          args: ["-y", "@anthropic-ai/mcp-filesystem"]
          description: File read/write
        - name: git
          type: stdio
          command: npx
          args: ["-y", "@anthropic-ai/mcp-git"]
      permissions:
        allow:
          - "Read(/src/**)"
          - "Bash(git diff *)"
        deny:
          - "Bash(rm -rf *)"
        ask:
          - "Bash(git push *)"
      allowed_tools: [read_file, write_file, bash, git]
      allowed_commands: [git, npm, python, pytest]

extensions:
  - id: data-platform
    name: Data Platform Extension
    prompt_file: extensions/data_platform.txt
    domain_tags:
      - "domain:data_platform"

team:
  id: supervisor-team
  name: My Supervisor Team
  mode: supervisor
```

### Applying a Pack

Packs are applied programmatically via the `PackLoader`:

```python
from supervisor.pack.loader import PackLoader

loader = PackLoader()
manifest = loader.load("packs/my-pack")
results = await loader.apply("packs/my-pack", manifest, api_client)
# results = {"prompts": [...], "agents": [...], "team": "supervisor-team"}
```

The apply operation:
1. Creates prompts for each agent and extension
2. Creates agent records with worker configurations
3. Creates the team with agent assignments

### Example: Default Supervisor Pack

The `hivegate-ai/tests` repository ships an example `packs/default_supervisor/` pack that can be loaded via the CLI. It includes:

| Agent | Role | Engine | Description |
|-------|------|--------|-------------|
| `agent-supervisor` | leader | claude_code | Classifies and routes requests |
| `worker-coding` | worker | claude_code | Repository-based coding |
| `worker-planning` | worker | claude_code | Read-only analysis and planning |
| `worker-infrastructure` | worker | claude_code | Terraform/Helm/CDK changes |
| `worker-operations` | worker | direct_ops | Runtime operations (kubectl, AWS) |
| `worker-documentation` | worker | claude_code | Documentation updates |
| `worker-verifier` | worker | claude_code | Test and lint verification |
| `worker-data-platform` | worker | claude_code | Data pipeline/DAG work |

---

## 16. Toolkits

Toolkits are Agno `Toolkit` adapters that provide agents with access to external services.

### Architecture

```
Toolkits (Agno adapters)  -->  workspace_suite (vendor-agnostic)  -->  Google/Microsoft APIs
```

All workspace toolkits inherit from `BaseToolkit(Toolkit, ABC)` in `toolkits/base.py`.

### CalendarToolkit

Multi-provider calendar management (Google Calendar, Microsoft Outlook).

**Capabilities**: List events, create events, update events, cancel meetings, check availability, find free slots.

```python
from toolkits import CalendarToolkit

calendar = CalendarToolkit(
    user_id="user-42",
    organizer_email="jane@acme.com",
    service_name="google_calendar",
    fetch_token_func=fetch_access_token,
)
```

### EmailToolkit

Multi-provider email management (Gmail, Outlook Mail).

**Capabilities**: Read emails, send emails, reply to threads, search inbox, manage labels/folders.

```python
from toolkits import EmailToolkit

email = EmailToolkit(
    user_id="user-42",
    service_name="gmail",
    fetch_token_func=fetch_access_token,
)
```

### ContactsToolkit

Multi-provider contact management (Google Contacts, Microsoft People).

**Capabilities**: Search contacts, get contact details, create contacts, update contacts.

```python
from toolkits import ContactsToolkit

contacts = ContactsToolkit(
    user_id="user-42",
    service_name="google_contacts",
    fetch_token_func=fetch_access_token,
)
```

### DriveToolkit

Multi-provider file management (Google Drive, OneDrive).

**Capabilities**: List files, search files, read file contents, upload files.

```python
from toolkits import DriveToolkit

drive = DriveToolkit(
    user_id="user-42",
    service_name="google_drive",
    fetch_token_func=fetch_access_token,
)
```

### ClaudeCodeToolkit

Dispatches work to Claude Code CLI with MCP plugins and permission hooks. Used by supervisor worker agents.

**Capabilities**: Execute coding tasks in a repository with configurable permissions, MCP servers, and hooks.

**Execution modes**: `local`, `ssh`, `remote_service`.

**Confirmation required**: `run_claude_code` requires human confirmation before execution.

```python
from toolkits.claude_code import ClaudeCodeToolkit

claude_code = ClaudeCodeToolkit(
    user_id="user-42",
    worker_config=worker_config,
    execution_target_type="local",
)
```

### ManagedAgentsToolkit

Dispatches work to managed agent APIs (Anthropic, OpenAI, Google, or custom webhook).

**Providers**: `anthropic`, `openai`, `google`, `webhook`.

**Confirmation required**: `run_managed_agent` requires human confirmation before execution.

```python
from toolkits.managed_agents import ManagedAgentsToolkit

managed = ManagedAgentsToolkit(
    user_id="user-42",
    worker_config=worker_config,
    provider_name="anthropic",
)
```

### HITL Confirmation Workflow

Toolkits register tools with `requires_confirmation_tools`. When an agent calls a confirmation-required tool:

1. The run pauses and returns `status: "paused"` with a `tools` array
2. The client inspects/edits tool arguments
3. The client sends confirmed tools to the `/chat/commit` or `/runs/commit` endpoint
4. The agent resumes execution with the updated tools

---

## 17. Remote Agent Service

The ClaudeCodeToolkit supports executing work on remote targets:

### Local Execution

Runs Claude Code CLI directly on the host machine. Default mode.

### SSH Execution

Runs Claude Code CLI on a remote host via SSH:

```json
{
  "type": "ssh",
  "host": "worker-1.internal.acme.com",
  "port": 22,
  "ssh_key_path": "/secrets/worker-key.pem",
  "repo_path": "/opt/repos/my-project"
}
```

### Remote Service Execution

Sends the task to a remote API endpoint:

```json
{
  "type": "remote_service",
  "api_url": "https://agent-service.internal.acme.com/execute"
}
```

### Docker / Kubernetes Execution

Jobs are tracked in the `execution_jobs` table with container metadata:

```json
{
  "container_id": "abc123",
  "target_host": "k8s-node-1",
  "memory_limit_mb": 4096,
  "timeout_at": "2025-01-15T10:15:00"
}
```

OOM detection: Exit code 137 (Docker SIGKILL) or K8s `OOMKilled` status triggers automatic retry with increased memory.

---

## 18. Configuration

### Required Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_HOST` | PostgreSQL host | `127.0.0.1` |
| `DB_PORT` | PostgreSQL port | `5432` |
| `DB_USER` | PostgreSQL user | `postgres` |
| `DB_PASS` | PostgreSQL password | (empty) |
| `DB_DATABASE` | PostgreSQL database name | `postgres` |
| `DB_DRIVER` | Database driver | `postgresql+psycopg` |
| `SECRET_TOKEN_ENC_KEY` | Fernet encryption key for token storage | (required in production) |
| `ADMIN_SECRET` | Admin endpoint authentication secret | (required in production) |

### Optional Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `AUTH_DISABLED` | Bypass all authentication | `false` |
| `TESTING` | Enable test mode (SQLite, debug) | `false` |
| `GOOGLE_API_KEY` | Google Gemini API key | (empty) |
| `OPENAI_API_KEY` | OpenAI API key | (empty) |
| `ANTHROPIC_API_KEY` | Anthropic Claude API key | (empty) |
| `QDRANT_URL` | Qdrant vector database URL | (empty, uses in-memory) |
| `QDRANT_API_KEY` | Qdrant authentication key | (empty) |
| `QDRANT_PORT` | Qdrant port | `6333` |
| `SERVICE_PROMPTS` | External prompts service URL | (empty) |
| `SKIP_DB_TABLE_CHECK` | Skip database table creation on startup | `false` |
| `ENABLE_GUARDRAILS` | Enable prompt injection + PII detection | `false` |
| `OTEL_TRACING_BACKEND` | OpenTelemetry tracing backend | `console` |
| `OTEL_LOGGING_BACKEND` | OpenTelemetry logging backend | `console` |
| `BRIGHT_DATA_API_KEY` | Bright Data API key | (empty) |
| `BRIGHT_DATA_WEB_UNLOCKER_ZONE` | Bright Data Web Unlocker zone | `web_unlocker1` |
| `BRIGHT_DATA_SERP_ZONE` | Bright Data SERP zone | `serp_api1` |

---

## 19. Database Schema

### prompts

Schema: `prompts`

| Column | Type | Description |
|--------|------|-------------|
| `id` | VARCHAR(255) PK | Prompt identifier |
| `name` | VARCHAR(255) | Human-readable name |
| `template` | TEXT | Prompt template text |
| `description` | TEXT | Description |
| `tags` | TEXT | JSON string of tags |
| `tools` | TEXT | JSON string of tool configurations |
| `version` | INTEGER | Version number |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active/deleted flag |

### agent_info

| Column | Type | Description |
|--------|------|-------------|
| `id` | VARCHAR PK | Agent identifier |
| `name` | VARCHAR | Agent name |
| `description` | TEXT | Description |
| `version` | VARCHAR | API version (default "2.0") |
| `prompt_service_id` | VARCHAR | Reference to prompt in storage backend |
| `tags` | VARCHAR | JSON string of tags |
| `config` | TEXT | JSON string of agent config (memory, history, reasoning, worker_config) |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active/deleted flag |

### team_info

| Column | Type | Description |
|--------|------|-------------|
| `id` | VARCHAR PK | Team identifier |
| `name` | VARCHAR | Team name |
| `description` | TEXT | Description |
| `version` | VARCHAR | API version (default "2.0") |
| `mode` | VARCHAR | Team mode: `coordinate` or `supervisor` |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active/deleted flag |

### team_agent

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PK | Auto-increment ID |
| `team_id` | VARCHAR FK | Team identifier |
| `agent_id` | VARCHAR FK | Agent identifier |
| `role` | VARCHAR | Agent's role in team |
| `order_index` | INTEGER | Execution order |
| `created_at` | DATETIME | Creation timestamp |
| `is_active` | BOOLEAN | Active flag |

### token_usage

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PK | Auto-increment ID |
| `agent_id` | VARCHAR | Agent identifier |
| `session_id` | VARCHAR | Session identifier |
| `user_id` | VARCHAR | User identifier |
| `model` | VARCHAR | Model used |
| `prompt_tokens` | INTEGER | Input tokens |
| `completion_tokens` | INTEGER | Output tokens |
| `total_tokens` | INTEGER | Total tokens |
| `is_estimated` | BOOLEAN | Whether count is estimated |
| `created_at` | DATETIME | Timestamp |

### knowledge_entries

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID PK | Entry identifier |
| `tenant_id` | VARCHAR(255) | Tenant identifier |
| `collection_id` | VARCHAR(255) | Collection identifier (null for tenant-level) |
| `file_id` | UUID | Source file UUID |
| `original_filename` | VARCHAR(500) | Original filename |
| `file_type` | VARCHAR(50) | `company` or `project` |
| `content_type` | VARCHAR(200) | MIME type |
| `gcs_path` | VARCHAR(1000) | GCS storage path |
| `status` | VARCHAR(50) | File status |
| `knowledge_status` | VARCHAR(50) | Indexing status |
| `metadata` | JSONB | Additional metadata |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |

Unique constraint: `(tenant_id, file_id)`

### user_tokens

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PK | Auto-increment ID |
| `user_id` | VARCHAR(255) | User identifier |
| `integration_key` | VARCHAR(100) | Integration key (google, slack, etc.) |
| `provider` | VARCHAR(50) | Provider name |
| `token_type` | VARCHAR(20) | `oauth2`, `api_key`, or `jwt` |
| `encrypted_token_data` | TEXT | Fernet-encrypted JSON blob |
| `scopes` | ARRAY(VARCHAR) | Permission scopes |
| `expires_at` | DATETIME | Token expiration |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active flag |

Unique constraint: `(user_id, integration_key)`

### skills

| Column | Type | Description |
|--------|------|-------------|
| `id` | VARCHAR PK | Skill identifier |
| `name` | VARCHAR | Skill name |
| `description` | TEXT | Description |
| `instructions` | TEXT | Skill instructions |
| `category` | VARCHAR | Category |
| `references` | TEXT | JSON list of `{name, content}` |
| `scripts` | TEXT | JSON list of `{name, content}` |
| `allowed_tools` | TEXT | JSON list of tool names |
| `tags` | VARCHAR | JSON string of tags |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active flag |

### api_keys

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PK | Auto-increment ID |
| `key_hash` | VARCHAR(64) | SHA-256 hash of the API key |
| `name` | VARCHAR(255) | Human-readable name |
| `owner_id` | VARCHAR(255) | Owner identifier |
| `scopes` | TEXT | JSON array of allowed scopes |
| `rate_limit` | INTEGER | Requests per hour (0 = unlimited) |
| `expires_at` | DATETIME | Expiration time |
| `last_used_at` | DATETIME | Last usage |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active flag |

### supervisor_runs

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID PK | Run identifier |
| `team_id` | VARCHAR | Team identifier |
| `session_id` | VARCHAR | Session identifier |
| `user_id` | VARCHAR | User identifier |
| `user_message` | TEXT | Original user message |
| `supervisor_response` | JSONB | Supervisor classification result |
| `worker_agent_id` | VARCHAR | Selected worker agent |
| `execution_engine` | VARCHAR | Engine used |
| `status` | VARCHAR | Run status (pending, running, completed, failed) |
| `execution_output` | TEXT | Execution output |
| `created_at` | DATETIME | Creation timestamp |
| `completed_at` | DATETIME | Completion timestamp |

### execution_jobs

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID PK | Job identifier |
| `supervisor_run_id` | UUID FK | Parent supervisor run |
| `worker_config` | JSONB | Worker configuration snapshot |
| `prompt` | TEXT | Task prompt |
| `execution_target` | JSONB | Target configuration |
| `status` | VARCHAR | Job status (queued, running, awaiting_approval, completed, failed, oom, failed_circuit_open) |
| `result` | JSONB | Execution result |
| `container_id` | VARCHAR | Docker/K8s container ID |
| `target_host` | VARCHAR | Execution host |
| `retry_count` | INTEGER | Retry count |
| `memory_limit_mb` | INTEGER | Memory limit |
| `last_failure_reason` | TEXT | Last failure reason |
| `created_at` | DATETIME | Creation timestamp |
| `started_at` | DATETIME | Start timestamp |
| `completed_at` | DATETIME | Completion timestamp |
| `timeout_at` | DATETIME | Timeout deadline |

### execution_engines

| Column | Type | Description |
|--------|------|-------------|
| `id` | VARCHAR PK | Engine identifier |
| `name` | VARCHAR | Engine name |
| `type` | VARCHAR | Engine type |
| `provider` | VARCHAR | Provider (default: "anthropic") |
| `handler_config` | JSONB | Engine-specific config |
| `description` | TEXT | Description |
| `is_default` | BOOLEAN | Default engine flag |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active flag |

### execution_targets

| Column | Type | Description |
|--------|------|-------------|
| `id` | VARCHAR PK | Target identifier |
| `name` | VARCHAR | Target name |
| `type` | VARCHAR | Target type |
| `connection_config` | JSONB | Connection details |
| `capacity` | JSONB | Resource capacity |
| `worker_pool` | VARCHAR | Worker pool name |
| `created_at` | DATETIME | Creation timestamp |
| `updated_at` | DATETIME | Last update timestamp |
| `is_active` | BOOLEAN | Active flag |

---

## 20. Models Reference

### Request Models

| Model | Endpoint | Description |
|-------|----------|-------------|
| `ChatRequest` | POST /v2/agents/{id}/chat | Agent chat request |
| `CommitRequest` | POST /v2/agents/{id}/chat/commit | Resume paused agent run |
| `CreateAgentRequest` | POST /v2/agents | Create agent |
| `GetAgentsByIdsRequest` | POST /v2/agents/batch | Batch get agents |
| `TeamRunRequest` | POST /v2/teams/{id}/runs | Team run request |
| `TeamCommitRequest` | POST /v2/teams/{id}/runs/commit | Resume paused team run |
| `CreateTeamRequest` | POST /v2/teams | Create team |
| `AddMemberRequest` | POST /v2/teams/{id}/members | Add team member |
| `UpdateMemberRequest` | PUT /v2/teams/{id}/members/{agent_id} | Update team member |
| `PromptCreate` | POST /v2/prompts | Create prompt |
| `PromptUpdate` | PUT /v2/prompts/{id} | Update prompt |
| `KnowledgeEntryCreate` | POST /v2/knowledge/{tenant_id} | Create knowledge entry |
| `KnowledgeEntryUpdate` | PATCH /v2/knowledge/{tenant_id}/files/{file_id} | Update knowledge entry |
| `StoreTokenRequest` | POST /v2/users/{user_id}/tokens | Store token |
| `SkillCreate` | POST /v2/skills | Create skill |
| `SkillUpdate` | PUT /v2/skills/{id} | Update skill |
| `EngineCreateRequest` | POST /v2/engines | Create engine |
| `TargetCreateRequest` | POST /v2/targets | Create target |
| `ApprovalDecisionRequest` | POST /v2/approvals/{job_id}/decide | Submit approval decision |
| `CreateApiKeyRequest` | POST /admin/api-keys | Create API key |
| `UpdateApiKeyRequest` | PATCH /admin/api-keys/{id} | Update API key |

### Response Models

| Model | Description |
|-------|-------------|
| `ChatResponse` | Agent chat response (content, status, run_id, tools) |
| `AgentInfo` | Agent metadata with template, tags, config |
| `CreateAgentResponse` | Agent creation confirmation |
| `DeleteAgentResponse` | Agent deletion confirmation |
| `TeamRunResponse` | Team run response (content, team_id, status, tools) |
| `TeamInfo` | Team metadata with agents and mode |
| `CreateTeamResponse` | Team creation confirmation |
| `TeamSession` | Team session metadata |
| `TeamMemory` | Team memory entry |
| `MemberResponse` | Team member metadata |
| `PromptResponse` | Prompt with template, tags, version |
| `PromptListResponse` | Paginated prompt list |
| `KnowledgeEntryResponse` | Knowledge entry metadata |
| `KnowledgeListResponse` | Paginated knowledge entry list |
| `KnowledgeCreateResponse` | Knowledge creation confirmation |
| `GetTokenResponse` | Token data with auto-refresh status |
| `TokenInfo` | Token metadata (no token data) |
| `StoreTokenResponse` | Token storage confirmation |
| `RefreshTokenResponse` | Token refresh result |
| `DeleteTokenResponse` | Token deletion confirmation |
| `SkillResponse` | Full skill details |
| `SkillInfo` | Lightweight skill metadata |
| `EngineResponse` | Engine metadata |
| `TargetResponse` | Target metadata |
| `ApprovalResponse` | Pending approval details |
| `DecisionResponse` | Approval decision result |
| `CreateApiKeyResponse` | API key creation (includes raw key) |

### Shared Models

| Model | Description |
|-------|-------------|
| `UserProfile` | User profile (profile_id, email, full_name, role, department, skills, tools, tenant_id) |
| `TenantProfile` | Tenant/org profile (tenant_id, name, description, website) |
| `AgentConfigRequest` | Agent config (memory, history, reasoning, worker_config) |
| `AgentConfigResponse` | Agent config in responses |

---

## 21. Supported LLM Models

The `Model` enum defines all supported models:

### OpenAI

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `gpt_5_4` | `gpt-5.4` | GPT-5.4 full |
| `gpt_5_4_mini` | `gpt-5.4-mini` | GPT-5.4 mini |
| `gpt_5_4_nano` | `gpt-5.4-nano` | GPT-5.4 nano |

### Google Gemini (Stable)

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `gemini_2_5_pro` | `gemini-2.5-pro` | Gemini 2.5 Pro (default) |
| `gemini_2_5_flash` | `gemini-2.5-flash` | Gemini 2.5 Flash |
| `gemini_2_5_flash_lite` | `gemini-2.5-flash-lite` | Gemini 2.5 Flash Lite |

### Google Gemini (Preview)

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `gemini_3_1_pro` | `gemini-3.1-pro-preview` | Gemini 3.1 Pro Preview |
| `gemini_3_flash` | `gemini-3-flash-preview` | Gemini 3 Flash Preview |

### Anthropic Claude

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `claude_opus_4_6` | `claude-opus-4-6` | Claude Opus 4.6 |
| `claude_sonnet_4_6` | `claude-sonnet-4-6` | Claude Sonnet 4.6 |
| `claude_haiku_4_5` | `claude-haiku-4-5-20251001` | Claude Haiku 4.5 |

### Provider Detection

The provider is determined automatically from the model ID prefix:
- `gpt-*` -> OpenAI
- `gemini-*` -> Google Gemini
- `claude-*` -> Anthropic

---

## 22. Deployment

### Development (Docker Compose)

The project uses `compose.yaml` at the repo root for local development infrastructure. It spins up PostgreSQL (pgvector) and Qdrant:

```yaml
# compose.yaml (excerpt)
services:
  pgvector:
    image: agnohq/pgvector:16
    ports:
      - "5432:5432"
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"  # REST
      - "6334:6334"  # gRPC
```

Start services and the API:

```bash
docker compose up -d
./scripts/start_server.sh
```

### Production Setup

For production deployment:

1. **PostgreSQL**: Use a managed database (Cloud SQL, RDS, etc.) with the `DB_*` environment variables
2. **Qdrant**: Deploy Qdrant for vector knowledge base or leave `QDRANT_URL` empty for in-memory
3. **Environment**: Set all required environment variables (see Configuration section)
4. **Authentication**: Set `ADMIN_SECRET` and create API keys via the Admin API
5. **Token Encryption**: Generate a Fernet key and set `SECRET_TOKEN_ENC_KEY`

Generate a Fernet key:

```python
from cryptography.fernet import Fernet
print(Fernet.generate_key().decode())
```

### Health Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/health` | GET | None | Database connectivity check |
| `/status` | GET | None | Service status |
| `/version` | GET | None | API version |
| `/` | GET | None | Root with docs link |

```bash
curl http://localhost:8000/health
```

```json
{
  "status": "success"
}
```

---

## 23. Development

### Setup

```bash
# Install dependencies
./scripts/dev_setup.sh && source .venv/bin/activate

# Start PostgreSQL + Qdrant
docker compose up -d

# Start server
./scripts/start_server.sh
```

### Validation

Run both scripts after any code change:

```bash
# Auto-fix formatting and lint issues
./scripts/run_validate.sh

# Quick check (no auto-fix)
./scripts/validate.sh
```

- Line length: 120 characters (ruff)
- Type checking: MyPy with SQLAlchemy plugin

### Testing

```bash
# Run all V2 API tests
TESTING=true pytest tests/v2/

# Run a single test file
TESTING=true python -m unittest tests.test_agent_info_crud

# With coverage
TESTING=true pytest --cov
```

Key testing rules:
- Always use `tests/test_utils.py:create_test_client()` for API tests (prevents CloudSQL connections)
- Never use `create_app()` directly in tests
- Database is mocked with SQLite in-memory
- External services mocked with `unittest.mock`
- Use `AsyncMock` for async mock testing
- Clear `_agent_cache` at test start for isolation

### Database Migrations

```bash
# Run a migration
python db/migrations/run_migration.py <migration_file.sql>

# Verify migrations
python db/migrations/verify_migration.py
```

### Dependency Management

```bash
# Edit pyproject.toml, then generate requirements.txt
./scripts/generate_requirements.sh
```
