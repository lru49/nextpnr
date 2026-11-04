
## Full-model B2 r1 re-measurement — harness error (2026-11-04)

- **B2 tier-2 (reference nondeterministic)** result root: `bench/results/20261104T144924Z-fast-pip-delay-source-bypass-full-model-b2-r1-tier2`; only the surviving 817662/817663 pair is valid. The reference reproduces the original r1 FASM payload, router history, route checksum and fast-model Fmax; its raw FASM differs only in the generated version comment.
- Printed fast-model Fmax is reference 30.58 MHz, candidate 30.68 MHz. The requested full-model Fmax is **unavailable**: both sides exited 125 on combinational loops immediately after the full-model marker and produced no report JSON. See `full-model-status.json` (`HARNESS_ERROR_NO_FULL_MODEL_FMAX`). No full-model QoR or performance verdict is drawn from this pair; the tier-2 acceptance is unchanged pending a corrected `--ignore-loops` diagnostic.
