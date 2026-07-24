# Models Reference

All supported models are available as compile-time constants in the `Models` namespace. They can also be passed as plain strings (e.g. `.WithModel("gemma3-4b")`). The **API ID** in each table is a copyable value: pass it unchanged to InferPage's `model` setting to populate `Utils.Model`.

For direct InferPage configuration, set the lowercase `model` environment variable to the API ID. The supplied Docker Compose file maps its `MODEL_NAME` variable to this same `model` setting and maps `BACKEND` to `backend`:

```bash
model=meta-llama/llama-4-scout-17b-16e-instruct backend=groqcloud GROQ_API_KEY=... dotnet run --project src/MaIN.InferPage
```

## Local (Self backend — GGUF via LLamaSharp)

`BackendType.Self` runs GGUF files locally using LLamaSharp, which is **built into MaIN.NET** — no extra NuGet package needed. Set `ModelsPath` to the directory containing your `.gguf` files.

### Tool-calling optimized models

These models are specifically fine-tuned for function/tool calling and work best with MaIN.NET's `WithTools()` API.

**Recommended (in order):**

| Constant | API ID | Model | GGUF Source | Why |
|---|---|---|---|---|
| `Models.Local.Qwen3_5_4b` | `qwen3.5-4b` | **Qwen 3.5 4B (RECOMMENDED)** — newest model (2026), specifically designed for tool calling and agentic AI, trained on tool calling datasets, native tool support, 4B parameters (fast), 262K context, has thinking mode | [unsloth/Qwen3.5-4B-GGUF](https://huggingface.co/unsloth/Qwen3.5-4B-GGUF) | Best overall for tool calling |
| `Models.Local.Granite4_1_3b` | `granite-4.1-3b` | **IBM Granite 4.1 3B (RECOMMENDED)** — enterprise-grade, specifically fine-tuned for function calling, BFCL v3 score 60.80, trained on xLAM and glaive tool calling datasets | [ibm-granite/granite-4.1-3b-GGUF](https://huggingface.co/ibm-granite/granite-4.1-3b-GGUF) | Best for enterprise/production |
| `Models.Local.Nemotron3Nano4b` | `nemotron-3-nano-4b` | **NVIDIA Nemotron 3 Nano 4B (RECOMMENDED)** — edge-optimized, hybrid Mamba-Transformer, trained on ToolBench and APIGen data | [nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF) | Best for edge/constrained environments |
| `Models.Local.Qwen3_5_9b` | `qwen3.5-9b` | Qwen 3.5 9B — latest flagship, multimodal | [unsloth/Qwen3.5-9B-GGUF](https://huggingface.co/unsloth/Qwen3.5-9B-GGUF) | More powerful Qwen 3.5 option |
| `Models.Local.Qwen2_5_7b` | `qwen2.5-7b` | Qwen 2.5 7B Instruct — proven tool calling | [Qwen/Qwen2.5-7B-Instruct-GGUF](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct-GGUF) | Previous generation |
| `Models.Local.Llama3_2_1b` | `llama3.2-1b` | Llama 3.2 1B Instruct — smallest with native tool support | [bartowski/Llama-3.2-1B-Instruct-GGUF](https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF) | Smallest with tool calling |
| `Models.Local.Mistral7b` | `mistral-7b` | Mistral 7B Instruct v0.3 — function calling | [bartowski/Mistral-7B-Instruct-v0.3-GGUF](https://huggingface.co/bartowski/Mistral-7B-Instruct-v0.3-GGUF) | Mistral function calling |
| `Models.Local.Phi3_5_3b` | `phi3.5-3b` | Phi 3.5 Mini Instruct — Microsoft, 128K context | [bartowski/Phi-3.5-mini-instruct-GGUF](https://huggingface.co/bartowski/Phi-3.5-mini-instruct-GGUF) | Microsoft option |
| `Models.Local.Phi4_4b` | `phi4-4b` | Phi 4 4B — latest Phi | — | Latest Phi |
| `Models.Local.Hermes3_3b` | `hermes3-3b` | Hermes 3 3B — NousResearch fine-tune | — | Hermes format |
| `Models.Local.Hermes3_8b` | `hermes3-8b` | Hermes 3 8B — enhanced dialogue | — | Hermes format |

### Modern 20B-32B models

These newer local models are also available through InferPage. The **API ID** column is the exact value returned by `GET /v1/models` and accepted by `/v1/chat/completions`.

| Constant | API ID | Model | Context | GGUF Source | Notes |
|---|---|---|---:|---|---|
| `Models.Local.Qwen3_30b_A3B_Instruct2507` | `qwen3-30b-a3b-instruct-2507` | Qwen 3 30B A3B Instruct 2507 | 256K | [unsloth/Qwen3-30B-A3B-Instruct-2507-GGUF](https://huggingface.co/unsloth/Qwen3-30B-A3B-Instruct-2507-GGUF) | Sparse MoE with 3.3B active parameters and tool use |
| `Models.Local.Qwen3_Coder_30b_A3B` | `qwen3-coder-30b-a3b` | Qwen 3 Coder 30B A3B | 256K | [unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF](https://huggingface.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF) | Agentic coding and repository-scale context |
| `Models.Local.MagistralSmall_24b` | `magistral-small-24b` | Magistral Small 24B | 40K | [unsloth/Magistral-Small-2506-GGUF](https://huggingface.co/unsloth/Magistral-Small-2506-GGUF) | Local multilingual reasoning |
| `Models.Local.GptOss_20b` | `gpt-oss-20b` | OpenAI gpt-oss 20B | 128K | [unsloth/gpt-oss-20b-GGUF](https://huggingface.co/unsloth/gpt-oss-20b-GGUF) | Open-weight MoE with reasoning and tool use |
| `Models.Local.DevstralSmall2_24b` | `devstral-small-2-24b` | Devstral Small 2 24B | 128K | [bartowski/mistralai_Devstral-Small-2-24B-Instruct-2512-GGUF](https://huggingface.co/bartowski/mistralai_Devstral-Small-2-24B-Instruct-2512-GGUF) | Software engineering and agentic coding |
| `Models.Local.Exaone4_32b` | `exaone-4-32b` | EXAONE 4.0 32B | 128K | [LGAI-EXAONE/EXAONE-4.0-32B-GGUF](https://huggingface.co/LGAI-EXAONE/EXAONE-4.0-32B-GGUF) | English, Korean, and Spanish reasoning |
| `Models.Local.Nemotron3Nano_30b_A3B` | `nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B A3B | 256K | [unsloth/Nemotron-3-Nano-30B-A3B-GGUF](https://huggingface.co/unsloth/Nemotron-3-Nano-30B-A3B-GGUF) | Hybrid Mamba-Transformer MoE with tool use |
| `Models.Local.Qwen3_VL_32b_Instruct` | `qwen3-vl-32b-instruct` | Qwen 3 VL 32B Instruct | 256K | [unsloth/Qwen3-VL-32B-Instruct-GGUF](https://huggingface.co/unsloth/Qwen3-VL-32B-Instruct-GGUF) | Vision, OCR, video, and visual agents; also downloads `mmproj-F16.gguf` |

### Reasoning models

| Constant | API ID | Model |
|---|---|---|
| `Models.Local.Qwen3_8b` | `qwen3-8b` | Qwen 3 8B — multilingual, logical reasoning |
| `Models.Local.Qwen3_14b` | `qwen3-14b` | Qwen 3 14B — complex reasoning, document analysis |
| `Models.Local.QwQ_7b` | `qwq-7b` | QwQ 7B — step-by-step problem solving |
| `Models.Local.DeepSeekR1_8b` | `deepseekr1-8b` | DeepSeek R1 8B — math proofs, scientific reasoning |
| `Models.Local.DeepSeekR1_1_5b` | `deepseekr1-1.5b` | DeepSeek R1 1.5B — basic logic, compact size |

### General purpose models

| Constant | API ID | Model |
|---|---|---|
| `Models.Local.Gemma2_2b` | `gemma2-2b` | Google Gemma 2 2B — fast, small, general purpose |
| `Models.Local.Gemma3_4b` | `gemma3-4b` | Google Gemma 3 4B — stronger reasoning, vision capable |
| `Models.Local.Gemma3_12b` | `gemma3-12b` | Google Gemma 3 12B — advanced visual reasoning |
| `Models.Local.Gemma3n_e4b` | `gemma3n-e4b` | Gemma 3n E4B — efficient multimodal |
| `Models.Local.Llama3_2_3b` | `llama3.2-3b` | Meta Llama 3.2 3B — compact instruction model |
| `Models.Local.Llama3_1_8b` | `llama3.1-8b` | Meta Llama 3.1 8B — stronger reasoning, larger context |
| `Models.Local.Qwen2_5_0_5b` | `qwen2.5-0.5b` | Qwen 2.5 0.5B — ultra-small, edge deployment |
| `Models.Local.Qwen2_5_Coder_3b` | `qwen2.5-coder-3b` | Qwen 2.5 Coder 3B — Python, JavaScript, bug fixing |
| `Models.Local.Qwen2_5_Coder_7b` | `qwen2.5-coder-7b` | Qwen 2.5 Coder 7B — full-stack development, API design |
| `Models.Local.Qwen2_5_Coder_14b` | `qwen2.5-coder-14b` | Qwen 2.5 Coder 14B — system design, architecture |
| `Models.Local.Llava_7b` | `llava-7b` | LLaVA 7B — vision model (image input) |
| `Models.Local.Llava16Mistral_7b` | `llava-1.6-mistral-7b` | LLaVA 1.6 Mistral 7B — enhanced vision |
| `Models.Local.OlympicCoder_7b` | `olympiccoder-7b` | OlympicCoder 7B — algorithms, contest programming |
| `Models.Local.Webgen_4b` | `webgen-4b` | Webgen 4B — web development, code generation |
| `Models.Local.LFM2_1_2b` | `lfm2-1.2b` | LFM 2 1.2B — fast inference, resource-constrained |
| `Models.Local.Minicpm4_8b` | `minicpm4-8b` | MiniCPM 4 8B — balanced performance |
| `Models.Local.Mistral3_2_24b` | `mistral-3.2-24b` | Mistral 3.2 24B — advanced reasoning, large model |
| `Models.Local.Yi_6b` | `yi-6b` | Yi 6B — bilingual Chinese-English |
| `Models.Local.Smollm2_0_1b` | `smollm2-0.1b` | SmolLM2 0.1B — keyword extraction, demos |
| `Models.Local.Olmo2_7b` | `olmo2-7b` | OLMo 2 7B — research, benchmarking |
| `Models.Local.Bielik2_5_11b` | `bielik-2.5-11b` | Bielik 2.5 11B — Polish language model |
| `Models.Local.DeepSeekR1_1_5b` | `deepseekr1-1.5b` | DeepSeek-R1 1.5B — reasoning/CoT model |

### Local image generation (diffusion models)

| Constant | API ID | Model |
|---|---|---|
| `Models.Local.StableDiffusion1_5` | `stable-diffusion-1.5` | Stable Diffusion 1.5 — local image generation (512×512, CPU-friendly) |
| `Models.Local.Flux1Shnell` | `FLUX.1_Shnell` | FLUX.1 Schnell — local image generation (768×768, balanced quality) |
| `Models.Local.QwenImage` | `qwen-image` | Qwen Image — local image generation (1024×1024, highest quality) |

### Local TTS and embeddings

| Constant | API ID | Model | Type |
|---|---|---|---|
| `Models.Local.Kokoro82m` | `kokoro-82m` | Kokoro 82M | Text-to-speech |
| `Models.Local.Mxbai_Embedding` | `mxbai-embedding` | mxbai-embed-large v1 | Embeddings (1024-dim) |
| `Models.Local.NomicEmbedding` | `nomic-embedding` | Nomic Embed v2 | Embeddings |

**Adding a custom GGUF model** (not in the built-in catalogue):

```csharp
ModelRegistry.Register(new GenericLocalModel(
    fileName: "Qwen2.5-7B-Instruct-Q4_K_M.gguf",
    name: "Qwen 2.5 7B",
    id: "qwen2.5-7b",
    SystemMessage: null
));

await AIHub.Chat()
    .WithModel("qwen2.5-7b")
    .EnsureModelDownloaded()
    .WithMessage("Hello!")
    .CompleteAsync(interactive: true);
```

**Key facts about Self backend:**
- LLamaSharp is already a dependency of `MaIN.NET` — do NOT add it separately
- Models are downloaded to `ModelsPath` (config key) or the `MaIN_ModelsPath` env variable
- `EnsureModelDownloaded()` auto-downloads before first use; `EnsureDownloadedAsync` supports progress reporting
- `LocalInferenceParams` controls temperature, grammar, top-p, max tokens for local text models
- Context size is set via `WithMemoryParams(new MemoryParams { ContextSize = 4096 })`
- **Tool calling support:** MaIN.NET automatically detects the model's native tool-call format (Hermes/Qwen2.5, IBM Granite, Llama 3.1+, Mistral v0.3+, Phi-3.5) and uses the correct prompt template and parser

**Local image generation (diffusion models):**
- `StableDiffusion1_5`, `Flux1Shnell`, and `QwenImage` run in-process via `StableDiffusion.NET` — no external service or `ImageGenUrl` needed
- Register the model class first: `ModelRegistry.RegisterOrReplace(new Flux1Shnell())`
- `EnsureDownloadedAsync` handles all required assets (VAE, CLIP, T5-XXL) automatically
- See [image-generation.md](image-generation.md) for full examples

---

## OpenAI

`BackendType.OpenAi` — set `OpenAiKey` in config.

### Frontier models (GPT-5.6 series)

| Constant | API ID | Notes |
|---|---|---|
| `Models.OpenAi.Gpt5_6Sol` | `gpt-5.6-sol` | Frontier model, complex reasoning & coding |
| `Models.OpenAi.Gpt5_6Terra` | `gpt-5.6-terra` | Balanced intelligence & cost |
| `Models.OpenAi.Gpt5_6Luna` | `gpt-5.6-luna` | Cost-optimized, high-volume workloads |

### GPT-5 series

| Constant | API ID |
|---|---|
| `Models.OpenAi.Gpt5_5` | `gpt-5.5` |
| `Models.OpenAi.Gpt5_5Pro` | `gpt-5.5-pro` |
| `Models.OpenAi.Gpt5_4` | `gpt-5.4` |
| `Models.OpenAi.Gpt5_4Mini` | `gpt-5.4-mini` |
| `Models.OpenAi.Gpt5_4Nano` | `gpt-5.4-nano` |
| `Models.OpenAi.Gpt5_2` | `gpt-5.2` |
| `Models.OpenAi.Gpt5_1` | `gpt-5.1` |
| `Models.OpenAi.Gpt5` | `gpt-5` |
| `Models.OpenAi.Gpt5Mini` | `gpt-5-mini` |
| `Models.OpenAi.Gpt5Nano` | `gpt-5-nano` |

### GPT-4 series

| Constant | API ID |
|---|---|
| `Models.OpenAi.Gpt4_1` | `gpt-4.1` |
| `Models.OpenAi.Gpt4_1Mini` | `gpt-4.1-mini` |
| `Models.OpenAi.Gpt4o` | `gpt-4o` |
| `Models.OpenAi.Gpt4oMini` | `gpt-4o-mini` |

### Reasoning & specialized

| Constant | API ID |
|---|---|
| `Models.OpenAi.O3` | `o3` (advanced reasoning) |
| `Models.OpenAi.O3Mini` | `o3-mini` |
| `Models.OpenAi.O4Mini` | `o4-mini` (latest reasoning) |
| `Models.OpenAi.CodexMini` | `codex-mini-latest` |
| `Models.OpenAi.DallE3` | `dall-e-3` (image generation) |
| `Models.OpenAi.GptImage1` | `gpt-image-1` (latest image model) |

---

## Anthropic

`BackendType.Anthropic` — set `AnthropicKey` in config.

### Latest generation (5th gen)

| Constant | API ID | Notes |
|---|---|---|
| `Models.Anthropic.ClaudeFable5` | `claude-fable-5` | 5th-gen Mythos-level, ambitious long-running work |
| `Models.Anthropic.ClaudeMythos5` | `claude-mythos-5` | Most advanced, vetted cybersecurity/biology research only |
| `Models.Anthropic.ClaudeSonnet5` | `claude-sonnet-5` | Frontier performance, professional work |
| `Models.Anthropic.ClaudeOpus4_8` | `claude-opus-4-8` | Previous flagship, strong reasoning |

### 4th generation

| Constant | API ID |
|---|---|
| `Models.Anthropic.ClaudeOpus4_7` | `claude-opus-4-7` |
| `Models.Anthropic.ClaudeSonnet4_6` | `claude-sonnet-4-6` |
| `Models.Anthropic.ClaudeSonnet4_5` | `claude-sonnet-4-5-20250929` |
| `Models.Anthropic.ClaudeSonnet4` | `claude-sonnet-4-20250514` |
| `Models.Anthropic.ClaudeHaiku4_5` | `claude-haiku-4-5-20251001` |

---

## Gemini (Google AI)

`BackendType.Gemini` — set `GeminiKey` in config.

| Constant | API ID |
|---|---|
| `Models.Gemini.Gemini3_5Flash` | `gemini-3.5-flash` — latest flagship, agentic tasks |
| `Models.Gemini.Gemini3_1ProPreview` | `gemini-3.1-pro-preview` — most capable Pro model |
| `Models.Gemini.Gemini3_1FlashLite` | `gemini-3.1-flash-lite` — low-latency, cost-efficient |
| `Models.Gemini.Gemini2_5Pro` | `gemini-2.5-pro` — most capable Gemini |
| `Models.Gemini.Gemini2_5Flash` | `gemini-2.5-flash` — fast and efficient |
| `Models.Gemini.Gemini2_0Flash` | `gemini-2.0-flash` — optimized for speed |
| `Models.Gemini.Gemini3_1FlashImagePreview` | `gemini-3.1-flash-image-preview` — image generation |
| `Models.Gemini.GeminiImagen4_0FastGenerate` | `imagen-4.0-fast-generate-001` — fast image generation |
| `Models.Gemini.GeminiNanoBanana` | `gemini-2.5-flash-image` — high-speed image generation |

---

## GroqCloud

`BackendType.GroqCloud` — set `GroqCloudKey` in config. Ultra-fast inference.

| Constant | API ID | Notes |
|---|---|---|
| `Models.Groq.Llama4Scout17b` | `meta-llama/llama-4-scout-17b-16e-instruct` | MoE model, vision support |
| `Models.Groq.Llama3_3_70b` | `llama-3.3-70b-versatile` | High-performance multilingual |
| `Models.Groq.Qwen3_32b` | `qwen/qwen3-32b` | Alibaba's Qwen 3 32B |
| `Models.Groq.Llama3_1_8b` | `llama-3.1-8b-instant` | Fast Llama 3.1 8B |
| `Models.Groq.GptOss120b` | `openai/gpt-oss-120b` | Open-source 120B GPT |
| `Models.Groq.GptOss20b` | `openai/gpt-oss-20b` | Open-source 20B GPT |
| `Models.Groq.Compound` | `groq/compound` | Built-in web search & code execution |
| `Models.Groq.CompoundMini` | `groq/compound-mini` | Lightweight compound system |

---

## DeepSeek

`BackendType.DeepSeek` — set `DeepSeekKey` in config.

| Constant | API ID |
|---|---|
| `Models.DeepSeek.Chat` | `deepseek-chat` |
| `Models.DeepSeek.Reasoner` | `deepseek-reasoner` |

---

## xAI

`BackendType.Xai` — set `XaiKey` in config.

| Constant | API ID |
|---|---|
| `Models.Xai.Grok4_20Reasoning` | `grok-4.20-reasoning` |
| `Models.Xai.Grok4_20NonReasoning` | `grok-4.20-non-reasoning` |
| `Models.Xai.Grok4_1FastReasoning` | `grok-4-1-fast-reasoning` |
| `Models.Xai.Grok4_1Fast` | `grok-4-1-fast-non-reasoning` |
| `Models.Xai.Grok3Beta` | `grok-3-beta` |

---

## Ollama

`BackendType.Ollama` — set `OllamaKey` to the Ollama server URL (e.g. `http://localhost:11434`). Use Ollama Cloud for hosted inference.

### Gemma family
| Constant | API ID | Model |
|---|---|---|
| `Models.Ollama.Gemma3_4b` | `gemma3:4b` | Gemma 3 4B via Ollama |
| `Models.Ollama.Gemma3_12b` | `gemma3:12b` | Gemma 3 12B via Ollama |
| `Models.Ollama.Gemma3_27b` | `gemma3:27b` | Gemma 3 27B via Ollama |
| `Models.Ollama.Gemma4_E4b` | `gemma4:e4b` | Gemma 4 E4B via Ollama |
| `Models.Ollama.Gemma4_26b` | `gemma4:26b` | Gemma 4 26B via Ollama |

### Llama family
| Constant | API ID | Model |
|---|---|---|
| `Models.Ollama.Llama4Scout` | `llama4:scout` | Llama 4 Scout via Ollama |
| `Models.Ollama.Llama4Maverick` | `llama4:maverick` | Llama 4 Maverick via Ollama |

### Qwen family
| Constant | API ID | Model |
|---|---|---|
| `Models.Ollama.Qwen3_8b` | `qwen3:8b` | Qwen 3 8B via Ollama |
| `Models.Ollama.Qwen3_14b` | `qwen3:14b` | Qwen 3 14B via Ollama |
| `Models.Ollama.Qwen3_30b` | `qwen3:30b` | Qwen 3 30B MoE via Ollama |
| `Models.Ollama.Qwen3_5_9b` | `qwen3.5:9b` | Qwen 3.5 9B multimodal via Ollama |
| `Models.Ollama.Qwen3_5_27b` | `qwen3.5:27b` | Qwen 3.5 27B multimodal via Ollama |
| `Models.Ollama.Qwen3_6_27b` | `qwen3.6:27b` | Qwen 3.6 27B agentic via Ollama |
| `Models.Ollama.Qwen3_6_35b` | `qwen3.6:35b` | Qwen 3.6 35B via Ollama |
| `Models.Ollama.Qwen3Coder_8b` | `qwen3-coder:8b` | Qwen 3 Coder 8B via Ollama |
| `Models.Ollama.Qwen3Coder_30b` | `qwen3-coder:30b` | Qwen 3 Coder 30B via Ollama |

### Other
| Constant | API ID | Model |
|---|---|---|
| `Models.Ollama.DeepSeekR1_7b` | `deepseek-r1:7b` | DeepSeek R1 7B via Ollama |
| `Models.Ollama.DeepSeekR1_14b` | `deepseek-r1:14b` | DeepSeek R1 14B via Ollama |
| `Models.Ollama.Phi4_14b` | `phi4:14b` | Phi 4 14B via Ollama |
| `Models.Ollama.Mistral_7b` | `mistral:7b` | Mistral 7B via Ollama |

Plain strings work too: `.WithModel("qwen2.5:7b")` uses whatever tag is pulled in Ollama.

---

## Vertex AI (Google Cloud)

`BackendType.Vertex` — requires service account credentials configured in the environment.

| Constant | API ID | Model |
|---|---|---|
| `Models.Vertex.Gemini2_5Pro` | `google/gemini-2.5-pro` | Gemini 2.5 Pro via Vertex |
| `Models.Vertex.Gemini2_5Flash` | `google/gemini-2.5-flash` | Gemini 2.5 Flash via Vertex |
| `Models.Vertex.Veo2_0` | `google/veo-2.0-generate-001` | Veo 2.0 (video generation) |
| `Models.Vertex.Imagen4_0` | `google/imagen-4.0-generate-001` | Imagen 4.0 (image generation) |

---

## Vision-capable models

Models that accept image input via `.WithMessage(text, imageBytes)` or `Message.Images`:

**Local:**
- `Models.Local.Gemma3_4b`
- `Models.Local.Gemma3_12b`
- `Models.Local.Llava_7b`, `Models.Local.Llava16Mistral_7b`
- `Models.Local.Qwen3_5_4b`, `Models.Local.Qwen3_5_9b`, `Models.Local.Qwen3_VL_32b_Instruct`

**Cloud:**
- `Models.OpenAi.*` (GPT-5.6 series, GPT-4o, GPT-5)
- `Models.Anthropic.*` (all Claude models)
- `Models.Gemini.*` (all)
- `Models.Xai.Grok3Beta`, `Models.Xai.Grok4_20Reasoning`
- `Models.Groq.Llama4Scout17b`

---

## Environment Variables

For InferPage, set the lowercase `model` environment variable to the exact API ID from the tables. This value is read into `Utils.Model`. When using the supplied Docker Compose file, set `MODEL_NAME` to the same value; Compose passes it through to `model`.

| Variable | Purpose |
|---|---|
| `model` | Exact catalog API ID passed to InferPage and assigned to `Utils.Model` |
| `MODEL_NAME` | Compose input mapped to InferPage's `model` setting |
| `BACKEND` | `self`, `openai`, `gemini`, `deepseek`, `groqcloud`, `anthropic`, `xai`, `ollama`, or `vertex` |
| `OPENAI_API_KEY` | OpenAI credentials |
| `GEMINI_API_KEY` | Gemini credentials |
| `DEEPSEEK_API_KEY` | DeepSeek credentials |
| `GROQ_API_KEY` | Groq Cloud credentials |
| `ANTHROPIC_API_KEY` | Anthropic credentials |
| `XAI_API_KEY` | xAI credentials |
| `OLLAMA_API_KEY` | Ollama server URL or cloud key |
| `GOOGLE_APPLICATION_CREDENTIALS` | Vertex service-account credentials path |

## Tool calling format support (Local models)

MaIN.NET automatically detects the model's native tool-call format and generates the appropriate prompt:

| Model Family | Format | Example |
|---|---|---|
| Qwen2.5, Qwen3, Qwen3.5, Hermes | Hermes JSON | `<tool_call>{"tool_calls": [{"name": "...", "arguments": "..."}]}</tool_call>` |
| IBM Granite | Granite | `<tool_call>{"name": "...", "arguments": {...}}</tool_call>` |
| Llama 3.1+ | Llama 3 | `<functioncall>{"name": "...", "arguments": "..."}</functioncall>` |
| Mistral v0.3+ | Mistral v3 | `[TOOL_CALLS][{"name": "...", "arguments": {...}}]` |
| Phi-3.5, Phi-4 | Phi 3.5 | `<|tool_call|>{"name": "...", "arguments": {...}}<|/tool_call|>` |

**Best models for tool calling (recommended):** `Qwen3_5_4b` (newest, best overall), `Granite4_1_3b` (enterprise), `Nemotron3Nano4b` (edge)

---

## Inference params by backend

| Backend | Params class | Key fields |
|---|---|---|
| `Self` | `LocalInferenceParams` | `Temperature`, `TopP`, `MaxTokens`, `Grammar` |
| `OpenAi` | `OpenAiInferenceParams` | `ResponseFormat`, `AdditionalParams` |
| `Vertex` | `VertexInferenceParams` | `Location` (GCP region) |
| Others | none required | pass nothing, provider defaults apply |

---

## InferPage OpenAI-compatible API

InferPage exposes all models via an OpenAI-compatible HTTP API at `http://localhost:5555/v1`. See [inferpage-api.md](inferpage-api.md) for the full API reference.

Key features:
- `/v1/chat/completions` and `/v1/responses` endpoints
- OpenAI hosted tool format support: `{"type": "web_search"}`, `{"type": "web_search_preview"}`
- Custom function tool support
- Vision input
- Streaming (SSE)
- Drop-in replacement for OpenAI API clients

---

## See Also

- [chat.md](chat.md) — ChatContext and tool calling
- [agents.md](agents.md) — AgentContext and tool calling
- [comparison.md](comparison.md) — Backend comparison
- [project-structure.md](project-structure.md) — Repository layout
- [inferpage-api.md](inferpage-api.md) — OpenAI-compatible HTTP API
