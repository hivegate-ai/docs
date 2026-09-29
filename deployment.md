# Deployment


### Development (Docker Compose)

The project uses Docker Compose for local development infrastructure:

```yaml
# docker-compose.test.yml - Qdrant for vector search
services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"  # REST API
      - "6334:6334"  # gRPC
```

Start services and the API:

```bash
docker compose up -d
./scripts/start_server.sh
```

### Production Setup

For production deployment:

1. **PostgreSQL**: Use a managed database (Cloud SQL, RDS, etc.) with the `DB_*` environment variables
2. **Qdrant**: Deploy Qdrant for vector knowledge base or leave `QDRANT_URL` empty for in-memory
3. **Environment**: Set all required environment variables (see Configuration section)
4. **Authentication**: Set `ADMIN_SECRET` and create API keys via the Admin API
5. **Token Encryption**: Generate a Fernet key and set `SECRET_TOKEN_ENC_KEY`

Generate a Fernet key:

```python
from cryptography.fernet import Fernet
print(Fernet.generate_key().decode())
```

### Health Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/health` | GET | None | Database connectivity check |
| `/status` | GET | None | Service status |
| `/version` | GET | None | API version |
| `/_/version` | GET | None | Commit the running process was built from (`{"commit": "<sha>"}`, or `"unknown"` without `GIT_SHA`). Use it after a deploy to confirm which code is actually answering |
| `/` | GET | None | Root with docs link |

```bash
curl http://localhost:8000/health
```

```json
{
  "status": "success"
}
```

---

