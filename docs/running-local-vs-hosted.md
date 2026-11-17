# Running it: local vs hosted

You have two ways to use an uncensored model: run the weights yourself, or call
a hosted API. Here's the honest tradeoff.

## Local

**You download the weights and run them on your own GPU.**

Pros: fully offline, no per-token cost, nothing leaves your machine, total
control over the model and sampling.

Cons: you buy or rent the hardware, you maintain the stack, and a 30B-class
model needs real VRAM.

### What it takes

A 30B mixture-of-experts (MoE) model at 4-bit quantization needs roughly
**18-20 GB of VRAM** for weights, plus more for the KV cache at long context.

| Setup | VRAM | Verdict |
|---|---|---|
| 1x RTX 3090 / 4090 | 24 GB | works at short/medium context |
| 2x RTX 3090 | 48 GB | comfortable, long context |
| Apple M-series (unified) | 32 GB+ | works, slower prompt processing |
| CPU only | — | runs, painfully slow |

**Memory bandwidth, not FLOPs, is the bottleneck** for single-user generation.
Token generation is memory-bound: each token reads a slice of the weights from
VRAM. A card with more bandwidth generates faster even at the same compute. Size
by VRAM and bandwidth first.

### Runners

- **llama.cpp** — the reference engine; `llama-server` gives you an
  OpenAI-compatible endpoint.
- **Ollama** — one command, friendliest.
- **vLLM** — highest throughput if you serve multiple users.

Use GGUF quantizations (Q4_K_M and up). See
[awesome-uncensored-ai](https://github.com/TartarusAIDev/awesome-uncensored-ai).

### Rent before you buy

Before dropping money on GPUs, rent by the hour (vast.ai, RunPod, Lambda). A
couple of 3090s is a few dollars an hour. You'll learn whether you actually run
it enough to justify owning it.

## Hosted

**Someone else runs the model; you call an API.**

Pros: no GPU, no setup, no maintenance, scales instantly, and you get long
context without sizing a KV cache.

Cons: per-token cost, you trust the host, and you're tied to their model
choices.

[Tartarus](https://tartarusai.dev) is the hosted path with a coding focus: an
OpenAI-compatible `/v1` endpoint and a CLI, billed in crypto with no KYC. Point
any OpenAI SDK at it and nothing else in your code changes.

## Which should you pick?

- **Privacy-critical / offline / high volume** → local.
- **Want it working in five minutes, no hardware** → hosted.
- **Both** → prototype hosted, move hot paths local once you know your usage.
