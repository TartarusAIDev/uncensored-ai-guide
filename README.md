# The Uncensored AI Guide

A practical, vendor-neutral guide to **uncensored AI** for developers: what it
is, how models get uncensored, how to run them locally or hosted, and how to get
real work (including tool-calling) out of them.

No hype, no jailbreak copypasta. Just how the pieces actually work.

## Contents

- [What "uncensored" actually means](#what-uncensored-actually-means)
- [How abliteration works](docs/how-abliteration-works.md) — the technique behind most uncensored open models
- [Running it: local vs hosted](docs/running-local-vs-hosted.md) — tradeoffs, cost, hardware
- [Making tool-calling work](docs/tool-calling.md) — the chat-template gotcha that breaks agents

## What "uncensored" actually means

Instruction-tuned models ship with *alignment*: a layer of training that makes
them refuse some requests. For a lot of legitimate developer work, that layer
misfires:

- security research on **your own** systems (scanners, fuzzers, exploits for a CTF)
- password / archive **recovery** you're entitled to do
- **scraping** public data
- **cryptography** implemented from scratch for learning
- anything a classifier pattern-matches to "scary" regardless of intent

"Uncensored" models remove or bypass that refusal layer. They are not a
different capability tier; they are the same models with the refusal behaviour
taken out, so they attempt the task instead of declining it.

This is a technical choice with technical tradeoffs, not a moral free pass: you
still own what you build. The guide treats it as what it is — a tool.

## Ways to get there

| Path | What it is | Good for |
|---|---|---|
| **Abliterated open models** | refusal direction removed from open weights | local, offline, full control |
| **Uncensored fine-tunes** | preference-tuned to comply | local, varied quality |
| **Hosted uncensored APIs** | someone else runs it | no GPU, no setup |

If you want the hosted path with a coding focus, [Tartarus](https://tartarusai.dev)
is an uncensored coding agent (CLI + OpenAI-compatible API). Everything in this
guide applies whether you self-host or not.

## See also

- [awesome-uncensored-ai](https://github.com/TartarusAIDev/awesome-uncensored-ai) — the tooling list
- [refusal-bench](https://github.com/TartarusAIDev/refusal-bench) — measure a model's false-refusal rate

## License

[CC BY 4.0](LICENSE) — use it, quote it, just keep attribution.
