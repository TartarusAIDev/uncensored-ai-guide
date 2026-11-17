# How abliteration works

Most "uncensored" open models you'll find are **abliterated**. The name is a
portmanteau of "ablate" + "obliterate." Here's the actual mechanism, without the
mysticism.

## The one-direction finding

In 2024, Arditi et al. ("Refusal in Language Models Is Mediated by a Single
Direction") showed something surprising: across many instruction-tuned models,
the decision to *refuse* is largely encoded as **one direction** in the model's
residual-stream activation space. Not a whole subnetwork. One direction.

Concretely:

1. Run a batch of **harmful-looking** prompts and a batch of **harmless**
   prompts through the model.
2. At some layer, take the mean activation for each batch.
3. The difference of those means is the **"refusal direction."**

## Ablating it

Once you have that direction `r` (a unit vector), you remove the model's ability
to represent it. For every activation `x` where the model would normally use it,
you project `r` out:

```
x' = x - (x · r) r
```

Do this at inference time (cheap, reversible) or **bake it into the weights** by
editing the matrices that write to and read from the residual stream (permanent,
no runtime cost). The baked version is what gets uploaded as an "abliterated"
model.

No retraining. No new data. You're doing linear algebra on existing weights.

## What it does and doesn't change

- **Does:** removes most refusals. The model stops saying "I can't help with
  that" and attempts the task.
- **Doesn't:** make the model smarter, or change its coding ability. Capability
  is roughly preserved; a sloppy abliteration can dent quality slightly, which is
  why the layer and prompt sets matter.
- **Doesn't:** add new knowledge. If the base model couldn't do it, the
  abliterated one can't either.

## Tools

Abliteration has gone from research notebooks to one-command tools. Automated
abliterators search for the best direction and layer for you and emit a ready
GGUF/safetensors. Search Hugging Face for
[`abliterated`](https://huggingface.co/models?search=abliterated) to find
community versions of popular open models.

## Caveats

- A heavy-handed abliteration can make a model *over*-comply or get repetitive.
- Quantization interacts with it; test the actual GGUF you'll run, not the
  fp16 original.
- Measure, don't vibe: run [refusal-bench](https://github.com/TartarusAIDev/refusal-bench)
  before and after.
