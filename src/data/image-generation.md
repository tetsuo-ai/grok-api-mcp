# Image Generation and Edit

Generate and edit images using Grok's image capabilities.

## Image Generation

### Endpoint

```
POST https://api.x.ai/v1/images/generations
```

### Basic Usage

```python
from openai import OpenAI

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

### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model to use (e.g., "grok-imagine-image") |
| `prompt` | string | Text description of the image to generate |
| `n` | integer | Number of images to generate (1-4) |
| `size` | string | Image size ("1024x1024", "1792x1024", "1024x1792") |
| `quality` | string | Image quality ("standard", "hd") |
| `style` | string | Image style ("vivid", "natural") |
| `response_format` | string | "url" or "b64_json" |

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

### Endpoint

```
POST https://api.x.ai/v1/images/edits
```

### Basic Usage

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
| `image` | file | Original image to edit (PNG, max 4MB) |
| `mask` | file | Mask image indicating areas to edit (PNG) |
| `prompt` | string | Description of the edit to make |
| `n` | integer | Number of images to generate |
| `size` | string | Output image size |

### Mask Requirements

- Must be same dimensions as original image
- Transparent areas indicate where to edit
- Opaque areas will be preserved

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

## Size Options

| Size | Aspect Ratio | Best For |
|------|--------------|----------|
| 1024x1024 | 1:1 | Square images, icons, avatars |
| 1792x1024 | 16:9 | Landscape, banners, headers |
| 1024x1792 | 9:16 | Portrait, mobile wallpapers |

## Quality Options

- **standard**: Faster generation, good for drafts
- **hd**: Higher detail, better for final outputs

## Style Options

- **vivid**: More dramatic, hyper-real images
- **natural**: More realistic, less stylized

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
