# MiMo V2.5 corrected artifact requirement - 2026-06-08

Status: release-blocking.

Scope: active Python vMLX/MLXStudio worktree only. Do not use parser repair, JSON repair, prompt folding, cache disabling, or broad release gates to clear this blocker.

## Current local artifacts

Kept local MiMo artifacts:

- `/Users/eric/.mlxstudio/models/JANGQ-AI/MiMo-V2.5-JANGTQ_2`
- `/Users/eric/.mlxstudio/models/JANGQ-AI/MiMo-V2.5-JANG_2L`

## Current proof

Fresh live proof:

- `build/current-mimo-v25-jangtq2-exactness-rerun-20260608/result.json`

Classifier:

- `build/current-mimo-v2-no-source-exactness-classifier-after-jangtq2-live-refresh-20260608.json`
- classification: `jangtq2_compact_hyphen_decode_quality_open_not_cache_parser_template_tokenizer`
- secondary: `jang2l_json_sentinel_semantic_mismatch_open`

Audit:

- `build/current-mimo-v2-jang2l-current-audit-after-live-refresh-20260608.json`
- blockers:
  - `mimo_jangtq2_artifact_exactness_blocked`
  - `mimo_jang2l_live_media_l2_missing`
  - `mimo_jang2l_media_capability_memory_gated`
  - `mimo_jang2l_tight_memory_prompt_budget_blocked`

## Exact live failures

`MiMo-V2.5-JANGTQ_2` loaded and served successfully. Server/tool structures were valid, but exact values were wrong.

Observed failures:

- plain completion: `blue-cat` -> `blue`
- plain completion: `B7-CAT-09` -> `B7 CAT-09`
- chat: `blue-cat` -> `blue grass`
- chat: `B7-CAT-09` -> `B7 CAT-09`
- JSON: `{"status":"ok","value":"blue-cat","count":3}` -> `{"status":"ok","value":"blue","count":3}`
- JSON: `{"status":"ok","value":"B7-CAT-09","count":3}` -> `{"status":"ok","value":"B7CAT-09","count":3}`
- tool call: `{"value":"blue-cat"}` -> `{"value":"blue"}`
- tool call: `{"value":"B7-CAT-09"}` -> `{"value":"B7CAT-09"}`

## What is excluded

The current evidence excludes these as primary fixes:

- parser argument rewrite
- JSON repair or semantic repair
- prompt-only formatting/folding
- prefix cache
- paged cache
- block-disk L2
- generic TurboQuant KV
- hidden stochastic sampling
- chat-template/tokenizer literal loss
- continuous batching as the primary exactness cause

The MiMo server reported native `mixed_swa_kv_v1` cache. Generic TurboQuant KV was correctly skipped because flat generic TQ-KV would violate MiMo asymmetric full/SWA cache semantics.

## Required corrected artifact direction

Do not upload another all-routed-2bit JANGTQ2 unless it passes the exactness probe below.

Recommended corrected profiles to try first:

- JANGTQ with routed `gate=3/up=2/down=3`
- JANGTQ with routed `gate=3/up=3/down=3`

Rationale: the current all-routed-2bit JANGTQ2 is fast enough and structurally loads, but corrupts compact hyphen/literal values across raw completion, chat, JSON, and tool arguments. Existing classifier evidence marks this as artifact/logit quality unless a runtime decode proof contradicts it.

For classic `JANG_2L`, exact JSON sentinel also remains open. A corrected JANG_2L must preserve both semantic value and count, and must avoid the current tight-memory prompt-budget failures for tool/sentinel rows.

## Acceptance proof after replacing artifact

After replacing the local artifact, do not run broad release gates first. Run the narrow exactness proof.

Start only the target MiMo artifact through active Python server, then run:

```bash
.venv/bin/python tests/cross_matrix/run_mimo_v2_exactness_variant_probe.py \
  --base-url http://127.0.0.1:8897 \
  --model mimo-v2.5-jangtq2 \
  --out build/current-mimo-v25-jangtq2-exactness-rerun-20260608/result.json \
  --timeout 240 \
  --max-tokens 48
```

Required result:

- `status=pass`
- all eight labels pass:
  - `plain_exact_blue_cat`
  - `plain_exact_sentinel`
  - `plain_exact_chat_blue_cat`
  - `plain_exact_chat_sentinel`
  - `json_blue_cat`
  - `json_sentinel`
  - `tool_blue_cat`
  - `tool_sentinel_json_call`

Then refresh classifier:

```bash
.venv/bin/python tests/cross_matrix/run_mimo_v2_no_source_exactness_classifier.py \
  --jangtq2-literal-variants build/current-mimo-v25-jangtq2-exactness-rerun-20260608/result.json \
  --out build/current-mimo-v2-no-source-exactness-classifier-after-jangtq2-live-refresh-20260608.json
```

Required result:

- classifier must no longer report `jangtq2_compact_hyphen_decode_quality_open_not_cache_parser_template_tokenizer`
- `model_upload_action_required` must be false or superseded by stronger runtime proof

Only after those pass should release checklist/manifest/current suite be regenerated.
