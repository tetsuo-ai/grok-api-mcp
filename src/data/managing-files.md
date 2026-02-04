# Managing Files

Learn how to upload, list, retrieve, and delete files using the Files API.

## Prerequisites

- xai-sdk version 1.4.0 or later required

## Uploading Files

Before using files in chat conversations, you need to upload them.

### From File Path

```python
from xai_sdk import Client

client = Client(api_key=os.environ.get("XAI_API_KEY"))

file = client.files.upload(
    file_path="./document.pdf",
    purpose="assistants"
)

print(f"File ID: {file.id}")
```

### From Raw Bytes

```python
with open("document.pdf", "rb") as f:
    content = f.read()

file = client.files.upload(
    file_bytes=content,
    filename="document.pdf",
    purpose="assistants"
)
```

### From BytesIO Object

```python
from io import BytesIO

buffer = BytesIO(content)
file = client.files.upload(
    file_bytes=buffer,
    filename="document.pdf",
    purpose="assistants"
)
```

### From Open File Handle

```python
with open("document.pdf", "rb") as f:
    file = client.files.upload(
        file_handle=f,
        purpose="assistants"
    )
```

### Using OpenAI SDK

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

file = client.files.create(
    file=open("document.pdf", "rb"),
    purpose="assistants"
)
```

## Listing Files

```python
files = client.files.list()

for file in files.data:
    print(f"{file.id}: {file.filename} ({file.status})")
```

### With Filtering

```python
# List files with specific purpose
files = client.files.list(purpose="assistants")

# With pagination
files = client.files.list(limit=10, after="file-abc123")
```

## Retrieving File Info

```python
file = client.files.retrieve("file-abc123")

print(f"Filename: {file.filename}")
print(f"Size: {file.bytes} bytes")
print(f"Status: {file.status}")
print(f"Created: {file.created_at}")
```

## Downloading File Content

```python
content = client.files.content("file-abc123")

with open("downloaded.pdf", "wb") as f:
    f.write(content.read())
```

## Deleting Files

```python
result = client.files.delete("file-abc123")

if result.deleted:
    print("File deleted successfully")
```

## File Status

| Status | Description |
|--------|-------------|
| `uploaded` | File uploaded, processing |
| `processed` | Ready to use in conversations |
| `error` | Processing failed |

## Supported File Types

- PDF (.pdf)
- Plain text (.txt)
- Markdown (.md)
- Word documents (.docx)
- CSV (.csv)
- JSON (.json)

## Limits

| Limit | Value |
|-------|-------|
| Max file size | 100 MB |
| Max files per account | 100,000 |
| Max total storage | 100 GB |

## Waiting for Processing

```python
import time

def upload_and_wait(client, file_path, timeout=60):
    # Upload file
    file = client.files.upload(file_path=file_path, purpose="assistants")

    # Wait for processing
    start_time = time.time()
    while time.time() - start_time < timeout:
        file = client.files.retrieve(file.id)
        if file.status == "processed":
            return file
        elif file.status == "error":
            raise Exception("File processing failed")
        time.sleep(2)

    raise Exception("File processing timed out")
```

## Error Handling

```python
try:
    file = client.files.upload(
        file_path="./large_file.pdf",
        purpose="assistants"
    )
except Exception as e:
    if "file_too_large" in str(e):
        print("File exceeds 100MB limit")
    elif "unsupported_file_type" in str(e):
        print("File type not supported")
    elif "quota_exceeded" in str(e):
        print("Storage quota exceeded")
    else:
        raise
```

## Best Practices

1. **Check file status**: Wait for `processed` status before using
2. **Use descriptive names**: Helps identify files later
3. **Clean up unused files**: Delete files when no longer needed
4. **Handle errors gracefully**: Implement retry logic for uploads
5. **Monitor storage**: Track usage to avoid quota limits
