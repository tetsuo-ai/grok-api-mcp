# Grok Voice Agent API

Build powerful real-time voice applications with the Grok Voice Agent API. Create interactive voice conversations with Grok models via WebSocket.

## Overview

The Grok Voice Agent API enables:
- Voice assistants
- Phone agents
- Interactive voice applications
- Customer support automation
- Real-time voice conversations
- AI-powered phone systems

Optimized for enterprise use cases across Customer Support, Medical, Legal, Finance, Insurance, and more.

## Key Features

### Real-time Performance
- Optimized for minimal response times
- Natural back-and-forth dialogue without awkward pauses
- Streams audio bidirectionally over WebSocket
- Instant voice-to-voice interactions that feel like talking to a human

### Multilingual Support
- Speaks over **100 languages** with native-quality accents
- Automatically detects input language
- Responds naturally in the same language
- No configuration required for language switching

### Supported Languages
English, Spanish, French, German, Italian, Portuguese, Dutch, Russian, Chinese (Mandarin), Japanese, Korean, Arabic, Hindi, Turkish, Polish, Swedish, Danish, Norwegian, Finnish, Czech, and many more.

## Audio Formats

### Input Formats
- PCM (Linear16) with configurable sample rates (8kHz–48kHz)
- G.711 μ-law (optimized for telephony)
- G.711 A-law (international telephony standard)

### Output Formats
- PCM (Linear16)
- G.711 μ-law
- G.711 A-law

## Voice Options

Choose from 5 distinct voices, each with unique characteristics suited to different applications.

## Connection

### WebSocket Endpoint

```
wss://api.x.ai/v1/voice/agent
```

### Authentication

Include your API key in the connection:

```javascript
const ws = new WebSocket('wss://api.x.ai/v1/voice/agent', {
  headers: {
    'Authorization': `Bearer ${process.env.XAI_API_KEY}`
  }
});
```

## Basic Usage

### JavaScript/Node.js

```javascript
const WebSocket = require('ws');

const ws = new WebSocket('wss://api.x.ai/v1/voice/agent', {
  headers: {
    'Authorization': `Bearer ${process.env.XAI_API_KEY}`
  }
});

// Configure session
ws.on('open', () => {
  ws.send(JSON.stringify({
    type: 'session.configure',
    config: {
      model: 'grok-4',
      voice: 'default',
      input_audio_format: 'pcm16',
      output_audio_format: 'pcm16',
      sample_rate: 16000
    }
  }));
});

// Handle incoming audio
ws.on('message', (data) => {
  const message = JSON.parse(data);

  if (message.type === 'audio.delta') {
    // Play audio chunk
    playAudio(message.audio);
  } else if (message.type === 'transcript') {
    console.log('Transcript:', message.text);
  }
});

// Send audio
function sendAudio(audioChunk) {
  ws.send(JSON.stringify({
    type: 'audio.append',
    audio: audioChunk.toString('base64')
  }));
}
```

### Python

```python
import asyncio
import websockets
import json
import base64

async def voice_chat():
    uri = "wss://api.x.ai/v1/voice/agent"
    headers = {"Authorization": f"Bearer {os.environ['XAI_API_KEY']}"}

    async with websockets.connect(uri, extra_headers=headers) as ws:
        # Configure session
        await ws.send(json.dumps({
            "type": "session.configure",
            "config": {
                "model": "grok-4",
                "voice": "default",
                "input_audio_format": "pcm16",
                "output_audio_format": "pcm16",
                "sample_rate": 16000
            }
        }))

        # Handle messages
        async for message in ws:
            data = json.loads(message)
            if data["type"] == "audio.delta":
                audio_bytes = base64.b64decode(data["audio"])
                # Process audio...
            elif data["type"] == "transcript":
                print(f"Transcript: {data['text']}")

asyncio.run(voice_chat())
```

## Message Types

### Client to Server

| Type | Description |
|------|-------------|
| `session.configure` | Configure session settings |
| `audio.append` | Send audio chunk |
| `audio.commit` | Signal end of audio input |
| `text.append` | Send text input |
| `conversation.interrupt` | Interrupt current response |

### Server to Client

| Type | Description |
|------|-------------|
| `session.configured` | Session configuration confirmed |
| `audio.delta` | Audio response chunk |
| `transcript` | Text transcript of speech |
| `tool_call` | Function call request |
| `error` | Error message |

## Tool Calling

The Voice API supports function calling:

```javascript
ws.send(JSON.stringify({
  type: 'session.configure',
  config: {
    model: 'grok-4',
    tools: [
      {
        type: 'function',
        function: {
          name: 'get_weather',
          description: 'Get weather for a location',
          parameters: {
            type: 'object',
            properties: {
              location: { type: 'string' }
            }
          }
        }
      }
    ]
  }
}));

// Handle tool calls
ws.on('message', (data) => {
  const message = JSON.parse(data);
  if (message.type === 'tool_call') {
    const result = executeFunction(message.function.name, message.function.arguments);
    ws.send(JSON.stringify({
      type: 'tool_result',
      tool_call_id: message.id,
      result: JSON.stringify(result)
    }));
  }
});
```

## Telephony Integration

Supported platforms:
- Twilio
- Vonage
- Other SIP providers

Integration supports:
- CRMs
- Calendars
- Ticketing systems
- Databases
- Custom APIs

## Enterprise Use Cases

Optimized for regulated industries:
- Customer Support
- Medical
- Legal
- Finance
- Insurance

## Telephony Integration

### Supported Platforms

- **Twilio** - Full integration support
- **Vonage** - SIP provider support
- **Other SIP providers** - Standard SIP integration

### Twilio Integration Example

```javascript
// Example: AI-powered phone system with Twilio
const twilio = require('twilio');
const WebSocket = require('ws');

// Connect Twilio media stream to Grok Voice API
app.post('/voice/incoming', (req, res) => {
  const twiml = new twilio.twiml.VoiceResponse();
  const connect = twiml.connect();
  connect.stream({
    url: 'wss://your-server.com/grok-voice-bridge'
  });
  res.type('text/xml');
  res.send(twiml.toString());
});
```

### Tool Calling for Business Systems

The Voice API supports tool calling for:
- CRMs (Salesforce, HubSpot)
- Calendars (Google Calendar, Outlook)
- Ticketing systems (Zendesk, Jira)
- Databases
- Custom APIs

## Third-Party Framework Integration

### LiveKit

Build real-time voice agents using LiveKit's open-source framework:
- Native Grok Voice Agent API integration
- WebRTC support
- Scalable infrastructure

### Voximplant

Build real-time voice agents using Voximplant:
- Native Grok Voice Agent API integration
- SIP support
- Global telephony infrastructure

## Important Notes

- Direct WebRTC connections are **not available** currently
- Use a WebRTC server to connect clients to a server that connects to the Grok Voice Agent API
- Dedicated speech-to-text and text-to-speech APIs coming soon

## Code Examples

### React Frontend with Voice Chat

Complete working examples are available for:
- Real-time voice chat in browser
- React frontend implementation
- Python backend
- Node.js backend

### AI-Powered Phone System

Example using Twilio integration for:
- Inbound call handling
- Outbound dialing
- Call transfers
- IVR systems

## Best Practices

1. **Handle interruptions**: Users may interrupt mid-response
2. **Buffer audio**: Smooth playback by buffering incoming audio
3. **Error recovery**: Reconnect on WebSocket disconnection
4. **Latency optimization**: Use regional endpoints for lower latency
5. **Audio quality**: Use appropriate sample rates for your use case
6. **Industry terminology**: The API handles domain-specific vocabulary accurately
