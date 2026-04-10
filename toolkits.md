# Toolkits


Toolkits are Agno `Toolkit` adapters that provide agents with access to external services.

### Architecture

```
Toolkits (Agno adapters)  -->  workspace_suite (vendor-agnostic)  -->  Google/Microsoft APIs
```

All workspace toolkits inherit from `BaseToolkit(Toolkit, ABC)` in `toolkits/base.py`.

### CalendarToolkit

Multi-provider calendar management (Google Calendar, Microsoft Outlook).

**Capabilities**: List events, create events, update events, cancel meetings, check availability, find free slots.

```python
from toolkits import CalendarToolkit

calendar = CalendarToolkit(
    user_id="user-42",
    organizer_email="jane@acme.com",
    service_name="google_calendar",
    fetch_token_func=fetch_access_token,
)
```

### EmailToolkit

Multi-provider email management (Gmail, Outlook Mail).

**Capabilities**: Read emails, send emails, reply to threads, search inbox, manage labels/folders.

```python
from toolkits import EmailToolkit

email = EmailToolkit(
    user_id="user-42",
    service_name="gmail",
    fetch_token_func=fetch_access_token,
)
```

### ContactsToolkit

Multi-provider contact management (Google Contacts, Microsoft People).

**Capabilities**: Search contacts, get contact details, create contacts, update contacts.

```python
from toolkits import ContactsToolkit

contacts = ContactsToolkit(
    user_id="user-42",
    service_name="google_contacts",
    fetch_token_func=fetch_access_token,
)
```

### DriveToolkit

Multi-provider file management (Google Drive, OneDrive).

**Capabilities**: List files, search files, read file contents, upload files.

```python
from toolkits import DriveToolkit

drive = DriveToolkit(
    user_id="user-42",
    service_name="google_drive",
    fetch_token_func=fetch_access_token,
)
```

### ClaudeCodeToolkit

Dispatches work to Claude Code CLI with MCP plugins and permission hooks. Used by supervisor worker agents.

**Capabilities**: Execute coding tasks in a repository with configurable permissions, MCP servers, and hooks.

**Execution modes**: `local`, `ssh`, `remote_service`.

**Confirmation required**: `run_claude_code` requires human confirmation before execution.

```python
from toolkits.claude_code import ClaudeCodeToolkit

claude_code = ClaudeCodeToolkit(
    user_id="user-42",
    worker_config=worker_config,
    execution_target_type="local",
)
```

### ManagedAgentsToolkit

Dispatches work to managed agent APIs (Anthropic, OpenAI, Google, or custom webhook).

**Providers**: `anthropic`, `openai`, `google`, `webhook`.

**Confirmation required**: `run_managed_agent` requires human confirmation before execution.

```python
from toolkits.managed_agents import ManagedAgentsToolkit

managed = ManagedAgentsToolkit(
    user_id="user-42",
    worker_config=worker_config,
    provider_name="anthropic",
)
```

### HITL Confirmation Workflow

Toolkits register tools with `requires_confirmation_tools`. When an agent calls a confirmation-required tool:

1. The run pauses and returns `status: "paused"` with a `tools` array
2. The client inspects/edits tool arguments
3. The client sends confirmed tools to the `/chat/commit` or `/runs/commit` endpoint
4. The agent resumes execution with the updated tools

---

