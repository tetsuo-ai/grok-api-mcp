# Image Understanding

Grok models can analyze and understand images, enabling visual question answering, image description, and visual analysis tasks.

## Overview

Image understanding allows Grok to:
- Describe image contents
- Answer questions about images
- Extract text from images (OCR)
- Analyze charts and graphs
- Compare multiple images
- Understand visual context for search results

## Basic Usage

### With Responses API

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

response = client.responses.create(
    model="grok-4",
    input=[
        {
            "type": "input_image",
            "image_url": "https://example.com/image.jpg"
        },
        {
            "type": "input_text",
            "text": "What's in this image?"
        }
    ]
)

print(response.output_text)
```

### With Chat Completions API

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "Describe this image in detail"
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://example.com/image.jpg"
                    }
                }
            ]
        }
    ]
)

print(response.choices[0].message.content)
```

## Supported Image Formats

- **JPEG/JPG**
- **PNG**

## Image Input Methods

### URL

```python
{
    "type": "image_url",
    "image_url": {
        "url": "https://example.com/image.jpg"
    }
}
```

### Base64 Encoded

```python
import base64

with open("image.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

{
    "type": "image_url",
    "image_url": {
        "url": f"data:image/jpeg;base64,{image_data}"
    }
}
```

## Detail Parameter

The `detail` parameter controls image pre-processing:

| Value | Description | Use Case |
|-------|-------------|----------|
| `auto` | System determines resolution (default) | General use |
| `low` | Low-resolution processing | Fast, fewer tokens |
| `high` | High-resolution processing | Fine details needed |

```python
{
    "type": "image_url",
    "image_url": {
        "url": "https://example.com/image.jpg",
        "detail": "high"
    }
}
```

### Token Impact

- **Low detail**: Faster, fewer tokens, may miss fine details
- **High detail**: Slower, more tokens, captures nuanced details

## Image Order

Any image/text input order is accepted:
- Text can precede image
- Image can precede text
- Multiple images and text can be interleaved

```python
messages=[
    {
        "role": "user",
        "content": [
            {"type": "text", "text": "Compare these two images:"},
            {"type": "image_url", "image_url": {"url": "image1.jpg"}},
            {"type": "image_url", "image_url": {"url": "image2.jpg"}},
            {"type": "text", "text": "What are the differences?"}
        ]
    }
]
```

## Image Understanding with Tools

### With Web Search

```python
response = client.chat.completions.create(
    model="grok-4",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What product is this? Find its price online."},
                {"type": "image_url", "image_url": {"url": "product.jpg"}}
            ]
        }
    ],
    tools=[{"type": "web_search"}]
)
```

### Enabling in Search Tools

Set `enable_image_understanding` to true to allow the agent to analyze images found during searches:

```python
tools=[
    {
        "type": "web_search",
        "web_search": {
            "enable_image_understanding": True
        }
    }
]
```

This equips the agent with access to the `view_image` tool, allowing it to interpret images encountered during search.

**Note**: Enabling this feature increases token usage as images are processed and represented as image tokens.

## Use Cases

### Visual Q&A

```python
"What brand is shown in this logo?"
"How many people are in this photo?"
"What color is the car?"
```

### Document Analysis

```python
"Extract the text from this receipt"
"Summarize the information in this chart"
"What does this diagram show?"
```

### Technical Analysis

```python
"Explain this circuit diagram"
"What's wrong with this code screenshot?"
"Analyze this architecture diagram"
```

## Recommended Models

- **grok-4**: Best understanding, recommended for complex analysis
- **grok-4-fast**: Good balance for most use cases

## Best Practices

1. **Use appropriate detail level**: High for text/fine details, low for general content
2. **Provide context**: Tell the model what you're looking for
3. **Clear images**: Better quality images yield better results
4. **Combine with text**: Give relevant context alongside images
5. **Monitor tokens**: Image processing uses additional tokens

## Limitations

- Maximum file size restrictions apply
- Some complex diagrams may be challenging
- Streaming is supported for responses
- Processing time increases with image complexity
