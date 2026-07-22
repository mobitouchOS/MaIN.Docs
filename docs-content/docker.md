# Docker — MaIN.NET InferPage

MaIN.NET ships a ready-to-run Docker image called **InferPage** (`ghcr.io/mobitouchos/main-inferpage`). It is a Blazor Server web app that wraps MaIN.NET and exposes a browser-based chat UI on **port 5555**. Zero code required — pull, run, chat.

InferPage is genuinely multiplatform. The same repo produces four image tags targeting different hardware:

| Tag | Base image | Ideal for |
|---|---|---|
| `:cuda` | `nvidia/cuda:12.9.1-runtime-ubuntu24.04` | Windows or Linux with an NVIDIA GPU |
| `:cpu` / `:latest` | `mcr.microsoft.com/dotnet/aspnet:10.0` | Mac (Apple Silicon) or CPU-only Linux |
| `:ollama` | `mcr.microsoft.com/dotnet/aspnet:10.0` | Any OS — delegates to an external Ollama instance |
| `:ollama-bundled` | `mcr.microsoft.com/dotnet/aspnet:10.0` | Any OS — Ollama binary bundled inside the image |

## Key runtime details

- **Port**: `5555` (set via `ASPNETCORE_URLS=http://+:5555`)
- **Models volume**: `/app/Models` — mount a host directory here to persist GGUF models across restarts
- **Data Protection volume**: `/app/DataProtection-Keys`
- **Default backend**: `Self` (local LLamaSharp inference). Override with `MaIN__BackendType` env var.

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `MaIN__ModelsPath` | `/app/Models` | Path where GGUF files are stored |
| `MaIN__BackendType` | `0` | Active backend: 0=Self, 1=OpenAi, 2=Gemini, 3=Anthropic, 4=GroqCloud, 7=Ollama |
| `MaIN__OllamaBaseUrl` | `http://host.docker.internal:11434` | Ollama endpoint (`:ollama` tag only) |
| `ASPNETCORE_ENVIRONMENT` | `Production` | ASP.NET Core environment |
| `ASPNETCORE_URLS` | `http://+:5555` | Listening address |
| `NVIDIA_VISIBLE_DEVICES` | `all` | GPU visibility (`:cuda` tag, set automatically) |
| `MaIN__GeminiKey` | — | Gemini API key when using Gemini backend |
| `MaIN__OpenAiKey` | — | OpenAI API key when using OpenAI backend |
| `MaIN__AnthropicKey` | — | Anthropic API key when using Anthropic backend |
| `MaIN__ApiKey` | — | Bearer token required by HTTP clients to call `/v1/*` endpoints. If unset, the API is open. |

## Quick start commands

### Mac (Apple Silicon) or Linux CPU

```bash
docker run -d \
  -p 5555:5555 \
  -v ~/models:/app/Models \
  ghcr.io/mobitouchos/main-inferpage:cpu
```

### Windows / Linux with NVIDIA GPU

Requires the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

```bash
docker run -d \
  --gpus all \
  -p 5555:5555 \
  -v ~/models:/app/Models \
  ghcr.io/mobitouchos/main-inferpage:cuda
```

### External Ollama (Ollama already running on the host)

```bash
docker run -d \
  -p 5555:5555 \
  ghcr.io/mobitouchos/main-inferpage:ollama
```

The `:ollama` image defaults `MaIN__OllamaBaseUrl` to `http://host.docker.internal:11434`. Override this to point at a remote Ollama.

### Bundled Ollama (Ollama inside the container)

```bash
docker run -d \
  -p 5555:5555 \
  -v ollama-data:/root/.ollama \
  ghcr.io/mobitouchos/main-inferpage:ollama-bundled
```

The entrypoint script starts Ollama first, then InferPage. Pull models via `docker exec <id> ollama pull gemma3:4b` or from the UI.

## Docker Compose examples

### CPU / Mac

