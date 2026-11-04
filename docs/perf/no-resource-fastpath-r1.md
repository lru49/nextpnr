# Router2 no-resource fast path

## Status

**ABANDONED on 2026-11-04: B1 identity/QoR passed, but both timing gates failed.** Candidate `perf/no-resource-fastpath-r1` at `8d058f2e9ea919db494d3f374ba0965f60de4a4c` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. No B2, rerun, merge or tag is permitted.

## Hypothesis and scope

The old-base B1 profile attributes 8.30% whole-run self time to `Router2::route_arc`. Xilinx has no routing resource groups, yet both forward and backward pip expansion call `getResourceKeyForPip` for every candidate pip and then discard the default empty `GroupId`. These virtual calls and their branches are unnecessary on that architecture.

Change only `common/route/router2.cc`. After resource setup, snapshot whether `wire_to_resource` is empty and use that invariant to skip only the two hot per-pip resource-conflict blocks when no resources exist. Do not alter or replace `dict<WireId,int>`, resource accounting, scoring arithmetic, pip availability, traversal, queue operations or bindings. Architectures with resources retain the original blocks.

Removing these no-op virtual calls should reduce Router2 and whole-wall time without changing any route decision or output. The profile percentage is an upper bound, not a promised gain.

## Kill criterion and acceptance gates

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using version string `perf-accepted-qor-revert-40d3b873`. Run one detached B1 pair with candidate on core 6 and the verified reference on core 7, seed 1, one thread, 100 MHz and a 1920 s cap.

Before candidate metrics are read, the reference must reproduce:

- complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`;
- report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`;
- checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`;
- history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal routing completion, byte-identical complete FASM, identical report bytes and parsed report, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.

## B1 result

Run `20261104T110900Z-no-resource-fastpath-r1-b1` used candidate core 6 and reference core 7, seed 1, one thread and 100 MHz, with three B2 reference-only runs concurrently (five nextpnr processes total). The reference reproduced every stored identity field before candidate metrics were read. Both sides completed normally with the expected timing-only exit 1.

| Metric | Reference | Candidate | Change |
| --- | ---: | ---: | ---: |
| Whole wall | 420.12 s | 421.38 s | +0.2999% |
| Router2 | 304.97 s | 305.26 s | +0.0951% |
| Whole user time | 418.92 s | 420.30 s | +0.3294% |
| Peak RSS | 1,558,076 KiB | 1,557,888 KiB | -0.0121% |

Complete FASM was byte-identical at `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`; report bytes and parsed report, full router history, checksums, Fmax 41.83 MHz, utilization, iteration 91 and final overuse zero were identical. Despite passing identity/QoR, the candidate was slower on both required timing metrics and is abandoned without B2 or rerun.
