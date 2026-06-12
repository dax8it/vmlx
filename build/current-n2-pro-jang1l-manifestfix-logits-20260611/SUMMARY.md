# N2 JANG_1L Manifest-Fix First-Token Proof - 2026-06-11

Artifact:

`/Volumes/EricsLLMDrive/jangq-ai/Nex-N2-Pro-JANG_1L-manifestfix-20260611`

## Result

Status: fail, first-token/logits blocked.

The manifest-fix artifact loads without the previous quant-shape repair warning,
but the first generated token is still token id `220` (`" "`). Chat
Completions logprobs show the literal space token as top-1 before parser,
cache, or API assembly can explain the blank output.

## Metadata And Load

- Source vMLX server on port `8137`.
- `VMLINUX_FORCE_TQ_AUTO=1`.
- `--reasoning-parser none`.
- `--default-enable-thinking false`.
- Paged cache + block disk cache enabled.
- Startup output did not emit the previous `quant_shape_inference: patched ...`
  warning. For this probe, quant-shape repair count is treated as `0`.
- Runtime path:
  - JANG_1L affine quantized matmul.
  - Metal NA eligible/active.
  - Hybrid cache: 15 attention KV layers and 45 SSM companion layers.
  - Live TurboQuant KV active for attention layers only.
  - Storage-boundary KV quantization q4 active for prefix/paged/L2.

## First-Token Logprobs

Request:

- Prompt: `Reply with exactly: blue cat`.
- `max_tokens=1`.
- `temperature=0`, `top_p=1`.
- `logprobs=true`, `top_logprobs=10`.
- `enable_thinking=false`.

Response:

- HTTP 200.
- `completion_tokens=1`.
- `message.content=null`.
- `finish_reason=length`.
- Raw diagnostic token ids: `[220]`.
- Decode with specials kept/skipped: `" "`.
- Speed: 1 token in 85.31s.

Top-10 tokens from Chat Completions logprobs:

| Rank | Token | Logprob |
| --- | --- | --- |
| 1 | `" "` | `-7.21875` |
| 2 | `"..."` | `-7.8125` |
| 3 | `"j"` | `-7.875` |
| 4 | `","` | `-7.96875` |
| 5 | `" S"` | `-7.96875` |
| 6 | `" D"` | `-8.0` |
| 7 | `" B"` | `-8.25` |
| 8 | `"-"` | `-8.3125` |
| 9 | `":"` | `-8.3125` |
| 10 | `"("` | `-8.375` |

The expected visible token (`blue` or a prefix of it) is not present in the
returned top-10.

## Follow-Up Decode Attempt

After the first-token proof, a narrow cached Chat decode with `max_tokens=8`
was attempted. It was rejected before generation by the Metal working-set guard:

`Metal GPU working set too full (103% of 107.5GB cap)`

Health after the first-token proof:

- Active memory: `113631.5 MB`.
- Peak memory: `113861.3 MB`.
- L2 block tokens on disk: `10`.
- L2 SSM companion tokens on disk: `10`.

## Classification

The metadata fix worked for the vMLX shape-repair path: the runtime no longer
had to infer hundreds of per-module quant settings from tensor shapes.

The output failure remains before API/parser/cache assembly. The model/runtime
logits path itself selects token `220` as top-1 on the first token. Since the
protected active path is 8-bit and previous linear/head one-variable controls
also failed, the remaining likely boundary is either:

- routed `switch_mlp` 2-bit sensitivity in the artifact, especially gate/up/down
  and expert output contribution; or
- a vMLX affine matmul/dequant/upcast bug that affects this N2 routed path even
  with correct metadata.

No source/high-bit reference was loaded in this pass; this proof only classifies
the manifest-fix artifact's own first-token distribution.

## Cleanup

The vMLX server on port `8137` was stopped after the probe. No intentional N2
vMLX probe server remains from this run.
