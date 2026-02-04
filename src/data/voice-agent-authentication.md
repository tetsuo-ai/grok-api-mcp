# Voice Agent Authentication

Authenticate WebSocket connections to the Grok Voice Agent API.

## Authentication Methods

You can authenticate WebSocket connections using:
1. **xAI API Key** - For server-side connections
2. **Ephemeral Token** - For client-side connections (recommended for browsers)

## API Key Authentication

Use your xAI API key directly for server-to-server connections:

```javascript
const WebSocket = require('ws');

const ws = new WebSocket('wss://api.x.ai/v1/voice/agent', {
  headers: {
    'Authorization': `Bearer ${process.env.XAI_API_KEY}`
  }
});
```

**Warning**: Never use your API key in client-side code (browsers) as it could be exposed.

## Ephemeral Token Authentication

For client-side applications (browsers, mobile apps), use ephemeral tokens.

### Why Ephemeral Tokens?

- Short-lived (expire quickly)
- Scoped access (limited permissions)
- Safe to use in client-side code
- Can be revoked without affecting your main API key

### Getting an Ephemeral Token

Request a token from your server:

```http
POST https://api.x.ai/v1/realtime/client_secrets
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "ttl": 3600,
  "scopes": ["voice"]
}
```

**Response**:
```json
{
  "token": "ephemeral_token_abc123...",
  "expires_at": "2024-01-20T11:00:00Z"
}
```

### Using Ephemeral Tokens

#### Server-side (Node.js)

```javascript
const express = require('express');
const app = express();

// Endpoint to get ephemeral token for clients
app.post('/api/voice-token', async (req, res) => {
  const response = await fetch('https://api.x.ai/v1/realtime/client_secrets', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.XAI_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      ttl: 3600,
      scopes: ['voice']
    })
  });

  const data = await response.json();
  res.json({ token: data.token });
});
```

#### Client-side (Browser)

```javascript
// Get ephemeral token from your server
const tokenResponse = await fetch('/api/voice-token', { method: 'POST' });
const { token } = await tokenResponse.json();

// Connect to Voice API with ephemeral token
const ws = new WebSocket(`wss://api.x.ai/v1/voice/agent?token=${token}`);
```

### Token Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `ttl` | integer | Time to live in seconds (max 86400) |
| `scopes` | array | Permissions: ["voice"] |

### Token Lifecycle

1. Client requests token from your server
2. Your server requests token from xAI (using API key)
3. xAI returns ephemeral token
4. Your server returns token to client
5. Client uses token to connect to Voice API
6. Token expires after TTL

## Security Best Practices

### Server-Side

1. **Never expose API key**: Keep your API key on the server only
2. **Validate requests**: Verify client requests before issuing tokens
3. **Set appropriate TTL**: Use short TTLs (1 hour or less)
4. **Log token requests**: Track who requests tokens

### Client-Side

1. **Use ephemeral tokens only**: Never embed API keys
2. **Request tokens as needed**: Don't cache tokens long-term
3. **Handle expiration**: Refresh tokens before they expire
4. **Secure storage**: Don't persist tokens in localStorage

## Token Refresh

Implement token refresh before expiration:

```javascript
class VoiceClient {
  constructor() {
    this.token = null;
    this.tokenExpiry = null;
    this.ws = null;
  }

  async ensureToken() {
    // Refresh if token expires in less than 5 minutes
    if (!this.token || Date.now() > this.tokenExpiry - 300000) {
      await this.refreshToken();
    }
    return this.token;
  }

  async refreshToken() {
    const response = await fetch('/api/voice-token', { method: 'POST' });
    const data = await response.json();
    this.token = data.token;
    this.tokenExpiry = Date.parse(data.expires_at);
  }

  async connect() {
    const token = await this.ensureToken();
    this.ws = new WebSocket(`wss://api.x.ai/v1/voice/agent?token=${token}`);
  }
}
```

## Error Handling

### Invalid Token

```json
{
  "error": {
    "code": "invalid_token",
    "message": "The provided token is invalid or expired"
  }
}
```

**Solution**: Request a new ephemeral token.

### Insufficient Scope

```json
{
  "error": {
    "code": "insufficient_scope",
    "message": "Token does not have required scopes"
  }
}
```

**Solution**: Request token with correct scopes.

## Scoped Resources

The ephemeral token gives the holder scoped access to resources. Current scopes:

| Scope | Access |
|-------|--------|
| `voice` | Voice Agent API |

More scopes may be added in the future.
