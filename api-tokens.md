# Tokens API


Base path: `/v2/users/{user_id}/tokens`

Manages encrypted OAuth2, API key, and JWT credentials per user per integration. All token data is encrypted with Fernet using the `SECRET_TOKEN_ENC_KEY` environment variable.

### Store Token

```bash
curl -X POST http://localhost:8000/v2/users/user-42/tokens \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "integration_key": "google",
    "provider": "google",
    "token_type": "oauth2",
    "token_data": {
      "access_token": "ya29.a0AfH6SM...",
      "refresh_token": "1//0dx2Xj...",
      "token_type": "Bearer",
      "expires_in": 3600,
      "scope": "https://www.googleapis.com/auth/calendar",
      "client_id": "123456.apps.googleusercontent.com",
      "client_secret": "GOCSPX-..."
    },
    "scopes": ["calendar", "email"],
    "expires_at": "2025-01-16T10:00:00Z"
  }'
```

**Request Body** (`StoreTokenRequest`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `integration_key` | string | Yes | Integration identifier (e.g., "google", "slack", "openai") |
| `provider` | string | Yes | Provider name (whitelisted: google, slack, openai, anthropic, github, microsoft, zoom, dropbox, notion) |
| `token_type` | string | Yes | `oauth2`, `api_key`, or `jwt` |
| `token_data` | TokenData | Yes | Token credentials (access_token, refresh_token, api_key, etc.) |
| `scopes` | string[] | No | Permission scopes |
| `expires_at` | datetime | No | Expiration time |

**Response** `201 Created`:

```json
{
  "integration_key": "google",
  "provider": "google",
  "token_type": "oauth2",
  "message": "Token stored successfully for google",
  "created_at": "2025-01-15T10:00:00"
}
```

### Get Token (with Auto-Refresh)

Retrieves the token and automatically refreshes OAuth2 tokens if expired.

```bash
curl http://localhost:8000/v2/users/user-42/tokens/google \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "integration_key": "google",
  "provider": "google",
  "token_type": "oauth2",
  "token_data": {
    "access_token": "ya29.a0AfH6SM...",
    "refresh_token": "1//0dx2Xj...",
    "token_type": "Bearer"
  },
  "scopes": ["calendar", "email"],
  "expires_at": "2025-01-16T10:00:00",
  "is_expired": false,
  "refreshed": false
}
```

### List User Tokens

Returns metadata only (no token data).

```bash
curl http://localhost:8000/v2/users/user-42/tokens \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
[
  {
    "integration_key": "google",
    "provider": "google",
    "token_type": "oauth2",
    "scopes": ["calendar", "email"],
    "expires_at": "2025-01-16T10:00:00",
    "created_at": "2025-01-15T10:00:00",
    "updated_at": "2025-01-15T10:00:00",
    "is_expired": false
  },
  {
    "integration_key": "openai",
    "provider": "openai",
    "token_type": "api_key",
    "scopes": null,
    "expires_at": null,
    "created_at": "2025-01-14T08:00:00",
    "updated_at": "2025-01-14T08:00:00",
    "is_expired": false
  }
]
```

### Refresh Token

Manually trigger an OAuth2 token refresh.

```bash
curl -X POST http://localhost:8000/v2/users/user-42/tokens/google/refresh \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "integration_key": "google",
  "success": true,
  "message": "Token refreshed successfully",
  "refreshed_at": "2025-01-15T11:00:00"
}
```

### Delete Token

```bash
curl -X DELETE http://localhost:8000/v2/users/user-42/tokens/google \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "integration_key": "google",
  "message": "Token for google deleted successfully"
}
```

---

