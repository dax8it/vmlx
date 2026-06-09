# vMLX / MLXStudio release blocker ledger - 2026-06-09

Scope: active Python engine and Electron/panel app in `/Users/eric/mlx/vllm-mlx-finite-launch-guard`.

Hard boundary: no fake fixes, no forced sampling/parser/cache behavior to hide runtime/model bugs, and no signing/notarization/tag/download release until the runtime, cache, parser, UI, and installed-app rows are green or Eric explicitly overrides.

Reporter credit: include GitHub `@Hornsan1` in next release notes/changelog/public acknowledgement for reported runtime/model/UI/API issues.

## Active blockers and proof requirements

1. Responses streaming tool arguments

- Need raw SSE local direct vs gateway vs tunnel comparison on the same model and same request.
- Do not fix by disabling reasoning or reducing reasoning effort.
- Current synthetic/local engine paths can emit non-empty `response.function_call_arguments.delta` and `response.function_call_arguments.done`; next proof must use the reported model/request and deployed tunnel path.
- Specific issue to trace: streaming code near line `13592` checks `if tc_args:`; reported failure means `tc_args` is empty. Trace why `_parse_tool_calls_with_parser` returns empty args when reasoning is on, and trace how the streaming loop accumulates tool-call deltas.
- Gateway/tunnel/port/wake/sleep/session routing must be checked separately from engine parsing. A Cloudflare/gateway model miss or stale sleeping backend is not the same as a local parser bug.
- No executable `{}` argument payload should be emitted for required tool calls when required args are actually missing; fail closed instead.

2. MiniMax random Chinese / visible planning

- Not cleared.
- Do not clamp sampling, inject sentinels, or hide visible planning as a "fix."
- Need isolate cache-on vs cache-off, TurboQuant KV vs none, L2 vs no L2, parser/template boundary, request/session reuse, and model-owned `generation_config.json`.
- Random Chinese or random-language output is likely cache continuation, prompt-contract contamination, parser/tool memory injection, L2 disk stale restore, or logit/decode-loop corruption until proven otherwise.
- Read prior docs/artifacts for similar cross-model cache/language/planning leaks before touching defaults.

3. MiMo V2.5

- JANGTQ_2 speed/cache is partially good, but release proof is not complete.
- JANG_2L, tool/JSON/loop, media, L2 restore, UI parity, and installed-app proof remain open.
- Do not run source-vs-quant comparisons if RAM-blocked unless Eric explicitly allows.
- Exactness failures must not be papered over by parser repair, sampling clamps, cache disabling, or JSON repair.
- Confirm runtime dynamically reads artifact config for bit size, grouped experts, stacked vs legacy layout, JANG/JANGTQ/MXFP metadata, and model-owned generation defaults.

4. N2 / Qwen-family JANG and JANGTQ

- Need live tool, reasoning, parser, cache, Responses, streaming args, and UI proof.
- Especially cover `gdn_sink`, MTP, hybrid SSM/cache, paged cache, TurboQuant KV encode/decode, block-disk L2, and no parser/reasoning leaks.
- JANG_1L should fit with careful RAM handling; treat current blocker as careful live-proof scheduling and memory discipline, not permanent infeasibility.
- Do not launch N2/JANG_1L below the preflight headroom gate after Metal OOM evidence. If preflight says `do_not_launch`, clear RAM or schedule later instead of forcing it.
- N2 JANGTQ_2 proof does not clear N2 JANG_1L.

5. DSV4

- Memory-unit harness fix is landed by the other agent.
- Still needs live default-cache tool-loop proof when memory gate allows.
- Must use native SWA/CSA/HCA cache behavior, not a generic fake KV substitute.
- Do not claim DSV4 release clearance from load, health, narrow cache proof, or memory-label cleanup alone.

6. Gemma 4 QAT/native MXFP4/MXFP8/JANG

- Source startup and several source smokes exist, but full MXFP4/MXFP8/JANG_4M/QAT media/cache/UI/installed-app/tunnel matrix remains open.
- Need E2B, E4B, 12B, 26B, and 31B downloaded/present from JANGQ HF repos for full multi-turn, tool-call, parser, cache-reuse, coherency, and media proof.
- Incoherent multilingual/token-soup output from older app versions is likely real runtime/load/decode corruption until proven otherwise. Known class: Gemma4 MoE/native-MXFP sidecar hydration or shared-KV/load compatibility can corrupt generation.
- `ModuleNotFoundError: No module named 'mlx_vlm.models.gemma4_unified'` is a release blocker until the packaged/bundled runtime has the current compatibility alias/shim and installed-app parity is green.
- VLM/image prefill guard recovery and post-error text recovery still need proof. A failed image turn must not poison later text turns in the same chat/session.
- Audio/video claims must be weight-backed and runtime-proven. Token metadata alone is not native audio proof.

7. Step 3.7

