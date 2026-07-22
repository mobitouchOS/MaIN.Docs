# InferPage OpenAI-Compatible API

InferPage exposes a fully OpenAI-compatible HTTP API, allowing you to use any MaIN.NET model with the OpenAI Python/Node.js SDK, curl, or any OpenAI-compatible client. Migrate from OpenAI by changing one URL.

> **Base URL:** `http://localhost:5555/v1` (default)  
> **Drop-in replacement** for `https://api.openai.com/v1`

---

## Table of Contents

- [Quick Start](#quick-start)
- [Endpoints](#endpoints)
  - [List Models](#list-models)
  - [Create Chat Completion](#create-chat-completion)
  - [Create Response (Responses API)](#create-response-responses-api)
- [Tool Calling](#tool-calling)
  - [Custom Function Tools](#custom-function-tools)
  - [OpenAI Hosted Tools](#openai-hosted-tools)
- [Vision Input](#vision-input)
- [Streaming](#streaming)
- [Authentication](#authentication)
- [Configuration](#configuration)
- [Examples](#examples)
- [Error Handling](#error-handling)

---

## Quick Start

### Python (OpenAI SDK)

```python
from openai import OpenAI

# Point to InferPage instead of OpenAI
client = OpenAI(
    base_url="http://localhost:5555/v1",
    api_key="not-needed"  # or your MaIN__ApiKey
)

# Use any MaIN model
response = client.chat.completions.create(
    model="gemma3-4b",  # or "gpt-5.6-sol", "claude-fable-5", etc.
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message.content)
```

### cURL

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3-4b",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
```

### Node.js

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'http://localhost:5555/v1',
  apiKey: 'not-needed',
});

const response = await client.chat.completions.create({
  model: 'gpt-5.6-sol',
  messages: [{ role: 'user', content: 'Hello!' }],
});

console.log(response.choices[0].message.content);
```

---

## Endpoints

### List Models

Returns all available models on this InferPage instance.

```http
GET /v1/models
```

**Response:**
```json
{
  "object": "list",
  "data": [
    {
      "id": "gemma3-4b",
      "object": "model",
      "created": 1784707010,
      "owned_by": "main-inferpage"
    }
  ]
}
```

**Example:**
```bash
curl http://localhost:5555/v1/models
```

---

### Create Chat Completion

OpenAI-compatible chat completions endpoint. Supports all MaIN.NET models, tool calling, vision, and streaming.

```http
POST /v1/chat/completions
```

**Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `model` | string | Yes | Model ID (e.g., `"gemma3-4b"`, `"gpt-5.6-sol"`, `"claude-fable-5"`) |
| `messages` | array | Yes | Array of message objects |
| `stream` | boolean | No | Enable Server-Sent Events streaming (default: `false`) |
| `temperature` | number | No | Sampling temperature (0.0-2.0) |
| `top_p` | number | No | Nucleus sampling parameter |
| `max_tokens` | integer | No | Maximum tokens to generate |
| `stop` | array | No | Stop sequences |
| `tools` | array | No | Array of tool definitions (see [Tool Calling](#tool-calling)) |
| `tool_choice` | string | No | `"auto"`, `"none"`, or specific tool name |
| `response_format` | object | No | Structured output format |

**Message Format:**

```json
{
  "role": "user",  // "user", "assistant", "system", or "tool"
  "content": "Hello!",
  // For tool calls in assistant messages:
  "tool_calls": [
    {
      "id": "call_123",
      "type": "function",
      "function": {
        "name": "get_weather",
        "arguments": "{\"city\":\"London\"}"
      }
    }
  ],
  // For tool result messages:
  "tool_call_id": "call_123"
}
```

**Response (non-streaming):**

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1784708411,
  "model": "gemma3-4b",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you?",
        "tool_calls": null
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 8,
    "total_tokens": 17
  }
}
```

**Finish Reasons:**
- `"stop"` — Model finished naturally
- `"tool_calls"` — Model wants to call tools
- `"length"` — Hit max_tokens limit

---

### Create Response (Responses API)

OpenAI's newer Responses API endpoint, also fully supported.

```http
POST /v1/responses
```

**Request Body:** Similar to `/v1/chat/completions` but uses the Responses format with `input` instead of `messages`.

**Response:** Similar structure with `output` array containing response items.

**Example:**
```bash
curl http://localhost:5555/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.6-sol",
    "input": "What is the capital of France?"
  }'
```

---

## Tool Calling

InferPage supports **two formats** for tool calling: custom function tools and OpenAI hosted tools. Both work out of the box.

### Custom Function Tools

Define your own functions that the model can call. The model returns tool calls, and you execute them.

**Request:**
```json
{
  "model": "gpt-5.6-sol",
  "messages": [
    {"role": "user", "content": "What's the weather in London?"}
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
              "description": "The city name"
            }
          },
          "required": ["location"]
        }
      }
    }
  ]
}
```

**Response (model wants to call tool):**
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
              "arguments": "{\"location\":\"London\"}"
            }
          }
        ]
      },
      "finish_reason": "tool_calls"
    }
  ]
}
```

**Send tool result back:**
```json
{
  "model": "gpt-5.6-sol",
  "messages": [
    {"role": "user", "content": "What's the weather in London?"},
    {
      "role": "assistant",
      "tool_calls": [{
        "id": "call_abc123",
        "type": "function",
        "function": {
          "name": "get_weather",
          "arguments": "{\"location\":\"London\"}"
        }
      }]
    },
    {
      "role": "tool",
      "tool_call_id": "call_abc123",
      "content": "{\"temperature\": 15, \"condition\": \"cloudy\"}"
    }
  ]
}
```

### OpenAI Hosted Tools

InferPage recognizes OpenAI's hosted tool format. These tools execute server-side automatically.

**Supported Hosted Tools:**

| Type | Description | Execution |
|---|---|---|
| `web_search` | Search the web for current information | Server-side, results inline |
| `web_search_preview` | Legacy web search format | Server-side, results inline |
| `file_search` | Search uploaded files (coming soon) | Server-side |
| `code_interpreter` | Execute Python code (coming soon) | Server-side |
| `computer_use_preview` | Computer use capability (coming soon) | Server-side |

**Request (OpenAI format):**
```json
{
  "model": "gpt-5.6-sol",
  "messages": [
    {"role": "user", "content": "What's the latest news about AI?"}
  ],
  "tools": [
    {
      "type": "web_search"
    }
  ]
}
```

**Response (with search results):**
```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Recent AI developments include...\n\n1. **OpenAI announces GPT-5.6**..."
      },
      "finish_reason": "stop"
    }
  ]
}
```

**Note:** Hosted tools return results inline in the `content` field, not as `tool_calls`. This matches OpenAI's behavior.

---

## Vision Input

Send images to vision-capable models.

**Request:**
```json
{
  "model": "gpt-5.6-sol",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What's in this image?"
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "data:image/jpeg;base64,/9j/4AAQ..."
          }
        }
      ]
    }
  ]
}
```

**Supported Models:**
- Cloud: `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`, `claude-fable-5`, `gemini-3.5-flash`, `grok-4.20-*`
- Local: `gemma3-4b`, `gemma3-12b`, `llava-7b`, `llava-1.6-mistral-7b`, `qwen3.5-4b`, `qwen3.5-9b`

---

## Streaming

Enable Server-Sent Events (SSE) streaming for real-time responses.

**Request:**
```json
{
  "model": "gpt-5.6-sol",
  "messages": [{"role": "user", "content": "Tell me a story"}],
  "stream": true
}
```

**Response (SSE stream):**
```
data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1784708411,"model":"gpt-5.6-sol","choices":[{"index":0,"delta":{"role":"assistant","content":"Once"},"finish_reason":null}]}

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1784708411,"model":"gpt-5.6-sol","choices":[{"index":0,"delta":{"content":" upon"},"finish_reason":null}]}

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1784708411,"model":"gpt-5.6-sol","choices":[{"index":0,"delta":{"content":" a time"},"finish_reason":null}]}

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1784708411,"model":"gpt-5.6-sol","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

**Python Example:**
```python
stream = client.chat.completions.create(
    model="gpt-5.6-sol",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

---

## Authentication

InferPage supports optional API key authentication.

**Configuration:**

Set in `appsettings.json`:
```json
{
  "MaIN": {
    "ApiKey": "your-secret-key-here"
  }
}
```

Or via environment variable:
```bash
export MaIN__ApiKey=your-secret-key-here
```

**Usage:**
```python
client = OpenAI(
    base_url="http://localhost:5555/v1",
    api_key="your-secret-key-here"  # Must match MaIN__ApiKey
)
```

**If no API key is configured, authentication is disabled** (open access).

---

## Configuration

### Starting InferPage

**With a local model:**
```bash
dotnet run --project src/MaIN.InferPage --framework net10.0 \
  --backend self \
  --model gemma3-4b \
  --path C:\path\to\models
```

**With an OpenAI model:**
```bash
export OPENAI_API_KEY=sk-...
dotnet run --project src/MaIN.InferPage --framework net10.0 \
  --backend openai \
  --model gpt-5.6-sol
```

**With an Anthropic model:**
```bash
export ANTHROPIC_API_KEY=sk-ant-...
dotnet run --project src/MaIN.InferPage --framework net10.0 \
  --backend anthropic \
  --model claude-fable-5
```

**Custom port:**
```json
// appsettings.json
{
  "Kestrel": {
    "EndPoints": {
      "Http": {
        "Url": "http://localhost:8080"
      }
    }
  }
}
```

### Model Path

For local models, InferPage stores GGUF files in a configurable directory:

```bash
export MaIN_ModelsPath=/path/to/models
```

Default: `./models` relative to the InferPage working directory.

---

## Examples

### Example 1: Basic Chat with Local Model

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3-4b",
    "messages": [
      {"role": "user", "content": "Explain quantum computing in 3 sentences"}
    ],
    "temperature": 0.7,
    "max_tokens": 200
  }'
```

### Example 2: Tool Calling with Custom Function

```python
import json
from openai import OpenAI

client = OpenAI(base_url="http://localhost:5555/v1", api_key="not-needed")

def get_weather(location):
    # Your actual weather API call here
    return json.dumps({"temp": 15, "condition": "cloudy"})

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            },
            "required": ["location"]
        }
    }
}]

messages = [{"role": "user", "content": "Weather in Paris?"}]

# First call: model decides to call tool
response = client.chat.completions.create(
    model="gpt-5.6-sol",
    messages=messages,
    tools=tools
)

# Check if model wants to call tool
if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    messages.append(response.choices[0].message)
    
    # Execute the tool
    args = json.loads(tool_call.function.arguments)
    result = get_weather(args["location"])
    
    # Send result back
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": result
    })
    
    # Get final response
    final = client.chat.completions.create(
        model="gpt-5.6-sol",
        messages=messages
    )
    print(final.choices[0].message.content)
```

### Example 3: Web Search (OpenAI Hosted Tool)

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.6-sol",
    "messages": [
      {"role": "user", "content": "What is the weather in Tokyo today?"}
    ],
    "tools": [
      {"type": "web_search"}
    ]
  }'
```

**Response:**
```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "I found several weather forecast websites that can help you get the most up-to-date weather information for Tokyo today..."
    },
    "finish_reason": "stop"
  }]
}
```

### Example 4: Vision Input

```python
import base64
from openai import OpenAI

client = OpenAI(base_url="http://localhost:5555/v1", api_key="not-needed")

# Read and encode image
with open("photo.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

response = client.chat.completions.create(
    model="gpt-5.6-sol",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image_data}"}}
        ]
    }]
)

print(response.choices[0].message.content)
```

### Example 5: Streaming

```python
stream = client.chat.completions.create(
    model="claude-fable-5",
    messages=[{"role": "user", "content": "Write a haiku about AI"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
print()
```

---

## Error Handling

InferPage returns OpenAI-compatible error responses.

**Error Format:**
```json
{
  "error": {
    "message": "The model 'invalid-model' does not match the model this InferPage instance is running.",
    "type": "invalid_request_error",
    "code": "model_not_found"
  }
}
```

**HTTP Status Codes:**
- `400` — Invalid request (bad parameters, malformed JSON)
- `401` — Unauthorized (invalid API key)
- `404` — Model not found
- `500` — Server error (model error, backend failure)

**Python Example:**
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:5555/v1", api_key="not-needed")

try:
    response = client.chat.completions.create(
        model="gpt-5.6-sol",
        messages=[{"role": "user", "content": "Hello"}]
    )
except Exception as e:
    print(f"Error: {e}")
```

---

## Compatibility Matrix

| OpenAI Feature | InferPage Support | Notes |
|---|---|---|
| Chat completions | ✅ Full | `/v1/chat/completions` |
| Responses API | ✅ Full | `/v1/responses` |
| List models | ✅ Full | `/v1/models` |
| Function calling | ✅ Full | Custom tools |
| Web search (hosted) | ✅ Full | `web_search`, `web_search_preview` |
| File search (hosted) | 🟡 Detected | Coming soon |
| Code interpreter | 🟡 Detected | Coming soon |
| Vision input | ✅ Full | Image URLs and base64 |
| Streaming (SSE) | ✅ Full | `stream: true` |
| Temperature, top_p | ✅ Full | All sampling params |
| Structured output | ✅ Full | `response_format` with JSON schema |
| Stop sequences | ✅ Full | `stop` parameter |
| System messages | ✅ Full | `role: "system"` |
| Multi-turn | ✅ Full | Full conversation history |

---

## Migration from OpenAI

**Before (OpenAI):**
```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
```

**After (InferPage):**
```python
from openai import OpenAI

# Just change the base_url!
client = OpenAI(
    base_url="http://localhost:5555/v1",
    api_key="not-needed"
)

# Use any model (local or cloud)
response = client.chat.completions.create(
    model="gemma3-4b",  # or "gpt-5.6-sol", "claude-fable-5", etc.
    messages=[{"role": "user", "content": "Hello"}]
)
```

**Zero code changes required** — just swap the `base_url` and `model` parameter!

---

## See Also

- [Supported Models](./models.html) — Complete model catalog
- [ChatContext](./chat.html) — C# fluent API
- [ModelContext](./model-context.html) — Model management
- [Docker Deployment](./docker.html) — Run InferPage in containers