```yaml
services:
  inferpage:
    image: ghcr.io/mobitouchos/main-inferpage:cpu
    ports:
      - "5555:5555"
    volumes:
      - ./models:/app/Models
    restart: unless-stopped
```

### CUDA (add the deploy block)

```yaml
services:
  inferpage:
    image: ghcr.io/mobitouchos/main-inferpage:cuda
    ports:
      - "5555:5555"
    volumes:
      - ./models:/app/Models
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    restart: unless-stopped
```

### Cloud backend (e.g., Gemini)

```yaml
services:
  inferpage:
    image: ghcr.io/mobitouchos/main-inferpage:cpu
    ports:
      - "5555:5555"
    environment:
      MaIN__BackendType: "2"
      MaIN__GeminiKey: "${GEMINI_KEY}"
    restart: unless-stopped
```

## Choosing the right tag

- **Windows + NVIDIA GPU** → `:cuda`. Provides the best local inference performance.
- **Mac (M1/M2/M3/M4)** → `:cpu`. LLamaSharp uses Apple's Accelerate framework natively.
- **Linux (no GPU)** → `:cpu`. Runs on any VPS or CI box.
- **Already running Ollama** → `:ollama`. Lightest image; no model storage needed.
- **Want everything self-contained** → `:ollama-bundled`. One container, batteries included.

## Backend type values (for `MaIN__BackendType`)

| Value | Backend |
|---|---|
| `0` | Self (local LLamaSharp — default) |
| `1` | OpenAI |
| `2` | Gemini |
| `3` | Anthropic |
| `4` | GroqCloud |
| `5` | DeepSeek |
| `6` | xAI |
| `7` | Ollama |
| `8` | Vertex |

## Accessing the UI

Once the container is running, open `http://localhost:5555` in any browser. On first launch with a fresh models volume, InferPage will prompt you to pick and download a model. Subsequent starts load from the cache immediately.

## Using InferPage as an OpenAI-Compatible API

Every InferPage container also serves an **OpenAI-compatible HTTP API** alongside the chat UI, on the same port (`5555`). This means any existing OpenAI SDK, library, or tool (the official `openai` Python/Node/`.NET` clients, LangChain, LlamaIndex, curl scripts written against `api.openai.com`, etc.) can point at your InferPage container instead and just work — no code changes beyond swapping the base URL.

This works identically no matter which backend InferPage is configured for (`MaIN__BackendType`) — a local GGUF model, Ollama, or a cloud provider like OpenAI/Gemini/Anthropic. InferPage is the client-facing surface; the configured backend does the actual inference behind it.

### Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/models` | Lists the single model this InferPage instance is currently serving |
| `POST` | `/v1/chat/completions` | OpenAI-compatible chat completions — supports streaming, tool calling, and `response_format` |
| `GET` | `/openapi/v1.json` | Machine-readable OpenAPI 3.1 document for the API above |
| `GET` | `/scalar/v1` | Interactive API explorer (Scalar) — browse the schema and fire test requests from a browser, no client code needed |

### Authentication (optional)

By default the API requires no authentication — fine for local development or a container only reachable on a private network. To require a bearer token (recommended once the container is reachable from outside your machine), set `MaIN__ApiKey`:

```bash
docker run -d \
  -p 5555:5555 \
  -v ~/models:/app/Models \
  -e MaIN__ApiKey=your-secret-key \
  ghcr.io/mobitouchos/main-inferpage:cpu
```

Callers then must send `Authorization: Bearer your-secret-key`. If `MaIN__ApiKey` is unset, the header is not required and, if sent, is ignored.

### Quick test with curl

Non-streaming:

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{ "role": "user", "content": "Say hello in one word." }]
  }'
```

Streaming (Server-Sent Events, identical wire format to OpenAI's `stream: true`):

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{ "role": "user", "content": "Count from 1 to 5." }],
    "stream": true
  }'
```

