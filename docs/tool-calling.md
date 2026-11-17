# Making tool-calling work

The single most common reason an uncensored open model "doesn't work as an
agent" isn't the weights. It's the **chat template**.

## The problem

Agentic coding needs the model to emit tool calls in a structured form the
runtime can parse (OpenAI's `tool_calls` array). The model only does that if the
**chat template** (the Jinja template that formats messages before they hit the
model) renders tools correctly and the model was trained to match it.

When a GGUF is converted from an upstream model, it embeds whatever template the
upstream had **at conversion time**. If the upstream later fixed its template
(this happens often), the GGUF is frozen with the old, broken one. Symptom: the
model emits malformed tool calls — mixed bracket styles, `function=NAME`
fragments, half-JSON — and your agent loop breaks.

## The fix

Override the template at launch instead of trusting the embedded one. With
llama.cpp:

```bash
llama-server \
  --model your-model.gguf \
  --jinja \
  --chat-template-file fixed_chat_template.jinja
```

Get the corrected template from the upstream model's current repo (the
`chat_template.jinja` file), not from the GGUF. With the right template,
llama-server emits canonical OpenAI `tool_calls` arrays directly and you need no
regex shim in your backend.

## Other switches that matter

- **Turn reasoning off** for non-thinking models. Some templates can flip a
  `<think>` mode on; stray `<think>` blocks before a tool call waste tokens and
  can cause a premature end-of-sequence. Pass the launcher's reasoning-off flag
  explicitly.
- **Temperature 0** when you want deterministic tool calls.
- **Validate the arguments.** Even a correct template can emit arguments that
  don't match your JSON schema; validate before executing.

## If you use a hosted API

A good host has already done all of this. [Tartarus](https://tartarusai.dev),
for example, serves canonical OpenAI `tool_calls` out of the box, so the
[tartarus-sdk](https://github.com/TartarusAIDev/tartarus-sdk) and any OpenAI SDK
get working tool calls with no template wrangling.

## Test it

```bash
curl $BASE/chat/completions -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" -d '{
    "model": "<model>",
    "messages": [{"role":"user","content":"list files here"}],
    "tools": [{"type":"function","function":{"name":"ls","parameters":{"type":"object","properties":{}}}}]
  }' | jq ".choices[0].message.tool_calls"
```

If that returns a clean array, your agent will work.
