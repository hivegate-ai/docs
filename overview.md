# Overview


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

