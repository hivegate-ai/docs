# Agent Packs


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

The `agents-gateway-tests` repository ships an example `packs/default_supervisor/` pack that can be loaded via the CLI. It includes:

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

