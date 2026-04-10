# Models Reference


### Request Models

| Model | Endpoint | Description |
|-------|----------|-------------|
| `ChatRequest` | POST /v2/agents/{id}/chat | Agent chat request |
| `CommitRequest` | POST /v2/agents/{id}/chat/commit | Resume paused agent run |
| `CreateAgentRequest` | POST /v2/agents | Create agent |
| `GetAgentsByIdsRequest` | POST /v2/agents/batch | Batch get agents |
| `TeamRunRequest` | POST /v2/teams/{id}/runs | Team run request |
| `TeamCommitRequest` | POST /v2/teams/{id}/runs/commit | Resume paused team run |
| `CreateTeamRequest` | POST /v2/teams | Create team |
| `AddMemberRequest` | POST /v2/teams/{id}/members | Add team member |
| `UpdateMemberRequest` | PUT /v2/teams/{id}/members/{agent_id} | Update team member |
| `PromptCreate` | POST /v2/prompts | Create prompt |
| `PromptUpdate` | PUT /v2/prompts/{id} | Update prompt |
| `KnowledgeEntryCreate` | POST /v2/knowledge/{tenant_id} | Create knowledge entry |
| `KnowledgeEntryUpdate` | PATCH /v2/knowledge/{tenant_id}/files/{file_id} | Update knowledge entry |
| `StoreTokenRequest` | POST /v2/users/{user_id}/tokens | Store token |
| `SkillCreate` | POST /v2/skills | Create skill |
| `SkillUpdate` | PUT /v2/skills/{id} | Update skill |
| `EngineCreateRequest` | POST /v2/engines | Create engine |
| `TargetCreateRequest` | POST /v2/targets | Create target |
| `ApprovalDecisionRequest` | POST /v2/approvals/{job_id}/decide | Submit approval decision |
| `CreateApiKeyRequest` | POST /admin/api-keys | Create API key |
| `UpdateApiKeyRequest` | PATCH /admin/api-keys/{id} | Update API key |

### Response Models

| Model | Description |
|-------|-------------|
| `ChatResponse` | Agent chat response (content, status, run_id, tools) |
| `AgentInfo` | Agent metadata with template, tags, config |
| `CreateAgentResponse` | Agent creation confirmation |
| `DeleteAgentResponse` | Agent deletion confirmation |
| `TeamRunResponse` | Team run response (content, team_id, status, tools) |
| `TeamInfo` | Team metadata with agents and mode |
| `CreateTeamResponse` | Team creation confirmation |
| `TeamSession` | Team session metadata |
| `TeamMemory` | Team memory entry |
| `MemberResponse` | Team member metadata |
| `PromptResponse` | Prompt with template, tags, version |
| `PromptListResponse` | Paginated prompt list |
| `KnowledgeEntryResponse` | Knowledge entry metadata |
| `KnowledgeListResponse` | Paginated knowledge entry list |
| `KnowledgeCreateResponse` | Knowledge creation confirmation |
| `GetTokenResponse` | Token data with auto-refresh status |
| `TokenInfo` | Token metadata (no token data) |
| `StoreTokenResponse` | Token storage confirmation |
| `RefreshTokenResponse` | Token refresh result |
| `DeleteTokenResponse` | Token deletion confirmation |
| `SkillResponse` | Full skill details |
| `SkillInfo` | Lightweight skill metadata |
| `EngineResponse` | Engine metadata |
| `TargetResponse` | Target metadata |
| `ApprovalResponse` | Pending approval details |
| `DecisionResponse` | Approval decision result |
| `CreateApiKeyResponse` | API key creation (includes raw key) |

### Shared Models

| Model | Description |
|-------|-------------|
| `UserProfile` | User profile (profile_id, email, full_name, role, department, skills, tools, tenant_id) |
| `TenantProfile` | Tenant/org profile (tenant_id, name, description, website) |
| `AgentConfigRequest` | Agent config (memory, history, reasoning, worker_config) |
| `AgentConfigResponse` | Agent config in responses |

---

