# Supported Models

MaIN.NET supports 100+ models across local GGUF, cloud APIs, and hosted backends. Every model is exposed as a compile-time constant under the `Models.*` namespaces so you never need to memorise string IDs.

> **Quick start:** `AIHub.Chat().WithModel(Models.Local.Gemma3_4b).WithMessage("...")` works out of the box — the model file is downloaded on first use.

---

## Table of Contents

- [Local GGUF Models (Self Backend)](#local-gguf-models-self-backend)
  - [Tool-Calling Optimized](#tool-calling-optimized)
  - [Reasoning Models](#reasoning-models)
  - [Vision-Language Models](#vision-language-models)
  - [Code Models](#code-models)
  - [General Purpose](#general-purpose)
  - [Image Generation](#image-generation)
  - [Embeddings & TTS](#embeddings--tts)
- [OpenAI Cloud Models](#openai-cloud-models)
- [Anthropic Claude Models](#anthropic-claude-models)
- [Google Gemini Models](#google-gemini-models)
- [xAI Grok Models](#xai-grok-models)
- [Groq Cloud Models](#groq-cloud-models)
- [DeepSeek Models](#deepseek-models)
- [Vertex AI Models](#vertex-ai-models)
- [Ollama Cloud Models](#ollama-cloud-models)

---

## Local GGUF Models (Self Backend)

Run entirely on your machine via LLamaSharp. Use `Models.Local.*` constants. First call downloads the model file automatically.

### Tool-Calling Optimized

These models are specifically fine-tuned for function/tool calling and work best with MaIN.NET's `WithTools()` API.

| Constant | Model | Size | GGUF Source | Tool Format | Notes |
|---|---|---|---|---|---|
| `Qwen2_5_7b` | Qwen 2.5 7B Instruct | 4.7GB | [Qwen/Qwen2.5-7B-Instruct-GGUF](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct-GGUF) | Hermes JSON | Best small model for tool calling |
| `Llama3_2_1b` | Llama 3.2 1B Instruct | ~1GB | [bartowski/Llama-3.2-1B-Instruct-GGUF](https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF) | Llama 3 | Smallest with native tool support |
| `Mistral7b` | Mistral 7B Instruct v0.3 | 4.5GB | [bartowski/Mistral-7B-Instruct-v0.3-GGUF](https://huggingface.co/bartowski/Mistral-7B-Instruct-v0.3-GGUF) | Mistral v3 | Function calling support |
| `Qwen3_5_4b` | Qwen 3.5 4B | ~2.7GB | [unsloth/Qwen3.5-4B-GGUF](https://huggingface.co/unsloth/Qwen3.5-4B-GGUF) | Hermes JSON | Latest, multimodal, 262K context |
| `Qwen3_5_9b` | Qwen 3.5 9B | ~6GB | [unsloth/Qwen3.5-9B-GGUF](https://huggingface.co/unsloth/Qwen3.5-9B-GGUF) | Hermes JSON | Latest flagship, 262K context |
| `Nemotron3Nano4b` | NVIDIA Nemotron 3 Nano 4B | ~2.5GB | [nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF) | Hermes JSON | Edge-optimized, Mamba-Transformer |
| `Granite4_1_3b` | IBM Granite 4.1 3B | 2.0GB | [ibm-granite/granite-4.1-3b-GGUF](https://huggingface.co/ibm-granite/granite-4.1-3b-GGUF) | Granite | Enterprise-grade, native tool calling |
| `Phi3_5_3b` | Phi 3.5 Mini Instruct | 2.5GB | [bartowski/Phi-3.5-mini-instruct-GGUF](https://huggingface.co/bartowski/Phi-3.5-mini-instruct-GGUF) | Phi 3.5 | Microsoft, 128K context |
| `Phi4_4b` | Phi 4 4B | ~2.5GB | [bartowski/Phi-4-GGUF](https://huggingface.co) | Phi 3.5 | Latest Phi, 16K context |
| `Hermes3_3b` | Hermes 3 3B | ~2GB | [NousResearch/Hermes-3-Llama-3.1-8B-GGUF](https://huggingface.co/NousResearch) | Hermes JSON | NousResearch fine-tune |
| `Hermes3_8b` | Hermes 3 8B | ~5GB | [NousResearch/Hermes-3-Llama-3.1-8B-GGUF](https://huggingface.co/NousResearch) | Hermes JSON | Enhanced dialogue |

### Reasoning Models

Models with explicit chain-of-thought or "thinking" capabilities.

| Constant | Model | Size | GGUF Source | Notes |
|---|---|---|---|---|
| `Qwen3_8b` | Qwen 3 8B | ~5GB | [Qwen/Qwen3-8B-GGUF](https://huggingface.co/Qwen) | Multilingual, logical reasoning |
| `Qwen3_14b` | Qwen 3 14B | ~9GB | [Qwen/Qwen3-14B-GGUF](https://huggingface.co/Qwen) | Complex reasoning, document analysis |
| `QwQ_7b` | QwQ 7B | ~5GB | [Qwen/QwQ-7B-GGUF](https://huggingface.co/Qwen) | Step-by-step problem solving |
| `DeepSeekR1_8b` | DeepSeek R1 8B | ~5GB | [deepseek-ai/DeepSeek-R1-Distill-Llama-8B-GGUF](https://huggingface.co/deepseek-ai) | Math proofs, scientific reasoning |
| `DeepSeekR1_1_5b` | DeepSeek R1 1.5B | ~1GB | [deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B-GGUF](https://huggingface.co/deepseek-ai) | Basic logic, compact size |

### Vision-Language Models

Models that can process images alongside text. Use `WithMessage(text, imageBytes)`.

| Constant | Model | Size | Notes |
|---|---|---|---|
| `Gemma3_4b` | Gemma 3 4B | 2.3GB | Balanced vision-language |
| `Gemma3_12b` | Gemma 3 12B | ~7GB | Advanced visual reasoning |
| `Gemma3n_e4b` | Gemma 3n E4B | ~3GB | Efficient multimodal |
| `Llava_7b` | LLaVA 7B | ~4GB | Image analysis, OCR |
| `Llava16Mistral_7b` | LLaVA 1.6 Mistral 7B | ~4GB | Enhanced visual Q&A |

### Code Models

Specialized for code generation and understanding.

| Constant | Model | Size | Notes |
|---|---|---|---|
| `Qwen2_5_Coder_3b` | Qwen 2.5 Coder 3B | ~2GB | Python, JavaScript, bug fixing |
| `Qwen2_5_Coder_7b` | Qwen 2.5 Coder 7B | ~4.5GB | Full-stack development, API design |
| `Qwen2_5_Coder_14b` | Qwen 2.5 Coder 14B | ~9GB | System design, architecture |
| `OlympicCoder_7b` | OlympicCoder 7B | ~4.5GB | Algorithms, contest programming |
| `Webgen_4b` | Webgen 4B | ~2.5GB | Web development, code generation |

### General Purpose

Versatile models for a wide range of tasks.

| Constant | Model | Size | Notes |
|---|---|---|---|
| `Gemma2_2b` | Gemma 2 2B | 1.7GB | Ultra-lightweight, basic tasks |
| `Qwen2_5_0_5b` | Qwen 2.5 0.5B | ~0.4GB | Ultra-lightweight text completion |
| `Llama3_2_3b` | Llama 3.2 3B | ~2GB | Chatbots, content creation |
| `Llama3_1_8b` | Llama 3.1 8B | ~5GB | Writing, coding, math, general assistance |
| `LFM2_1_2b` | LFM 2 1.2B | ~0.8GB | Fast inference, resource-constrained |
| `Minicpm4_8b` | MiniCPM 4 8B | ~5GB | Balanced performance and efficiency |
| `Yi_6b` | Yi 6B | ~3.5GB | Bilingual Chinese-English |
| `Smollm2_0_1b` | SmolLM2 0.1B | ~0.1GB | Keyword extraction, demos |
| `Olmo2_7b` | OLMo 2 7B | ~4GB | Research, benchmarking |
| `Bielik2_5_11b` | Bielik 2.5 11B | ~7GB | Polish language model |
| `Mistral3_2_24b` | Mistral 3.2 24B | ~15GB | Advanced reasoning, comprehensive knowledge |

### Image Generation

Text-to-image models. Use with image generation API.

| Constant | Model | Size | Notes |
|---|---|---|---|
| `StableDiffusion1_5` | Stable Diffusion 1.5 (Q8_0) | 1.6GB | Fast local image generation, good for CPUs |
| `Flux1Shnell` | FLUX.1 Schnell (Q4_0) | 6.3GB | High-quality, fast (4 steps) |
| `QwenImage` | Qwen-Image (Q4_0) | ~20GB | Highest quality, requires significant RAM/VRAM |

### Embeddings & TTS

Special-purpose models for vector search and speech synthesis.

| Constant | Model | Type | Size | Notes |
|---|---|---|---|---|
| `Mxbai_Embedding` | mxbai-embed-large v1 | Embedding | 0.2GB | Superior knowledge search recall, 1024-dim |
| `NomicEmbedding` | Nomic Embed v2 | Embedding | ~0.5GB | Embedding model for RAG |
| `Kokoro82m` | Kokoro 82M | TTS | ~0.1GB | Frontier TTS, text-to-audio |

---

## OpenAI Cloud Models

Requires `OPENAI_API_KEY` environment variable. Use `Models.OpenAi.*` constants.

### Frontier Models (GPT-5.6 Series)

| Constant | Model ID | Context | Input $/MTok | Output $/MTok | Notes |
|---|---|---|---|---|---|
| `Gpt5_6Sol` | `gpt-5.6-sol` | 1.05M | $5 | $30 | Frontier model, complex reasoning & coding |
| `Gpt5_6Terra` | `gpt-5.6-terra` | 1.05M | $2.50 | $15 | Balanced intelligence & cost |
| `Gpt5_6Luna` | `gpt-5.6-luna` | 1.05M | $1 | $6 | Cost-optimized, high-volume workloads |

### GPT-5 Series

| Constant | Model ID | Notes |
|---|---|---|
| `Gpt5_5` | `gpt-5.5` | Previous flagship |
| `Gpt5_5Pro` | `gpt-5.5-pro` | Professional tier |
| `Gpt5_4` | `gpt-5.4` | Standard |
| `Gpt5_4Mini` | `gpt-5.4-mini` | Lightweight |
| `Gpt5_4Nano` | `gpt-5.4-nano` | Ultra-lightweight |
| `Gpt5_2` | `gpt-5.2` | Standard tier |
| `Gpt5_1` | `gpt-5.1` | Previous generation |
| `Gpt5` | `gpt-5` | Base model |
| `Gpt5Mini` | `gpt-5-mini` | Mini variant |
| `Gpt5Nano` | `gpt-5-nano` | Nano variant |

### GPT-4 Series

| Constant | Model ID | Notes |
|---|---|---|
| `Gpt4_1` | `gpt-4.1` | Latest GPT-4 |
| `Gpt4_1Mini` | `gpt-4.1-mini` | Mini variant |
| `Gpt4o` | `gpt-4o` | Multimodal |
| `Gpt4oMini` | `gpt-4o-mini` | Lightweight multimodal |

### Reasoning & Specialized

| Constant | Model ID | Notes |
|---|---|---|
| `O3` | `o3` | Advanced reasoning |
| `O3Mini` | `o3-mini` | Reasoning, cost-effective |
| `O4Mini` | `o4-mini` | Latest reasoning |
| `CodexMini` | `codex-mini-latest` | Code generation |
| `DallE3` | `dall-e-3` | Image generation |
| `GptImage1` | `gpt-image-1` | Latest image model |

---

## Anthropic Claude Models

Requires `ANTHROPIC_API_KEY` environment variable. Use `Models.Anthropic.*` constants.

### Latest Generation (5th Gen)

| Constant | Model ID | Context | Notes |
|---|---|---|---|
| `ClaudeFable5` | `claude-fable-5` | 200K | 5th-gen Mythos-level, ambitious long-running work, $10/$50 per MTok |
| `ClaudeMythos5` | `claude-mythos-5` | 200K | Most advanced, vetted cybersecurity/biology research only |
| `ClaudeSonnet5` | `claude-sonnet-5` | 200K | Frontier performance, professional work |
| `ClaudeOpus4_8` | `claude-opus-4-8` | 200K | Previous flagship, strong reasoning |

### 4th Generation

| Constant | Model ID | Notes |
|---|---|---|
| `ClaudeOpus4_7` | `claude-opus-4-7` | Previous flagship |
| `ClaudeSonnet4_6` | `claude-sonnet-4-6` | Advanced Sonnet |
| `ClaudeSonnet4_5` | `claude-sonnet-4-5-20250929` | Enhanced capabilities |
| `ClaudeSonnet4` | `claude-sonnet-4-20250514` | Base Sonnet 4 |
| `ClaudeHaiku4_5` | `claude-haiku-4-5-20251001` | Fast, cost-effective |

---

## Google Gemini Models

Requires `GOOGLE_API_KEY` environment variable. Use `Models.Gemini.*` constants.

| Constant | Model ID | Context | Notes |
|---|---|---|---|
| `Gemini3_5Flash` | `gemini-3.5-flash` | 1M | Latest flagship, agentic tasks |
| `Gemini3_1ProPreview` | `gemini-3.1-pro-preview` | 1M | Most capable Pro model |
| `Gemini3_1FlashLite` | `gemini-3.1-flash-lite` | 1M | Low-latency, cost-efficient |
| `Gemini2_5Pro` | `gemini-2.5-pro` | 1M | Most capable Gemini |
| `Gemini2_5Flash` | `gemini-2.5-flash` | 1M | Fast and efficient |
| `Gemini2_0Flash` | `gemini-2.0-flash` | 1M | Optimized for speed |
| `Gemini3_1FlashImagePreview` | `gemini-3.1-flash-image-preview` | 4K | Image generation |
| `GeminiImagen4_0FastGenerate` | `imagen-4.0-fast-generate-001` | 4K | Fast image generation |
| `GeminiNanoBanana` | `gemini-2.5-flash-image` | 130K | High-speed image generation |

---

## xAI Grok Models

Requires `XAI_API_KEY` environment variable. Use `Models.Xai.*` constants.

| Constant | Model ID | Context | Notes |
|---|---|---|---|
| `Grok4_20Reasoning` | `grok-4.20-reasoning` | 2M | Flagship reasoning model |
| `Grok4_20NonReasoning` | `grok-4.20-non-reasoning` | 2M | Fast, agentic tool use |
| `Grok4_1FastReasoning` | `grok-4-1-fast-reasoning` | 2M | High-performance reasoning |
| `Grok4_1Fast` | `grok-4-1-fast-non-reasoning` | 2M | Multimodal agentic tool calling |
| `Grok3Beta` | `grok-3-beta` | 128K | Previous generation beta |
| `GrokImage` | `grok-2-image` | 4K | Image generation |
| `GrokImagineImage` | `grok-imagine-image` | 4K | Image generation |
| `GrokImagineImagePro` | `grok-imagine-image-pro` | 4K | Pro image generation |

---

## Groq Cloud Models

Requires `GROQ_API_KEY` environment variable. Use `Models.Groq.*` constants. Ultra-fast inference.

| Constant | Model ID | Notes |
|---|---|---|
| `Llama4Scout17b` | `meta-llama/llama-4-scout-17b-16e-instruct` | MoE model, vision support |
| `Llama3_3_70b` | `llama-3.3-70b-versatile` | High-performance multilingual |
| `Qwen3_32b` | `qwen/qwen3-32b` | Alibaba's Qwen 3 32B |
| `Llama3_1_8b` | `llama-3.1-8b-instant` | Fast Llama 3.1 8B |
| `GptOss120b` | `openai/gpt-oss-120b` | Open-source 120B GPT |
| `GptOss20b` | `openai/gpt-oss-20b` | Open-source 20B GPT |
| `Compound` | `groq/compound` | Built-in web search & code execution |
| `CompoundMini` | `groq/compound-mini` | Lightweight compound system |

---

## DeepSeek Models

Requires `DEEPSEEK_API_KEY` environment variable. Use `Models.DeepSeek.*` constants.

| Constant | Model ID | Context | Notes |
|---|---|---|---|
| `Reasoner` | `deepseek-reasoner` | 128K | Reasoning-focused, complex problem solving |
| `Chat` | `deepseek-chat` | 128K | General purpose |

---

## Vertex AI Models

Requires Google Cloud credentials. Use `Models.Vertex.*` constants.

| Constant | Model ID | Notes |
|---|---|---|
| `Gemini2_5Pro` | `google/gemini-2.5-pro` | Gemini Pro via Vertex |
| `Gemini2_5Flash` | `google/gemini-2.5-flash` | Gemini Flash via Vertex |
| `Veo2_0Generate` | `google/veo-2.0-generate-001` | Video generation |
| `Imagen4_0Generate` | `google/imagen-4.0-generate-001` | Image generation |

---

## Ollama Cloud Models

Requires Ollama running locally. Use `Models.Ollama.*` constants.

### Gemma Family
| Constant | Model ID | Notes |
|---|---|---|
| `OllamaGemma3_4b` | `gemma3:4b` | Balanced 4B |
| `OllamaGemma3_12b` | `gemma3:12b` | Mid-range |
| `OllamaGemma3_27b` | `gemma3:27b` | Largest Gemma 3 |
| `OllamaGemma4_E4b` | `gemma4:e4b` | Efficient Gemma 4 |
| `OllamaGemma4_26b` | `gemma4:26b` | Gemma 4 26B |

### Llama Family
| Constant | Model ID | Notes |
|---|---|---|
| `OllamaLlama4Scout` | `llama4:scout` | MoE with multimodal |
| `OllamaLlama4Maverick` | `llama4:maverick` | Larger Llama 4 MoE |

### Qwen Family
| Constant | Model ID | Notes |
|---|---|---|
| `OllamaQwen3_8b` | `qwen3:8b` | Qwen 3 8B |
| `OllamaQwen3_14b` | `qwen3:14b` | Qwen 3 14B |
| `OllamaQwen3_30b` | `qwen3:30b` | Qwen 3 30B MoE |
| `OllamaQwen3_5_9b` | `qwen3.5:9b` | Qwen 3.5 9B multimodal |
| `OllamaQwen3_5_27b` | `qwen3.5:27b` | Qwen 3.5 27B multimodal |
| `OllamaQwen3_6_27b` | `qwen3.6:27b` | Qwen 3.6 27B agentic |
| `OllamaQwen3_6_35b` | `qwen3.6:35b` | Qwen 3.6 35B |
| `OllamaQwen3Coder_8b` | `qwen3-coder:8b` | Code-focused |
| `OllamaQwen3Coder_30b` | `qwen3-coder:30b` | Most capable code model |

### Other
| Constant | Model ID | Notes |
|---|---|---|
| `OllamaDeepSeekR1_7b` | `deepseek-r1:7b` | Reasoning model |
| `OllamaDeepSeekR1_14b` | `deepseek-r1:14b` | Reasoning model |
| `OllamaPhi4_14b` | `phi4:14b` | Microsoft Phi 4 |
| `OllamaMistral_7b` | `mistral:7b` | Mistral 7B base |

---

## Usage Examples

### Basic Usage

```csharp
using MaIN.Core.Hub;
using MaIN.Domain.Models;

// Local model (auto-downloads)
var result = await AIHub.Chat()
    .WithModel(Models.Local.Gemma3_4b)
    .EnsureModelDownloaded()
    .WithMessage("Hello!")
    .CompleteAsync();

// OpenAI cloud
var result = await AIHub.Chat()
    .WithModel(Models.OpenAi.Gpt5_6Sol)
    .WithMessage("Hello!")
    .CompleteAsync();

// Anthropic Claude
var result = await AIHub.Chat()
    .WithModel(Models.Anthropic.ClaudeFable5)
    .WithMessage("Hello!")
    .CompleteAsync();
```

### Tool Calling

```csharp
var tools = new ToolsConfigurationBuilder()
    .AddTool("get_weather", "Get current weather", (string args) => "Sunny, 72°F")
    .WithToolChoice("auto")
    .Build();

var result = await AIHub.Chat()
    .WithModel(Models.Local.Qwen2_5_7b)  // Best for tool calling
    .WithMessage("What's the weather?")
    .WithTools(tools)
    .CompleteAsync();
```

### Vision Input

```csharp
var imageBytes = File.ReadAllBytes("photo.jpg");
var result = await AIHub.Chat()
    .WithModel(Models.Local.Gemma3_4b)  // Supports vision
    .WithMessage("What's in this image?", imageBytes)
    .CompleteAsync();
```

### Vision via Cloud

```csharp
var result = await AIHub.Chat()
    .WithModel(Models.OpenAi.Gpt5_6Sol)  // Multimodal
    .WithMessage("Describe this image", imageBytes)
    .CompleteAsync();
```

---

## Model Selection Guide

**For tool calling / agents:**
- Best: `Qwen2_5_7b`, `Qwen3_5_4b`, `Granite4_1_3b`, `Nemotron3Nano4b`
- Budget: `Llama3_2_1b`, `Qwen2_5_Coder_3b`

**For reasoning:**
- Best: `ClaudeFable5`, `ClaudeMythos5`, `Gpt5_6Sol`
- Budget: `QwQ_7b`, `DeepSeekR1_8b`, `Qwen3_8b`

**For coding:**
- Best: `ClaudeFable5`, `Gpt5_6Sol`, `Qwen2_5_Coder_14b`
- Budget: `Qwen2_5_Coder_3b`, `OlympicCoder_7b`

**For vision:**
- Best: `Gpt5_6Sol`, `ClaudeFable5`, `Gemini3_5Flash`
- Local: `Gemma3_12b`, `Llava16Mistral_7b`

**For cost-sensitive:**
- Cloud: `Gpt5_6Luna` ($1/$6), `Grok4_1Fast`, `Gemini2_5Flash`
- Local: `Qwen2_5_0_5b`, `Gemma2_2b`, `Smollm2_0_1b`

**For long context:**
- Best: `ClaudeFable5` (200K), `Gpt5_6Sol` (1.05M), `Grok4_20Reasoning` (2M)
- Local: `Qwen3_5_4b` (262K), `Gemini3_4b` (128K)

---

## Adding Custom Models

For GGUF models not in the registry, register at runtime:

```csharp
ModelRegistry.RegisterOrReplace(new GenericLocalModel(
    FileName: "my-model-Q4_K_M.gguf",
    Name: "My Custom Model",
    Id: "my-model",
    DownloadUrl: new Uri("https://huggingface.co/.../my-model-Q4_K_M.gguf")
));
```

Then use it:
```csharp
var result = await AIHub.Chat()
    .WithModel("my-model")
    .EnsureModelDownloaded()
    .WithMessage("Hello!")
    .CompleteAsync();
```

For cloud models:
```csharp
ModelRegistry.RegisterOrReplace(new GenericCloudModel(
    Id: "my-custom-model",
    Backend: BackendType.OpenAi,
    Name: "My Custom Model"
));
```

---

## Environment Variables

| Variable | Backend | Description |
|---|---|---|
| `OPENAI_API_KEY` | OpenAI | Required for OpenAI models |
| `ANTHROPIC_API_KEY` | Anthropic | Required for Claude models |
| `GOOGLE_API_KEY` | Gemini | Required for Google AI models |
| `XAI_API_KEY` | xAI | Required for Grok models |
| `GROQ_API_KEY` | Groq | Required for Groq Cloud |
| `DEEPSEEK_API_KEY` | DeepSeek | Required for DeepSeek |
| `MaIN_ModelsPath` | Self | Directory for downloaded GGUF models (default: `./models`) |
| `MaIN__ApiKey` | InferPage | API key for InferPage OpenAI-compatible endpoint |

---

## See Also

- [ChatContext](./chat.html) — Fluent builder for completions
- [ModelContext](./model-context.html) — Model management & downloads
- [AgentContext](./agents.html) — Multi-agent orchestration
- [InferPage OpenAI API](./inferpage-api.html) — HTTP API for all models
