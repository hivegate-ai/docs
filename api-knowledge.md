# Knowledge API


Base path: `/v2/knowledge`

Knowledge entries support dual-level isolation:
- **Tenant-level**: Shared across all collections for an organization (`/v2/knowledge/{tenant_id}`)
- **Collection-level**: Scoped to a specific project or collection (`/v2/knowledge/{tenant_id}/{collection_id}`)

### Create Tenant Knowledge Entry

```bash
curl -X POST http://localhost:8000/v2/knowledge/tenant-acme \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "file_id": "550e8400-e29b-41d4-a716-446655440001",
    "original_filename": "company-handbook.pdf",
    "file_type": "company",
    "content_type": "application/pdf",
    "gcs_path": "gs://acme-docs/company-handbook.pdf",
    "status": "active",
    "metadata": { "department": "HR", "year": 2025 }
  }'
```

**Request Body** (`KnowledgeEntryCreate`):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file_id` | string (UUID) | Yes | UUID of the source file |
| `original_filename` | string | Yes | Original filename |
| `file_type` | string | Yes | `company` or `project` |
| `content_type` | string | No | MIME type |
| `gcs_path` | string | No | Google Cloud Storage path |
| `status` | string | No (default: "active") | File status |
| `metadata` | object | No | Additional metadata |

**Response** `201 Created`:

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "tenant_id": "tenant-acme",
  "collection_id": null,
  "file_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "active",
  "knowledge_status": "indexing",
  "created_at": "2025-01-15T10:00:00",
  "updated_at": "2025-01-15T10:00:00",
  "message": "File added to tenant knowledge base. Knowledge indexing is processing asynchronously."
}
```

### List Tenant Knowledge Entries

```bash
curl "http://localhost:8000/v2/knowledge/tenant-acme?limit=50&offset=0" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`:

```json
{
  "files": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "tenant_id": "tenant-acme",
      "collection_id": null,
      "file_id": "550e8400-e29b-41d4-a716-446655440001",
      "original_filename": "company-handbook.pdf",
      "file_type": "company",
      "content_type": "application/pdf",
      "gcs_path": "gs://acme-docs/company-handbook.pdf",
      "status": "active",
      "knowledge_status": "indexed",
      "metadata": { "department": "HR", "year": 2025 },
      "created_at": "2025-01-15T10:00:00",
      "updated_at": "2025-01-15T10:05:00"
    }
  ],
  "pagination": {
    "total": 1,
    "limit": 50,
    "offset": 0,
    "has_more": false
  }
}
```

### Get Tenant Knowledge Entry

```bash
curl http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `200 OK`: Returns `KnowledgeEntryResponse` (same schema as list items).

### Update Tenant Knowledge Entry

```bash
curl -X PATCH http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: agk_abc123def456" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "archived",
    "metadata": { "department": "HR", "year": 2025, "archived_reason": "superseded" }
  }'
```

**Request Body** (`KnowledgeEntryUpdate`): All fields optional (`status`, `knowledge_status`, `metadata`).

**Response** `200 OK`: Returns updated `KnowledgeEntryResponse`.

### Delete Tenant Knowledge Entry

```bash
# Soft delete
curl -X DELETE http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: agk_abc123def456"

# Hard delete
curl -X DELETE "http://localhost:8000/v2/knowledge/tenant-acme/files/550e8400-e29b-41d4-a716-446655440001?hard_delete=true" \
  -H "X-API-Key: agk_abc123def456"
```

**Response** `204 No Content`

### Collection-Level Knowledge Endpoints

All tenant-level endpoints have collection-level equivalents at `/v2/knowledge/{tenant_id}/{collection_id}`:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/{tenant_id}/{collection_id}` | POST | Create collection knowledge entry |
| `/{tenant_id}/{collection_id}` | GET | List collection knowledge entries |
| `/{tenant_id}/{collection_id}/files/{file_id}` | GET | Get collection knowledge entry |
| `/{tenant_id}/{collection_id}/files/{file_id}` | PATCH | Update collection knowledge entry |
| `/{tenant_id}/{collection_id}/files/{file_id}` | DELETE | Delete collection knowledge entry |

Request and response models are identical to the tenant-level endpoints.

---

