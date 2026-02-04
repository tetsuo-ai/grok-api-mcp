# Image Generation and Edit

Generate and edit images using Grok's image capabilities.

> **Note**: You can also animate generated images using [video generation](./video-generation.md).

## Image Generation

### With xAI SDK (Recommended)

The [xAI SDK](https://github.com/xai-org/xai-sdk-python) is the recommended way to generate images:

```python
import os
from xai_sdk import Client

client = Client(api_key=os.getenv('XAI_API_KEY'))

response = client.image.sample(
    model="grok-imagine-image",
    prompt="A cat in a tree",
    image_format="url"
)

print(response.url)
```

By default, `image_format` is `url` and the generated image will be available for download on xAI managed storage.

### With OpenAI SDK

```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

response = client.images.generate(
    model="grok-imagine-image",
    prompt="A futuristic city with flying cars at sunset",
    n=1,
    size="1024x1024"
)

image_url = response.data[0].url
print(image_url)
```

### Endpoint

```
POST https://api.x.ai/v1/images/generations
```

### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model to use (`grok-imagine-image`) |
| `prompt` | string | Text description of the image to generate |
| `n` | integer | Number of images to generate (1-10) |
| `size` | string | Image size ("1024x1024", "1792x1024", "1024x1792") |
| `aspect_ratio` | string | Alternative to size - e.g., "4:3", "16:9", "1:1" |
| `response_format` | string | `url` (default) or `b64_json` |
| `style` | string | Image style ("vivid", "natural") |

> **Note**: The `quality` parameter is **not currently supported** by the xAI API.

### cURL Example

```bash
curl https://api.x.ai/v1/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $XAI_API_KEY" \
  -d '{
    "model": "grok-imagine-image",
    "prompt": "A serene mountain landscape with a lake",
    "n": 1,
    "size": "1024x1024"
  }'
```

### Response Format

```json
{
  "created": 1699000000,
  "data": [
    {
      "url": "https://...",
      "revised_prompt": "A detailed serene mountain landscape..."
    }
  ]
}
```

## Image Editing

Edit existing images with prompts.

### With xAI SDK

```python
import os
import base64
from xai_sdk import Client

client = Client(api_key=os.getenv('XAI_API_KEY'))

with open("cat-in-tree.jpg", "rb") as image_file:
    image_bytes = image_file.read()
    base64_string = base64.b64encode(image_bytes).decode("utf-8")

response = client.image.sample(
    model="grok-imagine-image",
    image_url=f"data:image/jpeg;base64,{base64_string}",
    prompt="Swap the cat in the picture with a dog."
)

print(response.url)
```

### Endpoint

```
POST https://api.x.ai/v1/images/edits
```

### With OpenAI SDK

```python
response = client.images.edit(
    model="grok-imagine-image",
    image=open("original.png", "rb"),
    mask=open("mask.png", "rb"),
    prompt="Add a rainbow in the sky",
    n=1,
    size="1024x1024"
)

edited_url = response.data[0].url
```

### Edit Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `image` | file/string | Original image to edit (file or base64 data URL) |
| `mask` | file | Mask image indicating areas to edit (PNG) |
| `prompt` | string | Description of the edit to make |
| `n` | integer | Number of images to generate |
| `size` | string | Output image size |

### Mask Requirements

- Must be same dimensions as original image
- Transparent areas indicate where to edit
- Opaque areas will be preserved

## Base64 Output

Instead of getting a URL, you can receive base64-encoded image data:

### With xAI SDK

```python
import os
from xai_sdk import Client

client = Client(api_key=os.getenv('XAI_API_KEY'))

response = client.image.sample(
    model="grok-imagine-image",
    prompt="A cat in a tree",
    image_format="base64"
)

print(response.image)  # returns the raw image bytes
```

### With REST API

Set `"response_format": "b64_json"` in the request body. The response will contain a `b64_json` field with the encoded image.

### With gRPC

Set `"format": "IMG_FORMAT_BASE64"` in the request.

## Generating Multiple Images

Generate up to 10 images in one request:

### With xAI SDK

```python
import os
from xai_sdk import Client

client = Client(api_key=os.getenv('XAI_API_KEY'))

response = client.image.sample_batch(
    model="grok-imagine-image",
    prompt="A cat in a tree",
    n=4,
    image_format="url",
)

for image in response:
    print(image.url)
```

### With OpenAI SDK

```python
response = client.images.generate(
    model="grok-imagine-image",
    prompt="A cat in a tree",
    n=4
)

for img in response.data:
    print(img.url)
```

## Setting Aspect Ratio

Instead of fixed sizes, you can specify an aspect ratio:

```python
import os
from xai_sdk import Client

client = Client(api_key=os.getenv('XAI_API_KEY'))

response = client.image.sample(
    model="grok-imagine-image",
    prompt="A cat in a tree",
    aspect_ratio="4:3"
)

print(response.url)
```

### Supported Aspect Ratios

- `1:1` - Square
- `4:3` - Standard
- `3:4` - Portrait standard
- `16:9` - Widescreen landscape
- `9:16` - Widescreen portrait
- `3:2` - Classic photo landscape
- `2:3` - Classic photo portrait

## Size Options

| Size | Aspect Ratio | Best For |
|------|--------------|----------|
| 1024x1024 | 1:1 | Square images, icons, avatars |
| 1792x1024 | ~16:9 | Landscape, banners, headers |
| 1024x1792 | ~9:16 | Portrait, mobile wallpapers |

## Style Options

- **vivid**: More dramatic, hyper-real images
- **natural**: More realistic, less stylized

## Best Practices

### Prompt Engineering

**Good prompts:**
- "A photorealistic portrait of an elderly woman with silver hair, warm lighting, professional photography"
- "An oil painting of a stormy sea with a lighthouse, impressionist style, dramatic lighting"

**Less effective prompts:**
- "A woman" (too vague)
- "Make it look good" (not descriptive)

### Tips

1. **Be specific**: Include details about style, lighting, composition
2. **Specify art style**: "oil painting", "digital art", "photorealistic", etc.
3. **Describe lighting**: "soft lighting", "dramatic shadows", "golden hour"
4. **Include composition details**: "close-up", "wide angle", "bird's eye view"
5. **Mention mood/atmosphere**: "serene", "dramatic", "mysterious"

## Rate Limits

Image generation has specific rate limits. Check the [Consumption and Rate Limits](./rate-limits.md) documentation for current limits.

## Error Handling

```python
try:
    response = client.images.generate(
        model="grok-imagine-image",
        prompt="...",
        n=1
    )
except Exception as e:
    if "content_policy" in str(e):
        print("Prompt violates content policy")
    elif "rate_limit" in str(e):
        print("Rate limit exceeded")
    else:
        print(f"Error: {e}")
```

## Streaming Note

Streaming is **NOT** supported for image generation. The response is returned only after the image is fully generated.
