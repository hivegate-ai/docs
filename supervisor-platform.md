# Supervisor Platform


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

