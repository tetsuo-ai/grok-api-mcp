# Collections API Reference

API reference for managing collections and documents.

## Base URL

```
https://management-api.x.ai/v1
```

## Authentication

Use your Management API key:

```
Authorization: Bearer <management-api-key>
```

**Note**: Management keys are different from inference API keys. Get them from xAI Console → Settings → Management Keys.

## Collection Endpoints

### Create Collection

```http
POST /v1/collections
```

**Request Body**:
```json
{
  "name": "my-collection",
  "description": "Optional description"
}
```

**Response**:
```json
{
  "id": "col_abc123",
  "name": "my-collection",
  "description": "Optional description",
  "created_at": "2024-01-15T10:30:00Z",
  "document_count": 0
}
```

### List Collections

```http
GET /v1/collections
```

**Query Parameters**:
- `limit` (optional): Number of results (default: 20, max: 100)
- `offset` (optional): Pagination offset

**Response**:
```json
{
  "data": [
    {
      "id": "col_abc123",
      "name": "my-collection",
      "document_count": 5
    }
  ],
  "has_more": false
}
```

### Get Collection

```http
GET /v1/collections/{collection_id}
```

**Response**:
```json
{
  "id": "col_abc123",
  "name": "my-collection",
  "description": "Collection description",
  "created_at": "2024-01-15T10:30:00Z",
  "document_count": 5,
  "total_size_bytes": 1048576
}
```

### Update Collection

```http
PATCH /v1/collections/{collection_id}
```

**Request Body**:
```json
{
  "name": "updated-name",
  "description": "Updated description"
}
```

### Delete Collection

```http
DELETE /v1/collections/{collection_id}
```

**Response**: 204 No Content

## Document Endpoints

### Upload Document

```http
POST /v1/collections/{collection_id}/documents
```

**Request**: multipart/form-data
- `file`: Document file (PDF, TXT, DOCX, MD)

**Response**:
```json
{
  "id": "doc_xyz789",
  "filename": "document.pdf",
  "size_bytes": 102400,
  "status": "processing",
  "created_at": "2024-01-15T10:35:00Z"
}
```

### List Documents

```http
GET /v1/collections/{collection_id}/documents
```

**Query Parameters**:
- `limit` (optional): Number of results
- `offset` (optional): Pagination offset
- `status` (optional): Filter by status (processing, ready, failed)

**Response**:
```json
{
  "data": [
    {
      "id": "doc_xyz789",
      "filename": "document.pdf",
      "size_bytes": 102400,
      "status": "ready",
      "created_at": "2024-01-15T10:35:00Z"
    }
  ],
  "has_more": false
}
```

### Get Document

```http
GET /v1/collections/{collection_id}/documents/{document_id}
```

### Delete Document

```http
DELETE /v1/collections/{collection_id}/documents/{document_id}
```

**Response**: 204 No Content

## Search in Collection

### Via Inference API

Use the `collections_search` tool with the inference API:

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[{"role": "user", "content": "Search query"}],
    tools=[{
        "type": "collections_search",
        "collections_search": {
            "collection_ids": ["col_abc123"]
        }
    }]
)
```

## Document Status

| Status | Description |
|--------|-------------|
| `processing` | Document is being indexed |
| `ready` | Document is ready for search |
| `failed` | Processing failed |

## Limits

| Resource | Limit |
|----------|-------|
| Max file size | 100 MB |
| Max files (global) | 100,000 |
| Max total storage | 100 GB |
| Max collections | Unlimited |

Contact xAI to increase limits.

## Supported File Types

- PDF (.pdf)
- Plain text (.txt)
- Markdown (.md)
- Word documents (.docx)
- CSV (.csv)

## Error Responses

```json
{
  "error": {
    "code": "invalid_request",
    "message": "File size exceeds maximum limit",
    "param": "file"
  }
}
```

### Error Codes

| Code | Description |
|------|-------------|
| `invalid_request` | Invalid request format |
| `not_found` | Collection/document not found |
| `quota_exceeded` | Storage quota exceeded |
| `file_too_large` | File exceeds size limit |
| `unsupported_format` | File format not supported |
