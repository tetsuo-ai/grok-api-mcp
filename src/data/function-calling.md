# Function Calling Guide

Function calling allows Grok models to generate structured outputs that can be used to call external functions or APIs.

## Overview

Function calling enables you to:
- Define custom functions with JSON schemas
- Have Grok decide when and how to call functions
- Build agentic applications that interact with external systems

## Basic Usage

Define tools in your API request:

```json
{
  "model": "grok-4",
  "messages": [
    {"role": "user", "content": "What's the weather in San Francisco?"}
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get the current weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "City and state, e.g. San Francisco, CA"
            }
          },
          "required": ["location"]
        }
      }
    }
  ]
}
```

## Response Format

When the model decides to call a function, it returns:

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "tool_calls": [
          {
            "id": "call_abc123",
            "type": "function",
            "function": {
              "name": "get_weather",
              "arguments": "{\"location\": \"San Francisco, CA\"}"
            }
          }
        ]
      }
    }
  ]
}
```

## Sending Tool Results

After executing the function, send results back:

```json
{
  "model": "grok-4",
  "messages": [
    {"role": "user", "content": "What's the weather in San Francisco?"},
    {"role": "assistant", "tool_calls": [...]},
    {
      "role": "tool",
      "tool_call_id": "call_abc123",
      "content": "{\"temperature\": 65, \"condition\": \"sunny\"}"
    }
  ]
}
```

## Best Practices

1. **Clear descriptions**: Write detailed function descriptions to help the model understand when to use them
2. **Validate inputs**: Always validate function arguments before execution
3. **Handle errors**: Return informative error messages in tool responses
4. **Parallel calls**: Grok can request multiple function calls in a single response - handle them appropriately

## Tool Choice

Control function calling behavior with `tool_choice`:

- `"auto"` - Model decides when to call functions (default)
- `"none"` - Disable function calling
- `{"type": "function", "function": {"name": "function_name"}}` - Force specific function

## Streaming

Function calls work with streaming. Tool call deltas are sent incrementally:

```json
{"choices": [{"delta": {"tool_calls": [{"index": 0, "function": {"arguments": "{\""}}]}}]}
```

## Models Supporting Function Calling

- grok-4 (recommended)
- grok-4-1-fast-reasoning (optimized for agentic use)
- grok-4-1-fast-non-reasoning
- grok-3
