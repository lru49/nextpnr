# Router2 shared source-use lookup

## Status

**ABANDONED on 2026-11-04 at the B1 timing gate.** Candidate `perf/route-cost-source-uses-r1` at `7678ff06064760716378b9e52e59b56bdd0dce3d` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Candidate binary SHA-256 is `aeb44efc89c1d95236361305b6991cb283ce82c43a6752b103b104d21d79c66b`. Identity/QoR passed, but both timing gains were below 3.0%. No B2, rerun, merge or tag is permitted.

## Hypothesis and scope

For each expanded next wire, `score_wire_for_arc` and `get_togo_cost` independently probe `nd.wires` to obtain the same source-use count. The forward expansion already has a binding lookup that can be reused. The old-base B1 profile attributes 1.56% whole-run time to `dict<WireId,pair<PipId,int>>`; this is an upper-bound clue, not a promised gain.

Change only `common/route/router2.cc`. Compute the existing source-use result once per expanded next wire, reuse the forward binding lookup, add the equivalent single lookup in the backward expansion, and pass the count to both scoring expressions. Preserve the bound-wire overuse adjustment, exact costs and floating-point evaluation order, pip traversal, queue order and bindings. Add no design-sized table and do not alter the excluded `dict<WireId,int>`. Expected tier: 1.

## Kill criterion and acceptance gates

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using version string `perf-accepted-qor-revert-40d3b873`. Per block 35, after the build start one detached B1 pair on candidate core 2 and the verified reference on core 3, seed 1, one thread, 100 MHz and a 1920 s cap.

The Release/IPO xc7z020-only build completed successfully on cores 40-63 with both compiler launchers set to `ccache`. Generated `common/version.h` and `--version` both report the reference version string; the source worktree and `git diff --check` are clean. The candidate branch is pushed.

Pair `20261104T114700Z-route-cost-source-uses-r1-b1` started at 11:52:05Z after all input, chipdb and binary hash guards passed. Detached driver 8634 is parented to PID 1; candidate nextpnr 8661 has affinity core 2 and reference nextpnr 8658 has affinity core 3. Actual launch concurrency is 1 pair = 2 processes because the visit-data pair had already completed. Reference reproduction is scripted before candidate comparison/metrics.

Before candidate metrics are read, the reference must reproduce:

- complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`;
- report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`;
- checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`;
- history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal completion with the expected timing-only exit 1, byte-identical complete FASM, identical report bytes and parsed report, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.

## B1 result

Both sides completed normally with expected timing-only exits 1/1. The reference reproduced every stored identity value before candidate metrics were read. Candidate complete FASM and report SHA-256 matched the stored values; report bytes/parsed report, full router history, checksums, iteration 91, final overuse zero and Fmax 41.83 MHz all matched. Comparison exit was 0.

Reference -> candidate whole wall was 403.60 -> 401.73 s (**-0.4633%**), and Router2 was 291.18 -> 289.20 s (**-0.6800%**). Both improvements are below the required 3.0%. **Decision: ABANDONED on timing; no B2, rerun, merge or accepted tag.** Raw artifacts remain in `bench/results/20261104T114700Z-route-cost-source-uses-r1-b1`.
