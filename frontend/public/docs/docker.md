# Docker — MaIN.NET InferPage

MaIN.NET ships a ready-to-run Docker image called **InferPage** — a self-hosted web UI that lets you chat with any MaIN.NET-supported model without writing a line of code. Pull it, point it at a GPU (or don't), and you have a private AI inference endpoint in under a minute.

## What InferPage Is

InferPage is a Blazor Server app that wires up MaIN.NET under the hood. It exposes a clean browser-based chat UI on **port 5555** and supports every backend the framework does — local GGUF models, Ollama, and all cloud providers.

It is **genuinely multiplatform**:

| Platform | Tag | What it uses |
|---|---|---|
| Windows / Linux (NVIDIA GPU) | `:cuda` | CUDA 12.9 runtime, full GPU acceleration |
| Mac (Apple Silicon) | `:cpu` / `:latest` | CPU inference via LLamaSharp |
| Linux (CPU-only) | `:cpu` / `:latest` | CPU inference, no GPU required |
| Any OS (external Ollama) | `:ollama` | Delegates to your running Ollama instance |
| Any OS (Ollama bundled) | `:ollama-bundled` | Ships Ollama inside the image |

## Quick Start

### Mac (Apple Silicon) or Linux CPU

```bash
docker run -d \
  -p 5555:5555 \
  -v ~/models:/app/Models \
  ghcr.io/mobitouchos/main-inferpage:cpu
```

Open `http://localhost:5555` — done.

### Windows / Linux with NVIDIA GPU

```bash
docker run -d \
  --gpus all \
  -p 5555:5555 \
  -v ~/models:/app/Models \
  ghcr.io/mobitouchos/main-inferpage:cuda
```

The `:cuda` image is based on `nvidia/cuda:12.9.1-runtime-ubuntu24.04` and automatically enables all visible GPUs. Make sure the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) is installed.

### External Ollama

If you already have Ollama running locally, skip the model volume entirely:

```bash
docker run -d \
  -p 5555:5555 \
  -e MaIN__OllamaBaseUrl=http://host.docker.internal:11434 \
  ghcr.io/mobitouchos/main-inferpage:ollama
```

### Bundled Ollama (zero dependencies)

```bash
docker run -d \
  -p 5555:5555 \
  -v ollama-data:/root/.ollama \
  ghcr.io/mobitouchos/main-inferpage:ollama-bundled
```

Ollama starts automatically inside the container. Pull models through the UI or via `docker exec`.

## Docker Compose

```yaml
services:
  inferpage:
    image: ghcr.io/mobitouchos/main-inferpage:cpu
    ports:
      - "5555:5555"
    volumes:
      - ./models:/app/Models
    environment:
      ASPNETCORE_ENVIRONMENT: Production
    restart: unless-stopped
```

For the CUDA variant, add:

```yaml
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `MaIN__ModelsPath` | `/app/Models` | Where GGUF model files are stored |
| `MaIN__BackendType` | `0` (Self) | Backend override (7 = Ollama) |
| `MaIN__OllamaBaseUrl` | — | Ollama endpoint for `:ollama` tag |
| `ASPNETCORE_URLS` | `http://+:5555` | Listening address |

## Models Volume

All local (GGUF) models are loaded from `/app/Models`. Mount a host directory there to persist models across container restarts:

```bash
-v /your/models/directory:/app/Models
```

First run with a fresh volume will prompt you to download a model from the UI. Subsequent runs load instantly from the cache.

## Cloud Backends

InferPage isn't limited to local models. Pass cloud keys as environment variables to use any provider:

```bash
docker run -d \
  -p 5555:5555 \
  -e MaIN__BackendType=2 \
  -e MaIN__GeminiKey=your-key \
  ghcr.io/mobitouchos/main-inferpage:cpu
```

Backend type values: `0` = Self (local), `1` = OpenAI, `2` = Gemini, `3` = Anthropic, `4` = GroqCloud, `7` = Ollama.

## Using InferPage as an API

InferPage isn't just a chat UI — every container also exposes an **OpenAI-compatible HTTP API** on the same port (`5555`), right next to the browser UI. Point any existing OpenAI client library at it — the official Python, Node, or .NET SDKs, LangChain, or a plain curl script — and it just works, no code rewrite required. This holds regardless of which backend InferPage is running (local GGUF, Ollama, or a cloud provider): InferPage is the client-facing surface, the configured backend does the actual inference.

| Endpoint | Purpose |
|---|---|
| `POST /v1/chat/completions` | Chat completions — streaming, tool calling, and `response_format` all supported |
| `GET /v1/models` | Lists the model this instance is serving |
| `GET /scalar/v1` | Interactive API explorer — try requests straight from a browser |
| `GET /openapi/v1.json` | Raw OpenAPI 3.1 schema |

### Try it with curl

```bash
curl http://localhost:5555/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{ "messages": [{ "role": "user", "content": "Say hello in one word." }] }'
```

Add `"stream": true` to get the response as Server-Sent Events, chunk by chunk, the same as OpenAI's own streaming API.

### Securing it

By default, no authentication is required — fine for local use. Once the container is reachable from outside your machine, lock it down with an API key:

```bash
docker run -d \
  -p 5555:5555 \
  -e MaIN__ApiKey=your-secret-key \
  ghcr.io/mobitouchos/main-inferpage:cpu
```

Clients then send `Authorization: Bearer your-secret-key`.

### Pointing a real SDK at it

```csharp
using OpenAI;
using OpenAI.Chat;
using System.ClientModel;

var options = new OpenAIClientOptions { Endpoint = new Uri("http://localhost:5555/v1") };
var chatClient = new ChatClient("your-model-name", new ApiKeyCredential("not-needed"), options);

var completion = await chatClient.CompleteChatAsync("Say hello in one word.");
Console.WriteLine(completion.Value.Content[0].Text);
```

Python and Node SDKs work the same way via their `base_url` / `baseURL` option. Tool calling follows the standard OpenAI shape too: InferPage returns `tool_calls` for your app to execute, just like `api.openai.com` — it never runs tools itself.

The quickest way to explore all of this without writing any code: open `http://localhost:5555/scalar/v1` in a browser once the container is running.
