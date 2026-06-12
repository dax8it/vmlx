# N2 JANG_1L 2-bit Prune15 First-Token Proof - 2026-06-11

Artifact:

`/Volumes/EricsLLMDrive/jangq-ai/Nex-N2-Pro-JANG_1L-2bit-prune15-20260611`

## Result

Status: fail, output-quality blocked.

The 15% pruned affine JANG artifact loads and no longer collapses to token
`220` on the first token. However, the first token is still not semantically
sane for the prompt: the model emits `!` for `Reply with exactly: blue cat`.
The requested top-k logprobs were not populated by this runtime path
(`logprob=null`, `top_logprobs=[]`), so this proof cannot clear logits/top-k
quality.

Short decode was not run because the first-token gate was not sane.

## Artifact Metadata

JANG validation source:

`/Users/eric/jang/build/n2-pro-jang1l-2bit-prune15-validation-20260611.json`

Observed facts:

- Size: `96G`.
- Experts per layer: `512 -> 435`.
- Dropped experts per layer: `77` (`15.039%`).
- Prune method: `router_row_l2_topk_per_layer`.
- Evidence level: `router_weight_proxy`.
- Sliced tensors: `600`.
- `config.text_config.num_experts=435`.
- `config.jang_expert_pruning.num_experts=435`.
- `jang_config.expert_pruning.num_experts=435`.
- `config.json["quantization"]` module override count: `647`.
- EOS: top-level and text config both `248046`.
- mRoPE fields preserved.

## Runtime Route

- Source vMLX server on port `8138`.
- `VMLINUX_FORCE_TQ_AUTO=1`.
- `--reasoning-parser none`.
- `--default-enable-thinking false`.
- Paged cache + block disk cache enabled.
- Route: `affine_qwen_hybrid_jang_text_only`.
- `mllm=False`.
- No `quant_shape_inference: patched ...` warning emitted during startup.
- Runtime cache: 15 attention KV layers with live TurboQuant KV and 45 SSM
  companion layers.
- Metal working-set baseline max: `107.5GB`.

## First-Token Request

Prompt:

`Reply with exactly: blue cat`

Request:

- `max_tokens=1`
- `temperature=0`
- `top_p=1`
- `logprobs=true`
- `top_logprobs=10`
- `enable_thinking=false`

Response:

- HTTP 200.
- `message.content="!"`.
- `completion_tokens=1`.
- `finish_reason=length`.
- `logprobs.content[0].token="!"`.
- `logprobs.content[0].logprob=null`.
- `logprobs.content[0].top_logprobs=[]`.
- Speed: 1 token in 37.97s.

## Cache / Memory

Health after first-token proof:

- Active memory: `98004.5 MB`.
- Peak memory: `98235.1 MB`.
- L2 block tokens on disk: `10`.
- L2 SSM companion tokens on disk: `10`.
- SSM companion entries: `1`.

This is materially below the prior 112G artifact's 113GB active memory and did
not trigger the Metal working-set guard during the first-token proof.

## Classification

This artifact is not coherent, but it changed the failure mode:

- unpruned/manifestfix affine artifacts: first token was space token `220`;
- prune15 artifact: first token is visible punctuation `!`.

That suggests pruning/shape metadata changed the logits path enough to escape
the exact repeated-space collapse, but the router-row-L2 proxy pruned artifact
still fails the semantic first-token gate.

Current classification:

- not API/parser/cache assembly;
- not the old quant-shape metadata repair issue;
- not a steady healthy model;
- likely prune-quality/routed-expert selection issue, or still a full routed
  aggregation/runtime issue beyond the sampled affine primitive parity probe.

Next JANG-side action should not conclude "15% pruning impossible" from this.
The shared lane caveat applies: this is router-row-L2 proxy pruning, not
activation-guided pruning.

## Cleanup

The vMLX server on port `8138` was stopped after the first-token proof. No
intentional N2 vMLX probe server remains from this run.
