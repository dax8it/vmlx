# vMLX 1.5.58 Checkpoint

This is a signed and notarized checkpoint release. It is intended to make the
current vMLX Python engine and MLXStudio app fixes available while several
larger model-family rows remain under active proof.

Highlights:

- Refreshes the bundled vMLX Python engine to `1.5.58`.
- Bundles the current JANG runtime package surface from `jang==2.5.31`.
- Keeps current parser/API fixes for Qwen/Qwen-coder Responses tool-call
  streaming, missing required argument fail-closed behavior, function-call
  argument deltas/done events, output-index validation, and tool-result
  continuation.
- Keeps the current Gemma4/JANG/QAT/MXFP runtime and bundled app parity fixes,
  including Gemma4 unified runtime registration and JANG/JANGTQ loader parity.
- Keeps current cache/runtime packaging parity for JANG/JANGTQ/TurboQuant
  kernel imports and block-disk/cache surfaces included in the bundled engine.

Verification:

- `panel/scripts/verify-bundled-python.sh` passed before packaging.
- `panel/scripts/build-release-dmgs.sh all` produced fresh `1.5.58` Sequoia and
  Tahoe DMGs under explicit checkpoint override.
- Apple notarization accepted both DMGs.
- `panel/scripts/verify-release-dmgs.sh` passed for both DMGs: disk image
  verification, Developer ID signature validation, stapled ticket validation,
  and Gatekeeper assessment all passed.

Artifacts:

- `vMLX-1.5.58-sequoia-arm64.dmg`
  - SHA256: `71925fa21857a631c7fdddfd14b217cef8e076a3ce88fb82439672a0196bd7f4`
  - Notary ID: `d94933ed-ba35-4d45-a4f5-5e9c8ce8ff1a`
- `vMLX-1.5.58-tahoe-arm64.dmg`
  - SHA256: `ffa671547b0de037d9e5257589f29d8e29c5cebb7358c127ed0a90b6925040dc`
  - Notary ID: `8615b942-0b30-4aea-a805-f62c49895ca2`

Known checkpoint boundary:

- This is not a full production-clear model matrix release.
- Open rows remain in the checkpoint override manifest for larger live
  model-family proof, selected UI/media rows, and deployed/tunnel parity
  surfaces.
- PyPI publishing is separate and remains gated on valid PyPI publishing
  credentials or trusted-publishing configuration.

Credit: thanks to @Hornsan1 for reported runtime/API issues that informed this
checkpoint hardening pass.
