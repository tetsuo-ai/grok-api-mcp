# Chat with Files

Use uploaded files in chat conversations with Grok.

## Overview

Once you've uploaded files, you can reference them in conversations. When files are attached, the system automatically enables document search capabilities, transforming your request into an agentic workflow.

## Prerequisites

- xai-sdk version 1.4.0 or later required
- Files must be uploaded first (see [Managing Files](./managing-files.md))

## Basic Usage

### Using the file() Helper

```python
from xai_sdk import Client, file

client = Client(api_key=os.environ.get("XAI_API_KEY"))

# Reference a previously uploaded file
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": [
                "Summarize this document",
                file("file-abc123")  # File ID from upload
            ]
        }
    ]
)

print(response.choices[0].message.content)
```

### Using OpenAI SDK

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
            "content": "What are the key points in this document?",
            "attachments": [
                {"file_id": "file-abc123"}
            ]
        }
    ]
)
```

## Automatic Document Search

When files are attached, the system automatically activates the `attachment_search` tool:

1. The request becomes an agentic workflow
2. Grok intelligently searches through your documents
3. Grok reasons over the content to answer questions

## Multiple Files

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": [
                "Compare these two reports",
                file("file-report1"),
                file("file-report2")
            ]
        }
    ]
)
```

## Combining Files with Images

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Compare this chart with the data in the document"},
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.com/chart.png"}
                }
            ],
            "attachments": [
                {"file_id": "file-data-report"}
            ]
        }
    ]
)
```

## Streaming with Files

```python
stream = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": "Analyze this document",
            "attachments": [{"file_id": "file-abc123"}]
        }
    ],
    stream=True
)

for chunk in stream:
    # Tool call notifications show when searching documents
    if chunk.choices[0].delta.tool_calls:
        print("Searching documents...")
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

## Tool Call Visibility

In streaming mode, you can see when the model is searching your documents via tool call notifications.

## Recommended Models

For best document understanding:
- **grok-4** - Best understanding for complex documents
- **grok-4-fast** - Good balance of speed and understanding

## Pricing

Document search is billed at **$10 per 1,000 tool invocations**, in addition to standard token costs.

Each time the model searches your documents counts as one tool invocation.

## Batch Mode Limitation

File attachments with document search are agentic requests and **do not support batch mode** (n > 1).

## Best Practices

1. **Use descriptive filenames**: Helps the model understand document purpose
2. **Ask specific questions**: More focused queries yield better results
3. **Use streaming**: See real-time progress during document analysis
4. **Choose appropriate models**: grok-4 for complex documents
5. **Combine modalities**: Mix files with images when helpful

## Error Handling

```python
try:
    response = client.chat.completions.create(
        model="grok-4",
        messages=[
            {
                "role": "user",
                "content": "Summarize this",
                "attachments": [{"file_id": "file-invalid"}]
            }
        ]
    )
except Exception as e:
    if "file_not_found" in str(e):
        print("File does not exist or has been deleted")
    elif "file_not_processed" in str(e):
        print("File is still processing, try again later")
    else:
        raise
```
