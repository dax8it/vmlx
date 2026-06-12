# N2 JANG_1L Runtime-Fit Raw Decode Probe - 2026-06-11

Artifact:

`/Users/eric/jangq-ai/Nex-N2-Pro-JANG_1L-full-runtimefit-20260611`

## Result

Status: fail, output-quality blocked.

The vMLX runtime is not dropping visible text in Chat/Responses assembly. The
model is generating token id `220` repeatedly, and the tokenizer decodes that
token stream to spaces both with and without special-token skipping.

Observed diagnostic excerpt from Chat Completions:

```json
{
  "completion_tokens": 8,
  "decode_keep_specials_head": "        ",
  "decode_skip_specials_head": "        ",
  "finish_reason": "length",
  "raw_text_head": "        ",
  "text_head": "",
  "token_ids_count": 8,
  "token_ids_head": [220, 220, 220, 220, 220, 220, 220, 220]
}
```

## Probes

- Chat Completions first miss:
  `chat1.json`
  - `message.content=null`
  - `completion_tokens=8`
  - `finish_reason=length`
  - raw token diagnostic: eight `220` tokens -> eight spaces
  - speed: 8 tokens in 47.69s, 0.2 tok/s

- Chat Completions cache hit:
  `chat2-cache.json`
  - `message.content=null`
  - `cached_tokens=10`
  - `cache_detail=paged+ssm+tq`
  - raw token diagnostic: eight `220` tokens -> eight spaces
  - speed: 8 tokens in 3.31s, 2.4 tok/s

- Responses API cache hit:
  `responses1.json`
  - `output=[]`
  - `output_text=null`
  - `output_tokens=8`
  - `cache_detail=paged+ssm+tq`
  - raw token diagnostic: eight `220` tokens -> eight spaces
  - speed: 8 tokens in 3.23s, 2.5 tok/s

- Raw Completions prompt:
  `completions-raw-prompt.json`
  - `choices[0].text=""`
  - `completion_tokens=8`
  - `finish_reason=length`
  - speed: 8 tokens in 1.27s, 6.3 tok/s

- No live TurboQuant control:
  `chat-nolive-tq.json`
  - launched with `VMLINUX_FORCE_TQ_AUTO=0`
  - live attention cache was standard `KVCache`, not `TurboQuantKVCache`
  - `message.content=null`
  - raw token diagnostic: eight `220` tokens -> eight spaces
  - speed: 8 tokens in 38.56s, 0.2 tok/s

## Cache And Memory

The cache/runtime path is functional but does not fix output quality:

- Hybrid layout: 15 attention KV layers + 45 SSM layers.
- Live TurboQuant run used `TurboQuantKVCache` for attention layers.
- Cache hit detail: `paged+ssm+tq`.
- Cache hit tokens after Chat/Responses probe: `20`.
- L2 block tokens on disk: `20`; L2 SSM tokens on disk: `20`.
- Health after probe reported active memory about `108.6 GB`, peak about
  `109.7 GB`.

## Classification

This is not a Chat Completions message assembly bug, not a Responses API output
assembly bug, and not reasoning parser suppression. It also reproduces with
live TurboQuant disabled.

Current most likely boundary: quantization/format/runtime matmul/logit collapse
before token selection, or a model-format mismatch in the JANG affine N2 path.
The JANG side should compare first-token logits / top-k token ids against a
source or high-bit artifact for the same prompt. The immediate bad token is
`220` (space), repeated greedily.

The vMLX side still needs to preserve these diagnostics and improve wired-limit
reporting, but this artifact is not a healthy runtime model yet.
