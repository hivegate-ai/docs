# Quick Start


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

