# Video Generation and Edit

Generate and edit videos using the Grok Imagine API - xAI's unified video-audio generative model.

## Overview

The Grok Imagine API is designed for end-to-end creative workflows with:
- **Native audio-video generation** - Synchronized audio without separate tools
- **10-second videos at 720p** - High quality output
- **Image-to-video** - Bring static images to life
- **Text-to-video** - Generate from text prompts
- **Video editing** - Restyle scenes, add/remove objects, control motion
- **Best-in-class instruction following**

All video generation/edit requests are **deferred requests**:
1. Send a video generation/edit request
2. Receive a response with a request ID
3. Retrieve the video result later using the request ID

If you're using the SDK, it can handle the polling automatically.

## Pricing

| Feature | Cost |
|---------|------|
| Video generation (with audio) | $4.20/minute |

## Video Generation

### Endpoint

```
POST https://api.x.ai/v1/videos/generations
```

### Basic Usage

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("XAI_API_KEY"),
    base_url="https://api.x.ai/v1"
)

# Start generation
response = client.videos.generate(
    model="grok-video",
    prompt="A rocket launching into space with dramatic lighting"
)

request_id = response.id
print(f"Request ID: {request_id}")

# Poll for result
import time
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

### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Video model (`grok-imagine-video` or `grok-video`) |
| `prompt` | string | Text description of the video |
| `duration` | number | Video duration in seconds (1-15 seconds) |
| `aspect_ratio` | string | Video aspect ratio (default: `16:9`) |

### Response (Initial)

```json
{
  "id": "video_gen_abc123",
  "status": "processing",
  "created": 1699000000
}
```

### Response (Completed)

```json
{
  "id": "video_gen_abc123",
  "status": "completed",
  "created": 1699000000,
  "data": [
    {
      "url": "https://..."
    }
  ]
}
```

## Video Editing

### Endpoint

```
POST https://api.x.ai/v1/videos/edits
```

### Basic Usage

```python
response = client.videos.edit(
    model="grok-video",
    video="https://example.com/video.mp4",
    prompt="Add slow motion effect to the action scene"
)
```

### Input Requirements

- The input video URL must be a **direct, publicly accessible link** to the video file
- Maximum supported video length: **8.7 seconds**

### Edit Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `video` | string | URL to the source video |
| `prompt` | string | Description of the edit |
| `mask_video` | string | Optional mask video URL |

## Async Handling with SDK

The SDK can automatically handle polling:

```python
# SDK handles polling automatically
response = client.videos.generate(
    model="grok-video",
    prompt="A timelapse of clouds moving over mountains",
    wait_for_completion=True  # SDK polls until complete
)

video_url = response.data[0].url
```

## Manual Polling

```python
import time

def generate_video_with_polling(client, prompt, max_wait=300):
    # Start generation
    response = client.videos.generate(
        model="grok-video",
        prompt=prompt
    )

    request_id = response.id
    start_time = time.time()

    # Poll for result
    while time.time() - start_time < max_wait:
        result = client.videos.retrieve(request_id)

        if result.status == "completed":
            return result.data[0].url
        elif result.status == "failed":
            raise Exception(f"Video generation failed: {result.error}")

        time.sleep(5)

    raise Exception("Video generation timed out")
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

- Maximum generated video duration: 15 seconds
- Maximum input video length for editing: 8.7 seconds
- Edited videos have same duration as original (no user-defined duration)
- Video URLs must be direct, publicly accessible links (not embedded players)
- Streaming is NOT supported for video generation
- Processing times vary based on complexity

## Error Handling

```python
try:
    response = client.videos.generate(
        model="grok-video",
        prompt="..."
    )
except Exception as e:
    if "invalid_url" in str(e):
        print("Invalid video URL format")
    elif "video_too_long" in str(e):
        print("Video exceeds maximum length")
    elif "rate_limit" in str(e):
        print("Rate limit exceeded")
    else:
        print(f"Error: {e}")
```

## Status Values

| Status | Description |
|--------|-------------|
| `pending` | Request queued |
| `processing` | Video being generated |
| `completed` | Video ready |
| `failed` | Generation failed |
