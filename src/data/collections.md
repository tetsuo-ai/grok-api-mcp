# Collections

Collections is xAI's state-of-the-art RAG (Retrieval-Augmented Generation) system built directly into the API. It serves as xAI's embeddings solution, allowing you to upload and search through entire datasets without managing indexing and retrieval infrastructure.

## Overview

Collections enable:
- **Automatic embeddings generation** - Embeddings created on document upload
- **Multi-mode retrieval** - Keyword, semantic, or hybrid search
- **OCR & layout-aware parsing** - Extracts text while preserving structure (PDF layouts, Excel tables, code syntax)
- **Enterprise knowledge bases** - From PDFs and Excel sheets to entire codebases
- **State-of-the-art performance** - Matches or outperforms leading models in real-world RAG tasks across finance, legal, and coding domains

## Pricing

| Operation | Cost |
|-----------|------|
| File indexing & storage | Free (first week) |
| Retrieval/searches | $2.50 per 1,000 searches |

## Limits

| Limit | Value |
|-------|-------|
| Maximum file size | 100MB |
| Maximum number of files | 100,000 files (globally) |
| Maximum total size | 100GB |

Contact xAI to increase any of these limits.

## SDK Requirements

Version **1.4.0** of the `xai-sdk` package is required to use the `collections_search` tool in the agentic tool calling API.

To use the Collections API, you need to create a **Management API Key** with the `AddFileToCollection` permission.

## Creating a Collection

### Using Management API

```python
import requests

management_key = os.environ.get("XAI_MANAGEMENT_KEY")

response = requests.post(
    "https://management-api.x.ai/v1/collections",
    headers={
        "Authorization": f"Bearer {management_key}",
        "Content-Type": "application/json"
    },
    json={
        "name": "company-knowledge-base",
        "description": "Internal company documents and policies"
    }
)

collection = response.json()
print(f"Collection ID: {collection['id']}")
```

## Adding Documents to Collection

```python
# Upload file to collection
with open("policy.pdf", "rb") as f:
    response = requests.post(
        f"https://management-api.x.ai/v1/collections/{collection_id}/documents",
        headers={
            "Authorization": f"Bearer {management_key}"
        },
        files={"file": f}
    )

document = response.json()
print(f"Document ID: {document['id']}")
```

## Searching Collections

### Using collections_search Tool

The `collections_search` built-in tool searches through your document collections:

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": "What is our company's vacation policy?"
        }
    ],
    tools=[
        {
            "type": "collections_search",
            "collections": ["collection-id-123"]
        }
    ]
)

print(response.choices[0].message.content)
```

## Managing Collections

### List Collections

```python
response = requests.get(
    "https://management-api.x.ai/v1/collections",
    headers={"Authorization": f"Bearer {management_key}"}
)

collections = response.json()
for collection in collections["data"]:
    print(f"{collection['id']}: {collection['name']}")
```

### Get Collection Details

```python
response = requests.get(
    f"https://management-api.x.ai/v1/collections/{collection_id}",
    headers={"Authorization": f"Bearer {management_key}"}
)

collection = response.json()
print(f"Name: {collection['name']}")
print(f"Document count: {collection['document_count']}")
```

### Delete Collection

```python
response = requests.delete(
    f"https://management-api.x.ai/v1/collections/{collection_id}",
    headers={"Authorization": f"Bearer {management_key}"}
)
```

## Managing Documents

### List Documents in Collection

```python
response = requests.get(
    f"https://management-api.x.ai/v1/collections/{collection_id}/documents",
    headers={"Authorization": f"Bearer {management_key}"}
)

documents = response.json()
for doc in documents["data"]:
    print(f"{doc['id']}: {doc['filename']}")
```

### Delete Document

```python
response = requests.delete(
    f"https://management-api.x.ai/v1/collections/{collection_id}/documents/{document_id}",
    headers={"Authorization": f"Bearer {management_key}"}
)
```

## Use Cases

### Enterprise Knowledge Base

```python
# Create collection for each department
hr_collection = create_collection("hr-policies")
engineering_collection = create_collection("engineering-docs")
legal_collection = create_collection("legal-documents")

# Query across specific collections
response = client.chat.completions.create(
    model="grok-4",
    messages=[{"role": "user", "content": "What are the code review guidelines?"}],
    tools=[{
        "type": "collections_search",
        "collections": [engineering_collection["id"]]
    }]
)
```

### Customer Support

```python
# Collection with product documentation
support_collection = create_collection("product-support")

# Add FAQs, manuals, troubleshooting guides
add_documents(support_collection["id"], ["faq.pdf", "manual.pdf", "troubleshooting.pdf"])

# Support agent query
response = client.chat.completions.create(
    model="grok-4",
    messages=[{"role": "user", "content": "Customer asks: How do I reset my password?"}],
    tools=[{
        "type": "collections_search",
        "collections": [support_collection["id"]]
    }]
)
```

## Best Practices

1. **Organize by topic**: Create separate collections for different domains
2. **Keep documents updated**: Refresh documents when content changes
3. **Use descriptive names**: Make collection names self-explanatory
4. **Monitor usage**: Track which collections are most queried
5. **Access control**: Manage who can add/remove documents
