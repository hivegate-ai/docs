# Agents Gateway Documentation

Comprehensive documentation for the [Agents Gateway API](https://github.com/liberzon/agno-agent-api).

## View Online

After building, open `_build/html/index.html` in your browser.

## Build

```bash
pip install sphinx sphinx-rtd-theme myst-parser
make html
```

Output: `_build/html/index.html`

## Structure

```
docs/
  conf.py                 # Sphinx configuration
  index.md                # Table of contents
  Makefile                # Build commands
  API_DOCUMENTATION.md    # Single-page reference (3,000+ lines)

  # Getting Started
  overview.md             # Architecture, key concepts
  quickstart.md           # Setup and first API call
  authentication.md       # API keys, admin auth, dev mode
  configuration.md        # Environment variables

  # API Reference
  api-agents.md           # /v2/agents endpoints
  api-teams.md            # /v2/teams endpoints (coordinate + supervisor)
  api-prompts.md          # /v2/prompts CRUD
  api-knowledge.md        # /v2/knowledge (tenant + collection level)
  api-tokens.md           # /v2/users/{id}/tokens
  api-skills.md           # /v2/skills CRUD
  api-engines.md          # /v2/engines (execution engine registry)
  api-targets.md          # /v2/targets (execution target registry)
  api-approvals.md        # /v2/approvals (HITL approval flow)
  api-admin.md            # /admin cache, API keys, background tasks

  # Supervisor Platform
  supervisor-platform.md  # Architecture, classification, worker types
  agent-packs.md          # YAML pack format, CLI
  remote-agent.md         # Docker/K8s container execution
  toolkits.md             # Calendar, Email, Drive, ClaudeCode, ManagedAgents

  # Reference
  models-reference.md     # All Pydantic request/response models
  database-schema.md      # All 12+ database tables
  llm-models.md           # Supported LLM models (OpenAI, Gemini, Claude)
  deployment.md           # Docker Compose, production setup
  development.md          # Dev setup, validation, testing
```
