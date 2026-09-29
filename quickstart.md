# Quick Start


### Prerequisites

- Python 3.11+
- Docker (for PostgreSQL and Qdrant)
- An API key for a model provider (the examples use Gemini)

### Setup

```bash
git clone https://github.com/hivegate-ai/hivegate
cd hivegate
cp .env.example .env
```

Edit `.env` **before starting the server**. The server reads it once at startup, so variables exported in another terminal won't reach it.

- Set `GOOGLE_API_KEY`, or another provider's key.
- For local development, uncomment `AUTH_DISABLED=true` so you can call `/v2` without an API key. Never enable it on a deployment other people can reach.

The database settings in `.env.example` already match the Postgres that Docker Compose starts.

```bash
# Start infrastructure (Postgres + Qdrant, seeds demo agents)
docker compose up -d

# Set up the Python environment and start the API server (port 8000)
./scripts/dev_setup.sh && source .venv/bin/activate
./scripts/start_server.sh
```

### First API Call

```bash
# Check health (no auth required)
curl http://localhost:8000/health

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
    "user_id": "user-1",
    "session_id": "session-1",
    "timezone": "UTC",
    "locale": "en-US"
  }'
```

---