The response is a sequence of `data: {...}` chunks ending in `data: [DONE]`, exactly like `api.openai.com`.

List the active model:

```bash
curl http://localhost:5555/v1/models
```

The `model` field in a chat completions request is optional — if omitted, InferPage uses whichever model it was started with. If provided, it must match that model or the request is rejected with a `model_not_found` error, so existing OpenAI client code that always sends a `model` string still works as long as it's the right one.

### Structured output (`response_format`)

`response_format: { "type": "json_object" }` and `response_format: { "type": "json_schema", "json_schema": { ... } }` are both supported, the same as OpenAI's API:

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{ "role": "user", "content": "Return a JSON object with a \"greeting\" field." }],
    "response_format": { "type": "json_object" }
  }'
```

On backends without native JSON-mode support, InferPage transparently falls back to grammar-constrained generation to enforce the same shape.

### Tool calling

`tools` and `tool_choice` follow the standard OpenAI function-calling shape. InferPage does **not** execute tools itself — like OpenAI's own API, it returns `tool_calls` in the response for your application to execute, then you send the tool's result back as a follow-up message:

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{ "role": "user", "content": "What is the weather in Paris?" }],
    "tools": [{
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Gets the current weather for a city",
        "parameters": {
          "type": "object",
          "properties": { "city": { "type": "string" } },
          "required": ["city"]
        }
      }
    }]
  }'
```

A response with `"finish_reason": "tool_calls"` means the model wants to invoke a tool; run it locally and send the result back in a `role: "tool"` message to continue the conversation, the same pattern used against `api.openai.com`.

#### OpenAI hosted tools (web search)

InferPage also supports OpenAI's hosted tool format. Use `{"type": "web_search"}` (or the legacy `{"type": "web_search_preview"}`) to have InferPage execute the web search server-side and return results inline — no function definition needed, no tool execution on your side:

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{ "role": "user", "content": "What is the weather in Tokyo today?" }],
    "tools": [{ "type": "web_search" }]
  }'
```

The response contains the model's answer with search results baked into the `content` field. This matches OpenAI's behavior — drop-in compatible with any code that uses `web_search_preview` against `api.openai.com`.

#### Best local models for tool calling

When using InferPage with a local GGUF model (`:cpu` or `:cuda` tag), pick a model that has been fine-tuned for function calling. The top three recommendations:

1. **`qwen3.5-4b`** — newest (2026), 4B parameters, 262K context, has thinking mode
2. **`granite-4.1-3b`** — IBM enterprise-grade, BFCL v3 score 60.80
3. **`nemotron-3-nano-4b`** — NVIDIA edge-optimized, hybrid Mamba-Transformer

MaIN.NET automatically detects each model's native tool-call format (Hermes/Qwen2.5, IBM Granite, Llama 3.1+, Mistral v0.3+, Phi-3.5) and uses the correct prompt template and parser — no configuration needed.

### Using an official OpenAI SDK client

Because the wire format matches OpenAI's API, you can point the real SDK at InferPage instead of writing raw HTTP calls. For example, in .NET:

```csharp
using OpenAI;
using OpenAI.Chat;
using System.ClientModel;

var options = new OpenAIClientOptions { Endpoint = new Uri("http://localhost:5555/v1") };
var chatClient = new ChatClient(model: "your-model-name", new ApiKeyCredential("not-needed"), options);

var completion = await chatClient.CompleteChatAsync("Say hello in one word.");
Console.WriteLine(completion.Value.Content[0].Text);
```

The same `Endpoint` override pattern applies to the official Python and Node SDKs (`base_url` / `baseURL`), and to any other OpenAI-compatible tooling.

### Exploring the API interactively

Rather than crafting curl commands by hand, open `http://localhost:5555/scalar/v1` in a browser once the container is running. It renders the full request/response schema and lets you send real test requests (including streaming and tool calling) directly from the page — useful for a first look at what the API expects without writing any client code.
