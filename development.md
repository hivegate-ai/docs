# Development


### Setup

```bash
# Install dependencies
./scripts/dev_setup.sh && source .venv/bin/activate

# Start PostgreSQL + Qdrant
docker compose up -d

# Start server
./scripts/start_server.sh
```

### Validation

Run both scripts after any code change:

```bash
# Auto-fix formatting and lint issues
./scripts/run_validate.sh

# Quick check (no auto-fix)
./scripts/validate.sh
```

- Line length: 120 characters (ruff)
- Type checking: MyPy with SQLAlchemy plugin

### Testing

```bash
# Run all V2 API tests
TESTING=true pytest tests/v2/

# Run a single test file
TESTING=true python -m unittest tests.test_agent_info_crud

# With coverage
TESTING=true pytest --cov
```

Key testing rules:
- Always use `tests/test_utils.py:create_test_client()` for API tests (prevents CloudSQL connections)
- Never use `create_app()` directly in tests
- Database is mocked with SQLite in-memory
- External services mocked with `unittest.mock`
- Use `AsyncMock` for async mock testing
- Clear `_agent_cache` at test start for isolation

### Database Migrations

```bash
# Run a migration
python db/migrations/run_migration.py <migration_file.sql>

# Verify migrations
python db/migrations/verify_migration.py
```

### Dependency Management

```bash
# Edit pyproject.toml, then generate requirements.txt
./scripts/generate_requirements.sh
```
