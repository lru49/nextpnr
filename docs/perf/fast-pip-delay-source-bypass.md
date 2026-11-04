# Fast-mode pip-delay source-RC bypass

Status: tier-2 acceptance gates passed; no tier-1 claim for the whole candidate.

## Change

`perf/l2-fast-pip-delay-source-bypass`, source commit `cbfc8d7a`, based on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. The only source change is in `himbaechel/arch.h` (`Arch::getPipDelay`): fast-delay mode already multiplies the source-side RC term by zero, so the candidate skips the source wire/node lookups and that zero-valued arithmetic. The non-fast path and destination/intrinsic arithmetic are unchanged. No cache, delay table, or persistent state was added.

The xc7z020-only Release/IPO build passed 63/63 steps and reference-chipdb `--test`. Candidate binary SHA-256: `47fc3bfbc2eab4f347f5d6e299000d84fd31eb376f58c268a13142a52b169776`. Original reference binary SHA-256: `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`.

All pairs used `xc7z020clg484-1`, router2, HeAP, seed 1, one thread, and 100 MHz. B1 uses `top_bringup` with a 1920 s cap; B2 uses `zta_v1` with an 8460 s cap. Candidate/reference ran together on neighboring cores. Reference artifacts were checked before candidate metrics.

## B1 identity and timing

Both pairs had the expected timing-only exit 1 with normal completion. Complete FASM, report bytes, parsed reports, router histories, checksums, Fmax, utilization, and final overuse were identical.

| Pair | Wall (s), reference -> candidate | Wall delta | Router2 (s), reference -> candidate | Router2 delta | Peak RSS delta | Iterations | FASM |
|---|---|---:|---|---:|---:|---:|---|
| r1 | 399.17 -> 373.61 | -6.4033% | 289.52 -> 264.34 | -8.6972% | +0.0442% | 91 -> 91 | identical |
| r2 | 419.49 -> 395.56 | -5.7045% | 306.95 -> 283.47 | -7.6495% | +0.0434% | 91 -> 91 | identical |

Evidence: `20261104T114735Z-fast-pip-delay-source-bypass-b1` and `20261104T122843Z-fast-pip-delay-source-bypass-b1-r2`.

## B2 tier-2 (reference nondeterministic)

The reference-only reproduction root `20261104T115910Z-b2-reference-repro` produced different FASM hashes in its two runs, so no tier-1/reference-deterministic claim is made. Both paired runs below completed normally with exit 0 and converged. Utilization was equal in all 52 fields, final overused/overuse/archfail was 0/0/0, and Fmax was equal or better.

| Pair | Wall (s), reference -> candidate | Wall delta | Router2 (s), reference -> candidate | Router2 delta | Peak RSS delta | Final Fmax (MHz), reference -> candidate | Iterations | FASM |
|---|---|---:|---|---:|---:|---|---:|---|
| r1 | 2272.83 -> 2188.52 | -3.7095% | 1287.45 -> 1206.45 | -6.2915% | +0.7846% | 30.582908630371094 -> 30.67860984802246 | 17 -> 16 | different |
| r2 | 2234.09 -> 2133.10 | -4.5204% | 1253.49 -> 1160.15 | -7.4464% | +0.4501% | 30.67860984802246 -> 30.67860984802246 | 16 -> 16 | identical |

The r1 reference reproduced the stored reference exactly. Its candidate FASM difference, using GNU `diff -U0`, is 22,114 all / 22,113 payload hunks, 15,537 non-comment additions, 15,351 non-comment deletions, and 70,203 changed non-comment lines (6,690,135 diff bytes). The r2 complete FASM and report are byte-identical between the pair, although that reference does not reproduce the stored reference.

Evidence: `20261104T124335Z-fast-pip-delay-source-bypass-b2-r1-tier2` and `20261104T124445Z-fast-pip-delay-source-bypass-b2-r2-tier2`; each contains `reference-reproduction.json`, `comparison.json`, `tier2-verdict.json`, and `fasm-diff.json`.

These are nextpnr comparisons only; no bitstream or board-level test was performed.
