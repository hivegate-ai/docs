# Skills API


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

