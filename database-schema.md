# Database Schema


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

