# Using Collections in Console

Create and manage collections through the xAI Console interface.

## Overview

The xAI Console provides a user-friendly interface for:
- Creating collections
- Uploading documents
- Managing collection contents
- Testing collection queries

## Accessing Collections

1. Log in to the xAI Console
2. Navigate to **Collections** in the sidebar
3. View your existing collections or create new ones

## Creating a Collection

### Step 1: Click "Create Collection"

In the Collections section, click the "Create Collection" button.

### Step 2: Configure Collection

- **Name**: Give your collection a descriptive name
- **Description**: Add an optional description

### Step 3: Save

Click "Create" to save your new collection.

## Uploading Documents

### Step 1: Select Collection

Click on the collection you want to add documents to.

### Step 2: Upload Files

- Click "Upload Documents" or drag and drop files
- Select one or more files from your computer
- Supported formats: PDF, TXT, DOCX, MD, and more

### Step 3: Wait for Processing

Documents are processed and indexed automatically. Processing time depends on document size and complexity.

## Managing Documents

### View Documents

- See all documents in a collection
- View document metadata (name, size, upload date)
- Check processing status

### Delete Documents

1. Select the document(s) to delete
2. Click "Delete"
3. Confirm deletion

**Note**: Deleted documents cannot be recovered.

## Testing Queries

### Using the Console

1. Select a collection
2. Open the "Query" or "Test" tab
3. Enter a natural language question
4. View the response with citations

### Example

**Query**: "What is the vacation policy?"

**Response**: "According to the employee handbook, full-time employees receive 15 days of paid vacation per year. [1]"

## Collection Settings

### Access Controls

- View who has access to collections
- Manage permissions (if available)

### Collection Info

- View collection ID (needed for API calls)
- See document count
- Check storage usage

## Using Collections in API

After creating collections in the console, use them via API:

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {"role": "user", "content": "What does the policy say about remote work?"}
    ],
    tools=[
        {
            "type": "collections_search",
            "collections_search": {
                "collection_ids": ["col_abc123"]  # From console
            }
        }
    ]
)
```

## Finding Collection IDs

1. Open the collection in console
2. Look for "Collection ID" in settings or details
3. Copy the ID (format: `col_xxxxx`)

## Best Practices

### Organization

- Create separate collections for different topics
- Use clear, descriptive names
- Add descriptions for team clarity

### Documents

- Use meaningful file names
- Keep documents focused on specific topics
- Update documents when content changes

### Maintenance

- Regularly review collection contents
- Remove outdated documents
- Monitor storage usage

## Limits

| Limit | Value |
|-------|-------|
| Max file size | 100MB |
| Max files | 100,000 (global) |
| Max total storage | 100GB |

Contact xAI to increase limits.

## Troubleshooting

### Document Not Processing

- Check file format is supported
- Verify file isn't corrupted
- Check file size limits

### Query Not Finding Content

- Ensure document processing is complete
- Try different query phrasing
- Check document actually contains the information

### Upload Failures

- Check internet connection
- Verify file size is within limits
- Try uploading fewer files at once
