# Configuration


### Required Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_HOST` | PostgreSQL host | `127.0.0.1` |
| `DB_PORT` | PostgreSQL port | `5432` |
| `DB_USER` | PostgreSQL user | `postgres` |
| `DB_PASS` | PostgreSQL password | (empty) |
| `DB_DATABASE` | PostgreSQL database name | `postgres` |
| `DB_DRIVER` | Database driver | `postgresql+psycopg` |
| `SECRET_TOKEN_ENC_KEY` | Fernet encryption key for token storage | (required in production) |
| `ADMIN_SECRET` | Admin endpoint authentication secret | (required in production) |

### Optional Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `AUTH_DISABLED` | Bypass all authentication | `false` |
| `TESTING` | Enable test mode (SQLite, debug) | `false` |
| `GOOGLE_API_KEY` | Google Gemini API key | (empty) |
| `OPENAI_API_KEY` | OpenAI API key | (empty) |
| `ANTHROPIC_API_KEY` | Anthropic Claude API key | (empty) |
| `QDRANT_URL` | Qdrant vector database URL | (empty, uses in-memory) |
| `QDRANT_API_KEY` | Qdrant authentication key | (empty) |
| `QDRANT_PORT` | Qdrant port | `6333` |
| `SERVICE_PROMPTS` | External prompts service URL | (empty) |
| `SKIP_DB_TABLE_CHECK` | Skip database table creation on startup | `false` |
| `ENABLE_GUARDRAILS` | Enable prompt injection + PII detection | `false` |
| `OTEL_TRACING_BACKEND` | OpenTelemetry tracing backend | `console` |
| `OTEL_LOGGING_BACKEND` | OpenTelemetry logging backend | `console` |
| `BRIGHT_DATA_API_KEY` | Bright Data API key | (empty) |
| `BRIGHT_DATA_WEB_UNLOCKER_ZONE` | Bright Data Web Unlocker zone | `web_unlocker1` |
| `BRIGHT_DATA_SERP_ZONE` | Bright Data SERP zone | `serp_api1` |

---

