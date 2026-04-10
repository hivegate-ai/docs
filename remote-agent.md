# Remote Agent Service


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

