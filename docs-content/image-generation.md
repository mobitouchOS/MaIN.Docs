# Image Generation in MaIN.NET

MaIN.NET supports AI image generation through the same `ChatContext` API used for text. Switching from text to image generation only requires changing the model — no new API to learn.

---

## Supported Backends

| Backend | Models | Config required |
|---------|--------|----------------|
| OpenAI | DALL-E 3, GPT Image 1 | `OpenAiKey` |
| Gemini | Imagen 4.0 Fast, Gemini 2.5 Flash Image | `GeminiKey` |
| xAI (Grok) | Grok Image, Grok Imagine, Grok Imagine Pro | `XaiKey` |
| Vertex AI | Imagen 4.0, Veo 2.0 | `GoogleServiceAccountAuth` |
| Local (Self) | Stable Diffusion 1.5, FLUX.1 Schnell, Qwen Image | None — runs in-process via GGUF |

**Not supported:** Anthropic, DeepSeek, GroqCloud, Ollama.

---

## The API

Image generation uses `AIHub.Chat()` — exactly like text chat, just with an image-capable model. MaIN detects image generation automatically from the model type.

```csharp
var result = await AIHub.Chat()
    .WithModel(Models.OpenAi.DallE3)
    .WithMessage("A cyberpunk cat warrior in neon rain")
    .CompleteAsync();

byte[] imageBytes = result.Message.Image;
```

The result is always a `ChatResult` where `result.Message.Image` contains the raw image as a `byte[]`.

---

## Model Constants

```csharp
// OpenAI
Models.OpenAi.DallE3           // "dall-e-3"
Models.OpenAi.GptImage1        // "gpt-image-1"

// Gemini
Models.Gemini.Imagen4_0_FastGenerate   // "imagen-4.0-fast-generate-001"
Models.Gemini.NanoBanana               // "gemini-2.5-flash-image"

// xAI (Grok)
Models.Xai.GrokImage           // "grok-2-image"
Models.Xai.GrokImagineImage    // "grok-imagine-image"
Models.Xai.GrokImagineImagePro // "grok-imagine-image-pro"

// Vertex AI
Models.Vertex.Imagen4_0_Generate   // "google/imagen-4.0-generate-001"
Models.Vertex.Veo2_0_Generate      // "google/veo-2.0-generate-001"

// Local (in-process, no external service required)
Models.Local.StableDiffusion1_5   // "stable-diffusion-1.5" — 512×512, SD1, CPU-friendly
Models.Local.Flux1Shnell          // "FLUX.1_Shnell"        — 768×768, FLUX, good quality
Models.Local.QwenImage            // "qwen-image"           — 1024×1024, highest quality
```

---

## Examples by Backend

### OpenAI (DALL-E 3)

```csharp
using Examples.Utils;
using MaIN.Core.Hub;
using MaIN.Domain.Models;

namespace Examples.Chat;

public class ChatWithImageGenOpenAiExample : IExample
{
    public async Task Start()
    {
        OpenAiExample.Setup();

        var result = await AIHub.Chat()
            .WithModel(Models.OpenAi.DallE3)
            .WithMessage("A rock-style cow playing electric guitar on stage")
            .CompleteAsync();

        // result.Message.Image is byte[] — save, display, or forward as needed
        await File.WriteAllBytesAsync("output.png", result.Message.Image!);
    }
}
```

### Gemini (Imagen)

```csharp
using Examples.Utils;
using MaIN.Core.Hub;
using MaIN.Domain.Models;
using MaIN.Domain.Models.Concrete;
using MaIN.Services.Registry;

namespace Examples.Chat;

public class ChatWithImageGenGeminiExample : IExample
{
    public async Task Start()
    {
        GeminiExample.Setup();

        // Option A: use a built-in model constant
        var result = await AIHub.Chat()
            .WithModel(Models.Gemini.Imagen4_0_FastGenerate)
            .WithMessage("A hamster astronaut on the moon")
            .CompleteAsync();

        // Option B: register a custom model ID at runtime
        var customModel = new GenericImageGenerationCloudModel("imagen-3", BackendType.Gemini);
        ModelRegistry.RegisterOrReplace(customModel);

        var result2 = await AIHub.Chat()
            .WithModel(customModel.Id)
            .WithMessage("A hamster astronaut on the moon")
            .CompleteAsync();

        await File.WriteAllBytesAsync("output.png", result.Message.Image!);
    }
}
```

### xAI (Grok Image)

```csharp
using MaIN.Core;
using MaIN.Core.Hub;
using MaIN.Domain.Configuration;
using MaIN.Domain.Models;

namespace Examples.Chat;

public class ChatWithImageGenXaiExample : IExample
{
    public async Task Start()
    {
        MaINBootstrapper.Initialize(options =>
        {
            options.BackendType = BackendType.Xai;
            options.XaiKey = "your-xai-key";
        });

        var result = await AIHub.Chat()
            .WithModel(Models.Xai.GrokImage)
            .WithMessage("A futuristic city floating above clouds at sunset")
            .CompleteAsync();

        await File.WriteAllBytesAsync("output.png", result.Message.Image!);
    }
}
```

### Vertex AI (Imagen 4.0)

```csharp
using MaIN.Core;
using MaIN.Core.Hub;
using MaIN.Domain.Configuration;
using MaIN.Domain.Models;

namespace Examples.Chat;

public class ChatWithImageGenVertexExample : IExample
{
    public async Task Start()
    {
        MaINBootstrapper.Initialize(options =>
        {
            options.BackendType = BackendType.Vertex;
            options.GoogleServiceAccountAuth = new GoogleServiceAccountConfig
            {
                ProjectId = "my-gcp-project",
                PrivateKeyId = "...",
                PrivateKey = "-----BEGIN PRIVATE KEY-----\n...",
                ClientEmail = "my-sa@my-project.iam.gserviceaccount.com",
            };
        });

        var result = await AIHub.Chat()
            .WithModel(Models.Vertex.Imagen4_0_Generate)
            .WithMessage("A photorealistic mountain lake at dawn")
            .CompleteAsync();

        await File.WriteAllBytesAsync("output.png", result.Message.Image!);
    }
}
```

