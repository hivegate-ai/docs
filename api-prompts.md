# Prompts API


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

