# Using Collections via API

Manage collections programmatically using the xAI SDK and REST API.

## Prerequisites

To use the Collections API, you need to create a **Management API Key** with the `AddFileToCollection` permission. This permission is required for uploading documents to collections.

## Creating a Collection

### Using xAI SDK

```python
from xai_sdk import Client

client = Client(api_key=os.environ.get("XAI_API_KEY"))

# Create a collection
collection = client.collections.create(
    name="my-knowledge-base",
    description="Company documentation"
)

print(f"Collection ID: {collection.id}")
```

### Using REST API

```bash
curl -X POST https://management-api.x.ai/v1/collections \
  -H "Authorization: Bearer $MANAGEMENT_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-knowledge-base",
    "description": "Company documentation"
  }'
```

## Uploading Documents

### Using xAI SDK

```python
# Upload from file path
document = client.collections.upload_document(
    collection_id=collection.id,
    file_path="./document.pdf"
)

# Upload from bytes
with open("document.pdf", "rb") as f:
    document = client.collections.upload_document(
        collection_id=collection.id,
        file_bytes=f.read(),
        filename="document.pdf"
    )

# Upload from BytesIO
from io import BytesIO
buffer = BytesIO(content)
document = client.collections.upload_document(
    collection_id=collection.id,
    file_bytes=buffer,
    filename="document.pdf"
)
```

### Using REST API

```bash
curl -X POST "https://management-api.x.ai/v1/collections/{collection_id}/documents" \
  -H "Authorization: Bearer $MANAGEMENT_KEY" \
  -F "file=@document.pdf"
```

## Embeddings

You can choose to generate embeddings on document upload:

```python
document = client.collections.upload_document(
    collection_id=collection.id,
    file_path="./document.pdf",
    generate_embeddings=True  # Recommended: leave enabled
)
```

We recommend leaving `generate_embeddings` enabled for optimal search performance.

## Search Methods

Three search methods are available:

### Keyword Search
Traditional keyword matching.

### Semantic Search
Meaning-based search using embeddings.

### Hybrid Search (Default)
Combines both keyword and semantic methods. Generally delivers the best and most comprehensive results.

The hybrid approach balances precision and recall, making it the recommended default for most queries.

```python
# Search with specific method
results = client.collections.search(
    collection_id=collection.id,
    query="company policies",
    search_method="hybrid"  # "keyword", "semantic", or "hybrid"
)
```

## Searching via Responses API

Use the `file_search` tool to search collections during chat:

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {"role": "user", "content": "What does the policy say about vacation time?"}
    ],
    tools=[
        {
            "type": "file_search",
            "file_search": {
                "collection_ids": ["col_abc123"]
            }
        }
    ]
)
```

See the [Collections Search Tool](./collections-search-tool.md) guide for more details.

## Listing Collections

```python
collections = client.collections.list()

for collection in collections:
    print(f"{collection.id}: {collection.name} ({collection.document_count} docs)")
```

## Listing Documents

```python
documents = client.collections.list_documents(collection_id=collection.id)

for doc in documents:
    print(f"{doc.id}: {doc.filename} - {doc.status}")
```

## Deleting Documents

```python
client.collections.delete_document(
    collection_id=collection.id,
    document_id=document.id
)
```

## Deleting Collections

```python
client.collections.delete(collection_id=collection.id)
```

**Warning**: Deleting a collection removes all documents within it.

## Document Status

| Status | Description |
|--------|-------------|
| `processing` | Document is being indexed |
| `ready` | Document is ready for search |
| `failed` | Processing failed |

## Error Handling

```python
try:
    document = client.collections.upload_document(
        collection_id=collection.id,
        file_path="./large_file.pdf"
    )
except Exception as e:
    if "file_too_large" in str(e):
        print("File exceeds 100MB limit")
    elif "quota_exceeded" in str(e):
        print("Storage quota exceeded")
    else:
        raise
```

## Limits

| Resource | Limit |
|----------|-------|
| Max file size | 100 MB |
| Max files per collection | 100,000 |
| Max total storage | 100 GB |

Contact xAI to increase limits.
