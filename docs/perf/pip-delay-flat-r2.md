# Flat fast-mode pip-delay table

## Status

**ABANDONED on 2026-11-04.** Both B1 pairs passed, but B2 r1 failed complete-FASM/report/history/checksum identity. B2 r2's reference independently failed reproduction, so no r2 candidate verdict was drawn. B2 r3 was subsequently stopped under block 31 because the valid r1 identity failure had already mandated abandonment; r3 has no verdict. The candidate is not accepted, merged or tagged; its measured timing does not override the identity failure.

## Base and branch

- Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`
- Branch: `perf/pip-delay-flat-r2`
- Reference binary SHA-256: `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`

## Profile evidence and hypothesis

The old-base B1 profile attributes 10.78% whole-run and 12.80% Router2 self time to `Arch::getPipDelay`. During routing, `fast_pip_delays` makes the result independent of the mutable `drive_res` and `load_cap` dictionaries. The remaining chipdb timing data and normalized endpoint/node timing data are immutable.

Precompute the scalar delay of every pip once on entry to fast-delay routing, using the original arithmetic, and store it in a flat `std::vector<delay_t>` indexed by the base's existing `pip_tile_offsets`. A private helper retains the original uncached computation for non-fast-mode callers and for table initialization. Fast-mode `getPipDelay` then constructs the same `DelayQuad` from the cached scalar. The original implementation always returns `DelayQuad(total_delay)` or `DelayQuad(100)`, so no quadrant information is discarded.

This is a new implementation, not a rerun of `44471597`: it reuses the base's pip offsets and direct nested tile/pip initialization rather than adding a second offset table and iterating `getPips()`. Only `himbaechel/arch.h` and `himbaechel/arch.cc` are in scope. Route ordering, delay arithmetic, availability, bindings, costs, RNG and output generation remain unchanged.

## B1 kill criterion

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache` and the reference version string. Run exactly one detached B1 pair on candidate core 4/reference core 5 with seed 1, one thread, 100 MHz and a 1920 s cap.

Before candidate metrics are inspected, the reference must reproduce:

- complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`;
- report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`;
- checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`;
- iteration 91, final route triple 0/0/0 and Fmax 41.83 MHz.

Abandon on a crash, timeout, non-timing error, non-convergence, any complete-FASM/report/history/checksum/QoR mismatch, or less than 3.0% paired improvement in either Router2 time or whole wall time. No second B1 pair or B2 run follows a failed screen. If it passes, acceptance still requires a second B1 pair and two B2 pairs under the existing tier gates.

## Implementation and build

Candidate source `d06f5c44233dbbfa825ec189839a428b72ab5a91` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. It adds 18 lines in the two published files: the original arithmetic is retained in `compute_pip_delay`, fast-mode calls read a flat `std::vector<delay_t>`, and `set_fast_pip_delays` initializes it through the existing `pip_tile_offsets` and direct nested tile/pip loops. The source branch is pushed and its worktree is clean.

The Release/IPO Xilinx-only build completed in 214.11 s with peak RSS 2,677,976 KiB. Both C and C++ compiler launchers are `ccache`; actual candidate/reference compile commands both use `-O3 -g` and `-flto=auto`. Candidate binary SHA-256 is `e18d4e3dce93034e5695762505323e5dd9ed0154bb2abe4d721d2919937f2d63`. Version/help smoke checks passed with the reference version string.

## B1 screen r1: pass

Result directory: `bench/results/20261104T080300Z-pip-delay-flat-r2-b1`. The reference reproduced every stored FASM/report/checksum/history/Fmax value before candidate metrics were inspected. Both runs then completed normally with the expected timing-only exit 1. Complete FASM, report bytes, parsed reports, router history and checksums are identical; iteration 91, final route 0/0/0 and Fmax 41.83 MHz are unchanged.

| Metric | Reference | Candidate | Change |
| --- | ---: | ---: | ---: |
| Whole wall | 392.17 s | 358.65 s | **-8.5473%** |
| Router2 | 285.75 s | 250.08 s | **-12.4829%** |
| User time | 391.03 s | 357.49 s | -8.5773% |
| Analytic iteration sum | 42.69 s | 42.88 s | +0.4451% |
| HeAP | 45.29 s | 45.48 s | +0.4195% |
| SA | 54.38 s | 54.68 s | +0.5517% |
| Peak RSS | 1,558,076 KiB | 1,704,204 KiB | +9.3787% |

This passes the B1 screen but is not final acceptance. One confirmation B1 pair and two B2 pairs remain mandatory before any integration branch merge or tag.

## B1 confirmation r2: pass

Result directory: `bench/results/20261104T081400Z-pip-delay-flat-r2-b1-r2`. Reference reproduction again passed before candidate metrics were inspected. Both runs completed normally with the expected timing-only exit 1; comparison exit 0. Complete FASM, report bytes, parsed reports, full router history and checksums are identical, with iteration 91, final route 0/0/0 and Fmax 41.83 MHz.

| Metric | Reference | Candidate | Change |
| --- | ---: | ---: | ---: |
| Whole wall | 392.05 s | 359.18 s | **-8.3841%** |
| Router2 | 285.61 s | 250.73 s | **-12.2125%** |
| User time | 390.93 s | 358.03 s | -8.4158% |
| Peak RSS | 1,558,080 KiB | 1,704,180 KiB | +9.3769% |

Both B1 pairs therefore pass the pre-published gates. This is not acceptance.

## B2 repeats: identity failure; abandon

Two pairs started at 2026-11-04T08:30:35Z in `bench/results/20261104T082700Z-pip-delay-flat-r2-b2`: r1 used candidate core 0/reference core 1 and r2 used candidate core 2/reference core 3. All four processes completed normally with exit 0.

In r1, the reference reproduced every stored result before candidate metrics were inspected. The candidate then failed the mandatory identity gates: complete FASM, report bytes, parsed reports, full router history and checksums all differed. The reference reached iteration 17/Fmax 30.58 MHz/third checksum `0xeae85389`; the candidate reached iteration 16/Fmax 30.68 MHz/third checksum `0x6ff14c06`. Its measured wall change was -5.2819% and Router2 change -9.4364%, but these are **not a valid speedup** because the routed result is different.

In r2, the reference itself did not reproduce the stored result: FASM SHA-256 was `18d00b1ccde28e327fd7fc41024487dc501e4d36a4f509a7dbb178f5e99e5b4a`, report SHA-256 `62fd9c1b31399315491fd7ee694c51f868d108f95001d29eb659204a78fe9058`, third checksum `0x0ab634fe`, final iteration 15 and Fmax 30.68 MHz. The driver therefore skipped candidate comparison (`compare=100`); no r2 candidate metrics or acceptance claim are used.

One valid reference/candidate comparison is sufficient to reject this candidate on identity. `perf/pip-delay-flat-r2` is abandoned with no merge, tag or further acceptance run.

## B2 r3: stopped, no verdict

Block 29 item 3 incorrectly ordered a replacement after the valid r1 identity failure had already required abandonment. Block 31 superseded that launch and the r3 processes were stopped by pid at approximately 2026-11-04T10:00Z. Driver 52637, candidate 52662, reference 52659 and wrappers 52657/52660 were terminated; the driver log records candidate/reference exits 143 at 10:00:15Z after about 29 minutes on cores 2/3.

No reference-reproduction, comparison or completed timing artifact exists for r3. Its reference reproduction, identity checks and performance are therefore unavailable, and it supplies no candidate verdict. The candidate remains abandoned with no merge or tag.

The r2 reference-side nondeterminism remains a standing finding: no further pair that depends on the implicated B2 reference may start until that reference reproduces twice in separate pairs.
