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
| `XAI_API_KEY` | xAI Grok API key | (empty) |
| `ZAI_API_KEY` | Z.ai GLM API key | (empty) |
| `DEEPSEEK_API_KEY` | DeepSeek API key | (empty) |
| `DEFAULT_CHAT_MODEL` | Model used when a request doesn't set `model`: a model id or a `*-latest` alias ([LLM models](/docs/llm-models/)). An unknown value stops the server at startup | `gemini-3-flash-preview` |
| `PROMPT_STORAGE_BACKEND` | Where prompt templates are stored: `postgres`, `langsmith`, or `service` (needs `SERVICE_PROMPTS`) | `postgres` |
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
| `GIT_SHA` | Commit the image was built from, returned by `GET /_/version`. Set at build time with `--build-arg GIT_SHA=...` | `unknown` |

---

