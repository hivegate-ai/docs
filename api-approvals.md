# Approvals API


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

