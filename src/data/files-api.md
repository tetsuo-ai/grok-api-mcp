# Files API

The Files API enables you to upload documents and use them in chat conversations with Grok.

## Overview

When you attach files to a chat message, the system automatically activates the `attachment_search` tool, transforming your request into an agentic workflow.

## Uploading Files

### Endpoint

```
POST https://api.x.ai/v1/files
```

### Basic Usage

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

# Upload a file
with open("document.pdf", "rb") as f:
    file = client.files.create(
        file=f,
        purpose="assistants"
    )

print(f"File ID: {file.id}")
```

### Supported File Types

- PDF (.pdf)
- Text (.txt)
- Markdown (.md)
- Word documents (.docx)
- And more

### File Limits

| Limit | Value |
|-------|-------|
| Maximum file size | 100MB |
| Maximum files (global) | 100,000 |
| Maximum total size | 100GB |

Contact xAI to increase these limits.

## Chat with Files

### Basic File Chat

```python
# Upload file first
file = client.files.create(
    file=open("report.pdf", "rb"),
    purpose="assistants"
)

# Use file in chat
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": "Summarize the key findings in this document",
            "attachments": [
                {"file_id": file.id}
            ]
        }
    ]
)

print(response.choices[0].message.content)
```

### Multiple Files

```python
file1 = client.files.create(file=open("doc1.pdf", "rb"), purpose="assistants")
file2 = client.files.create(file=open("doc2.pdf", "rb"), purpose="assistants")

response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": "Compare these two documents",
            "attachments": [
                {"file_id": file1.id},
                {"file_id": file2.id}
            ]
        }
    ]
)
```

### Combining Files with Images

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Compare this chart with the data in the PDF"},
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.com/chart.png"}
                }
            ],
            "attachments": [
                {"file_id": file.id}
            ]
        }
    ]
)
```

## Managing Files

### List Files

```python
files = client.files.list()

for file in files.data:
    print(f"{file.id}: {file.filename} ({file.bytes} bytes)")
```

### Retrieve File Info

```python
file = client.files.retrieve("file-abc123")
print(f"Filename: {file.filename}")
print(f"Size: {file.bytes}")
print(f"Created: {file.created_at}")
```

### Delete File

```python
client.files.delete("file-abc123")
```

### Download File Content

```python
content = client.files.content("file-abc123")
with open("downloaded.pdf", "wb") as f:
    f.write(content.read())
```

## Recommended Models

For best document understanding:
- **grok-4-fast** - Good balance of speed and understanding
- **grok-4** - Best understanding, slower

## Agentic Behavior

When files are attached, the system:
1. Activates `attachment_search` tool automatically
2. Indexes the document content
3. Searches relevant sections based on your query
4. Returns contextually relevant answers

## Batch Mode Limitation

File attachments with document search are agentic requests and **do not support batch mode** (n > 1).

## Error Handling

```python
try:
    file = client.files.create(
        file=open("large_file.pdf", "rb"),
        purpose="assistants"
    )
except Exception as e:
    if "file_too_large" in str(e):
        print("File exceeds size limit")
    elif "unsupported_format" in str(e):
        print("File format not supported")
    else:
        print(f"Upload failed: {e}")
```

## Best Practices

1. **Organize files**: Use meaningful filenames for easier management
2. **Clean up**: Delete files when no longer needed
3. **Chunk large documents**: For very large documents, consider splitting
4. **Use appropriate models**: Choose grok-4 for complex document analysis
5. **Be specific in queries**: Help the model find relevant sections
