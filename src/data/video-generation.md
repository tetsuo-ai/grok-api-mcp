# Video Generation and Edit

Generate and edit videos using the Grok Imagine API - xAI's unified video-audio generative model.

## Overview

All video generation/edit requests are **deferred requests**:
1. Send a video generation/edit request
2. Receive a response with a request ID
3. Retrieve the video result later using the request ID

If you're using the xAI SDK, it can handle the polling automatically with `client.video.generate()`.

### Key Features

- **Native audio-video generation** - Synchronized audio without separate tools
- **Up to 15-second videos at 720p** - High quality output
- **Image-to-video** - Bring static images to life
- **Text-to-video** - Generate from text prompts
- **Video editing** - Restyle scenes, add/remove objects, control motion

## Pricing

| Feature | Cost |
|---------|------|
| Video generation (with audio) | $4.20/minute |

## Video Generation with Automatic Polling

The xAI SDK can automatically poll for results until the video is ready:

### Text-to-Video

```python
from xai_sdk import Client

client = Client()

response = client.video.generate(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
)
print(f"Video URL: {response.url}")
```

### Image-to-Video

Generate a video from an existing image:

```python
from xai_sdk import Client

client = Client()

response = client.video.generate(
    prompt="Generate a video based on the provided image.",
    model="grok-imagine-video",
    image_url="<url of the image>",
)
print(f"Video URL: {response.url}")
```

### Video Editing

Edit an existing video with a prompt:

```python
from xai_sdk import Client

client = Client()

response = client.video.generate(
    prompt="Make the ball larger.",
    model="grok-imagine-video",
    video_url="<url of the video to edit>",
)
print(f"Video URL: {response.url}")
```

## Manual Polling

If you prefer to control the polling yourself, use `client.video.start()` to initiate generation and `client.video.get()` to retrieve results.

### Start Video Generation

```python
from xai_sdk import Client

client = Client()

response = client.video.start(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
)
print(f"Request ID: {response.request_id}")
```

Response:
```json
{"request_id":"aa87081b-1a29-d8a6-e5bf-5807e3a7a561"}
```

### Image-to-Video (Manual)

```python
from xai_sdk import Client

client = Client()

response = client.video.start(
    prompt="Generate a video based on the provided image.",
    model="grok-imagine-video",
    image_url="<url of the image>",
)
print(f"Request ID: {response.request_id}")
```

### Video Editing (Manual)

```python
from xai_sdk import Client

client = Client()

response = client.video.start(
    prompt="Make the ball in the video larger.",
    model="grok-imagine-video",
    video_url="<url of the previous video>",
)
print(f"Request ID: {response.request_id}")
```

> **Note**: The input video URL must be a direct, publicly accessible link to the video file. The maximum supported video length for editing is **8.7 seconds**.

### Retrieve Video Results

After getting a `request_id`, retrieve the generated video:

```python
response = client.video.get(request_id)
print(f"Video URL: {response.url}")
```

## REST API (OpenAI SDK Compatible)

### Endpoint

```
POST https://api.x.ai/v1/videos/generations
```

### Basic Usage

```python
from openai import OpenAI
import os
import time

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

# Start generation
response = client.videos.generate(
    model="grok-imagine-video",
    prompt="A rocket launching into space with dramatic lighting"
)

request_id = response.id
print(f"Request ID: {request_id}")

# Poll for result
while True:
    result = client.videos.retrieve(request_id)
    if result.status == "completed":
        video_url = result.data[0].url
        print(f"Video URL: {video_url}")
        break
    elif result.status == "failed":
        print(f"Generation failed: {result.error}")
        break
    time.sleep(5)
```

## Video Output Parameters

### Duration

Specify video duration in seconds (1-15 seconds):

```python
from xai_sdk import Client

client = Client()

response = client.video.generate(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
    duration=10
)
print(f"Video URL: {response.url}")
print(f"Duration: {response.duration}")
```

Or with manual polling:

```python
response = client.video.start(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
    duration=10
)
print(f"Request ID: {response.request_id}")
```

> **Note**: Video editing doesn't support user-defined `duration`. The edited video will have the same duration as the original video.

### Aspect Ratio

Specify the aspect ratio of the video. Default is `16:9`.

**Supported aspect ratios:**
- `16:9` - Widescreen landscape (default)
- `9:16` - Widescreen portrait
- `4:3` - Standard landscape
- `3:4` - Standard portrait
- `1:1` - Square
- `3:2` - Classic landscape
- `2:3` - Classic portrait

```python
from xai_sdk import Client

client = Client()

response = client.video.generate(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
    aspect_ratio="4:3"
)
print(f"Video URL: {response.url}")
```

Or with manual polling:

```python
response = client.video.start(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
    aspect_ratio="4:3"
)
print(f"Request ID: {response.request_id}")
```

### Resolution

Select output resolution from supported options:

**Supported resolutions:**
- `720p` - HD quality
- `480p` - Standard quality

```python
from xai_sdk import Client

client = Client()

response = client.video.generate(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
    resolution="720p"
)
print(f"Video URL: {response.url}")
```

Or with manual polling:

```python
response = client.video.start(
    prompt="A cat playing with a ball",
    model="grok-imagine-video",
    resolution="720p"
)
print(f"Request ID: {response.request_id}")
```

## Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Video model (`grok-imagine-video`) |
| `prompt` | string | Text description of the video |
| `duration` | number | Video duration in seconds (1-15) |
| `aspect_ratio` | string | Video aspect ratio (default: `16:9`) |
| `resolution` | string | Output resolution (`720p`, `480p`) |
| `image_url` | string | Source image URL for image-to-video |
| `video_url` | string | Source video URL for editing |

## Response Format

### Initial Response (Manual Polling)

```json
{
  "request_id": "video_gen_abc123"
}
```

### Completed Response

```json
{
  "url": "https://...",
  "duration": 10
}
```

## Best Practices

### Prompt Engineering

**Good prompts:**
- "A golden retriever running through a field of wildflowers, slow motion, cinematic lighting, 4K"
- "Timelapse of a flower blooming, macro photography, soft natural lighting"

**Tips:**
1. Describe the motion/action clearly
2. Specify camera movements if desired ("pan left", "zoom in")
3. Include lighting and mood
4. Mention video style ("cinematic", "documentary", "slow motion")

### Duration Considerations

- Shorter videos generate faster
- Keep prompts focused for best results
- Complex scenes may need more generation time

## Limitations

| Constraint | Value |
|------------|-------|
| Maximum generated video duration | 15 seconds |
| Maximum input video length for editing | 8.7 seconds |
| Edited video duration | Same as original (cannot specify custom duration) |
| Supported resolutions | 720p, 480p |

**Additional notes:**
- Video URLs must be direct, publicly accessible links (not embedded players)
- Streaming is NOT supported for video generation
- Processing times vary based on complexity

## Error Handling

```python
try:
    response = client.video.generate(
        prompt="...",
        model="grok-imagine-video"
    )
except Exception as e:
    if "invalid_url" in str(e):
        print("Invalid video URL format")
    elif "video_too_long" in str(e):
        print("Video exceeds maximum length (8.7s for edits)")
    elif "rate_limit" in str(e):
        print("Rate limit exceeded")
    else:
        print(f"Error: {e}")
```

## Status Values

When using manual polling, check the status field:

| Status | Description |
|--------|-------------|
| `pending` | Request queued |
| `processing` | Video being generated |
| `completed` | Video ready |
| `failed` | Generation failed |