- Metadata/runtime route matrix is improved.
- Do not make a fake `has_vision=false` release claim unless the release row is explicitly scoped text-only.
- Tool dialect loops, raw XML-like leaks, tool-result continuation, multi-turn synthesis, and thinking-template mismatch remain model/runtime parser issues to prove per path.
- If Step3p7 VLM runtime is advertised, prove live media or fail closed with honest capability reporting.

8. Structured JSON/XML

- Repair/validation is application and benchmark hygiene, not a substitute for runtime coherence.
- Guided/schema decoding should only be claimed if real runtime support exists.
- JSON/XML repair should report raw parse success, repair-needed status, normalized object, schema validation result, and retry/fail-closed behavior.

9. UI/CLI/API parity

- Parser, reasoning, cache type, prefix cache, paged cache, L2 disk cache, TurboQuant KV, MTP, max output tokens, max context, generation defaults, and media settings must match CLI, API, panel settings, installed-app launch, and persisted session settings.
- Responses endpoints must cover content-delta streaming, function-call argument streaming, final object consistency, previous response/session reuse, cancel/restart, and gateway/tunnel parity.
- Do not clear panel/UI rows from CLI source proof alone.

10. Release/package/sign/notarize

- Developer ID signing and notary profile access are currently usable after the documented keychain unlock/partition-list sequence; `build/current-signed-checkpoint-dmg-readiness-20260609.json` records fresh signing `pass` and notarization `pass`.
- Current release packaging is still blocked on rebuilding current-source DMGs, notarizing/stapling/verifying those current artifacts, and clearing or explicitly scoping the remaining runtime/model/UI/cache rows.
- No signing, notarization, tag, public download update, or release announcement until runtime/model/UI/cache blockers are green or Eric explicitly overrides.
- If a release is forced with known open rows, the release notes must list exact open rows and not imply full clearance.
- Proper release mechanics are documented in `/Users/eric/wiki/infra/apple-notarization.md`; do not invent an alternate path. The canonical keychain is `~/Library/Keychains/vmlx-build.keychain-db`, the Developer ID identity is `Developer ID Application: ShieldStack LLC (55KGF2S5AY)`, and notarization uses the `vmlx-notary` keychain profile.
- If signing returns `errSecInternalComponent`, fix key access with the documented sequence and retry once after the partition-list grant settles:

```sh
security unlock-keychain -p vmlx-release ~/Library/Keychains/vmlx-build.keychain-db
security set-keychain-settings ~/Library/Keychains/vmlx-build.keychain-db
security set-key-partition-list -S apple-tool:,apple:,codesign: -s -k vmlx-release ~/Library/Keychains/vmlx-build.keychain-db
```

- Current Python/Electron DMG flow is the repo script path, not a manual ad-hoc app signing path:

```sh
VMLINUX_PREPACKAGE_READY_MANIFEST_OUT=build/current-release-regression-manifest-pre-dmg-release-build-<scope>.json panel/scripts/build-release-dmgs.sh all
VMLINUX_NOTARY_KEYCHAIN=$HOME/Library/Keychains/vmlx-build.keychain-db panel/scripts/notarize-release-dmgs.sh
panel/scripts/verify-release-dmgs.sh
```

- `panel/scripts/build-release-dmgs.sh` first runs `tests/cross_matrix/run_release_regression_manifest.py --require-prepackage-ready`; if that ledger fails, the script must stop before DMG build. Do not bypass this stop except under an explicit checkpoint-release override that records the open rows in the release notes.
- `panel/scripts/notarize-release-dmgs.sh` submits the final signed Sequoia and Tahoe DMG containers, staples both, regenerates blockmaps, runs `spctl`, and prints final SHA-256 values. `panel/scripts/verify-release-dmgs.sh` is the post-staple verification pass.

## Other-agent integration requirements

- Before release, include all pushed source fixes from the other agent and verify they are present in the active worktree, `origin/main`, bundled Python, and installed app.
- Track issue/fix coverage by architecture: dense, MoE/routed, hybrid SSM, SWA/CSA/HCA, MLLM/VLM, audio/video, JANG, JANGTQ/MXTQ, MXFP4, MXFP8, QAT, native MTP.
- A fix for one family does not clear another family unless the same path is live-proven.
- Do not duplicate old work from `/Users/eric/vmlx`, ADLab, Max2 transport notes, or deprecated Swift lanes.

## Immediate focus order

1. Keep status/docs current so a second agent can pick up without guessing.
2. Fix and prove Responses streaming tool arguments with reasoning enabled on the real reported model/request and direct/gateway/tunnel surfaces.
3. Continue Gemma QAT/native MXFP4/MXFP8/JANG proof after downloads are available, including multi-turn, tools, parser leaks, cache reuse, media, and installed app.
4. Continue MiMo and N2 live runtime proof under memory gates, not source-vs-quant heavy comparisons.
5. Rebuild current-source DMGs only after the chosen checkpoint scope is explicit; then notarize/staple/verify with the documented `vmlx-build.keychain-db`/`vmlx-notary` path.
