# Collections Metadata

Add structured attributes to documents in collections for filtering, improved retrieval, and data integrity.

## Overview

Metadata enables:

1. **Filtered retrieval** - Narrow search results to documents matching specific criteria
2. **Contextual embeddings** - Inject metadata into chunks to improve retrieval accuracy
3. **Data integrity constraints** - Enforce required fields or uniqueness across documents

## Defining Metadata Fields

Define metadata fields when creating a collection using `field_definitions`:

```python
from xai_sdk import Client

client = Client(api_key=os.environ.get("XAI_API_KEY"))

collection = client.collections.create(
    name="product-docs",
    description="Product documentation",
    field_definitions=[
        {
            "name": "product_version",
            "type": "string",
            "required": True
        },
        {
            "name": "doc_type",
            "type": "string",
            "inject_into_chunk": True
        },
        {
            "name": "document_id",
            "type": "string",
            "unique": True
        }
    ]
)
```

## Field Properties

| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Field name (required) |
| `type` | string | Data type: `string`, `number`, `boolean` |
| `required` | boolean | Document uploads must include this field (default: `false`) |
| `unique` | boolean | Only one document can have a given value for this field |
| `inject_into_chunk` | boolean | Prepends field value to every embedding chunk |

### Required Fields

When `required: true`, all document uploads must include this field:

```python
# This will fail if product_version is required but not provided
document = client.collections.upload_document(
    collection_id=collection.id,
    file_path="./guide.pdf",
    fields={
        "product_version": "2.0"  # Required field
    }
)
```

### Unique Fields

When `unique: true`, no two documents can have the same value:

```python
# This ensures each document has a unique document_id
document = client.collections.upload_document(
    collection_id=collection.id,
    file_path="./guide.pdf",
    fields={
        "document_id": "DOC-001"  # Must be unique
    }
)
```

### Inject Into Chunk

When `inject_into_chunk: true`, the field value is prepended to every embedding chunk, improving retrieval accuracy:

```python
# With inject_into_chunk enabled for doc_type,
# each chunk will be prefixed with "doc_type: API Reference"
document = client.collections.upload_document(
    collection_id=collection.id,
    file_path="./api-ref.pdf",
    fields={
        "doc_type": "API Reference"
    }
)
```

## Uploading Documents with Metadata

Include metadata in the `fields` parameter as a JSON object:

```python
document = client.collections.upload_document(
    collection_id=collection.id,
    file_path="./user-guide.pdf",
    fields={
        "product_version": "2.0",
        "doc_type": "User Guide",
        "document_id": "UG-002",
        "last_updated": "2024-01-15"
    }
)
```

## Filtering Search Results

Use AIP-160 filter syntax to narrow search results by metadata.

### Filter Syntax

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equals | `product_version = "2.0"` |
| `!=` | Not equals | `doc_type != "Draft"` |
| `<` | Less than | `priority < 5` |
| `>` | Greater than | `priority > 1` |
| `<=` | Less than or equal | `version <= "3.0"` |
| `>=` | Greater than or equal | `version >= "1.0"` |
| `AND` | Logical AND | `version = "2.0" AND type = "Guide"` |
| `OR` | Logical OR | `type = "Guide" OR type = "Reference"` |

### Filter Examples

```python
# Filter by product version
results = client.collections.search(
    collection_id=collection.id,
    query="authentication",
    filter='product_version = "2.0"'
)

# Combine multiple filters
results = client.collections.search(
    collection_id=collection.id,
    query="getting started",
    filter='product_version = "2.0" AND doc_type = "User Guide"'
)

# Use OR for multiple values
results = client.collections.search(
    collection_id=collection.id,
    query="API endpoints",
    filter='doc_type = "API Reference" OR doc_type = "Developer Guide"'
)
```

### Filtering in Chat

Apply filters when using collections_search in chat:

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {"role": "user", "content": "How do I authenticate?"}
    ],
    tools=[{
        "type": "collections_search",
        "collections_search": {
            "collection_ids": ["col_abc123"],
            "filter": 'product_version = "2.0"'
        }
    }]
)
```

## Best Practices

1. **Plan your schema** - Define fields before uploading documents
2. **Use inject_into_chunk** - Improves retrieval for categorical fields
3. **Enforce data integrity** - Use `required` and `unique` where appropriate
4. **Keep filters simple** - Complex filters may impact performance
5. **Use consistent values** - String comparisons are exact matches (case-sensitive)

## Limitations

- String comparisons require exact matches (no wildcard patterns)
- Filter syntax follows AIP-160 specification
- Metadata changes require re-uploading documents
- Maximum number of fields per collection may be limited
