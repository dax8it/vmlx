# N2 JANG_1L Linear4/Out4 Raw Decode Probe - 2026-06-11

Artifact:

`/Users/eric/jangq-ai/Nex-N2-Pro-JANG_1L-full-runtimefit-linear4-out4-20260611`

## Result

Status: fail, output-quality blocked.

Restoring linear-attention input projections to 4-bit did not fix the repeated
space-token collapse. The first Chat Completions probe generated token id `220`
eight times, and tokenizer decode with special tokens kept/skipped is eight
spaces in both cases.

Observed diagnostic excerpt:

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

## Probe Configuration

- Source vMLX server on port `8135`.
- `VMLINUX_FORCE_TQ_AUTO=1`.
- `--reasoning-parser none`.
- `--default-enable-thinking false`.
- Deterministic sampling: `temperature=0`, `top_p=1`.
- Paged cache + block disk cache enabled.
- Prompt: `Reply with exactly: blue cat`.

## Runtime Evidence

- First Chat response:
  - HTTP 200.
  - `message.content=null`.
  - `completion_tokens=8`.
  - `finish_reason=length`.
  - Warning surfaced by vMLX blank-generation diagnostic.
  - Raw token ids: eight `220` tokens.
  - Speed: 8 tokens in 44.59s, `0.2 tok/s`.

- Subsequent Chat/Responses/raw Completions requests:
  - HTTP 503 from Metal working-set guard.
  - Guard reason: active working set 99.1% of 107.5GB cap.
  - This means the 107.252 GiB artifact is runtime-loadable but too tight for
    repeated probes under the current working-set cap after the first decode.

## Classification

This artifact is not healthy. The linear-attn 4-bit input restoration did not
remove or change the repeated token `220` collapse in the first greedy decode.

This does not look like Chat/Responses assembly, tokenizer special-token
filtering, or reasoning-parser suppression. The first generated token stream is
already whitespace before API assembly.

The next most direct split is first-token logits/top-k against source or a
higher-bit artifact, plus runtime affine matmul/upcast validation for the N2
JANG affine path. The artifact also triggers 467 runtime quant-shape metadata
repairs because top-level config remains uniform `bits=2 group_size=128` while
tensor shapes encode mixed precision.

## Cleanup

The vMLX server on port `8135` was stopped after the probe. No intentional N2
probe server remains from this run.