### Local — in-process diffusion (no external service)

Local image generation runs entirely in-process via `StableDiffusion.NET` (stable-diffusion.cpp). No separate service, no `ImageGenUrl` — just register the model, download it once, and generate.

**Stable Diffusion 1.5** — fastest, single-file GGUF, great for CPU or low-VRAM machines:

```csharp
using MaIN.Core.Hub;
using MaIN.Domain.Models;
using MaIN.Domain.Models.Concrete;
using MaIN.Services.Registry;

ModelRegistry.RegisterOrReplace(new StableDiffusion1_5());
await AIHub.Model().EnsureDownloadedAsync(Models.Local.StableDiffusion1_5);

var result = await AIHub.Chat()
    .WithModel(Models.Local.StableDiffusion1_5)
    .WithMessage("Fluffy cat with a book - anime style")
    .CompleteAsync();

await File.WriteAllBytesAsync("output.png", result.Message.Image!);
```

**FLUX.1 Schnell** — better quality, downloads additional VAE + text-encoder assets automatically:

```csharp
ModelRegistry.RegisterOrReplace(new Flux1Shnell());
await AIHub.Model().EnsureDownloadedAsync(Models.Local.Flux1Shnell);

var result = await AIHub.Chat()
    .WithModel(Models.Local.Flux1Shnell)
    .WithMessage("Cyberpunk godzilla cat warrior")
    .CompleteAsync();

await File.WriteAllBytesAsync("output.png", result.Message.Image!);
```

**Qwen Image** — highest quality (1024×1024), requires significant RAM/VRAM, downloads a large Qwen2.5-VL text encoder:

```csharp
ModelRegistry.RegisterOrReplace(new QwenImage());
await AIHub.Model().EnsureDownloadedAsync(Models.Local.QwenImage);

var result = await AIHub.Chat()
    .WithModel(Models.Local.QwenImage)
    .WithMessage("A photorealistic mountain lake at dawn")
    .CompleteAsync();

await File.WriteAllBytesAsync("output.png", result.Message.Image!);
```

**Key facts about local diffusion:**
- `EnsureDownloadedAsync` downloads all required assets (VAE, CLIP, T5-XXL, etc.) automatically — you don't manage them separately
- Models are loaded once and cached in memory for the lifetime of the process
- `ModelsPath` config (or `MaIN_ModelsPath` env var) controls where GGUF files are stored
- No `ImageGenUrl` or external service needed

---

## Custom Model Registration

If you need to use a model ID not in the `Models.*` constants, register it at runtime:

```csharp
using MaIN.Domain.Models.Abstract;
using MaIN.Domain.Models.Concrete;
using MaIN.Services.Registry;

// Cloud image generation model (picks up the current backend's service):
var model = new GenericImageGenerationCloudModel("my-model-id", BackendType.Gemini);
ModelRegistry.RegisterOrReplace(model);

// Local in-process diffusion model (GGUF via stable-diffusion.cpp):
var localModel = new GenericLocalImageGenerationModel(
    FileName: "my-sd-model-Q8_0.gguf",
    Architecture: DiffusionArchitecture.SD1,
    Name: "My Custom SD Model",
    Width: 512,
    Height: 512,
    Steps: 25,
    CfgScale: 7.0f
);
ModelRegistry.RegisterOrReplace(localModel);
```

For FLUX-based custom models that need separate encoder assets:
```csharp
var fluxModel = new GenericLocalImageGenerationModel(
    FileName: "my-flux-model-Q4_0.gguf",
    Architecture: DiffusionArchitecture.Flux,
    Name: "My FLUX Model",
    Width: 768,
    Height: 768,
    Steps: 4,
    CfgScale: 1.0f,
    Vae:   new ModelAsset("ae.safetensors",            new Uri("https://...")),
    ClipL: new ModelAsset("clip_l.safetensors",        new Uri("https://...")),
    T5Xxl: new ModelAsset("t5xxl_fp8_e4m3fn.safetensors", new Uri("https://..."))
);
ModelRegistry.RegisterOrReplace(fluxModel);
```

---

## Result Structure

```csharp
ChatResult result = await AIHub.Chat()
    .WithModel(Models.OpenAi.DallE3)
    .WithMessage("prompt")
    .CompleteAsync();

result.Message.Image    // byte[] — the generated image (first image)
result.Message.Images   // List<byte[]> — all generated images (usually one)
result.Model            // string — model ID used
result.Done             // bool
result.CreatedAt        // DateTime
```

---

## How Detection Works

MaIN detects image generation automatically — no extra flag needed. When you call `.WithModel(id)`, the framework checks whether the model implements `IImageGenerationModel`. If it does, the request is automatically routed to the image generation service for that backend instead of the text completion service.

This means the same code pattern works for both text and images — only the model ID changes.

---

## Multi-Turn Image Prompts

Multiple `.WithMessage()` calls or a `WithMessages(history)` list work the same way as in text chat. All messages are concatenated into a single prompt (first message as-is, subsequent ones prefixed with `&&`).

```csharp
var result = await AIHub.Chat()
    .WithModel(Models.OpenAi.DallE3)
    .WithMessage("A red dragon")
    .WithMessage("flying over a medieval castle")
    .WithMessage("at dusk, painted in watercolor style")
    .CompleteAsync();
```
