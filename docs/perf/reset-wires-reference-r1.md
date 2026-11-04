# Router2 dirty-wire reset access reuse

## Status

**B1 RUNNING on 2026-11-04.** Candidate `perf/reset-wires-reference-r1` source `1047f166607a833849e9aa1a05d397219e1cb820` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Candidate binary SHA-256 is `5faf3b55cd9de970d3d5316b6980b65d4cc5cb67fe22f5ce4f11e300bdc86c2a`. No B2 run is permitted while the B2 reference remains nondeterministic.

## Build and launch

The Release/IPO xc7z020-only build completed on cores 40-63 in 236.86 s with both C/C++ compiler launchers set to `ccache`; generated `common/version.h` and `--version` report `perf-accepted-qor-revert-40d3b873`. B1 started at `2026-11-04T12:12:51Z` in `bench/results/20261104T120700Z-reset-wires-reference-r1-b1`, with candidate nextpnr 13740 on core 10 and reference nextpnr 13737 on core 11 under detached driver 13712. Actual concurrency is one pair (two processes), seed 1, one thread, 100 MHz and a 1920 s cap.

## Hypothesis and scope

`reset_wires` resets six fields for every dirty wire through six independent `flat_wires[w]` accesses. Reuse one `PerWireData &wd = flat_wires[w]` per iteration and perform the same six assignments through that reference. Change only `common/route/router2.cc`; preserve dirty-list order and clearing, visit state, pip bindings, costs, queue behavior and route decisions. Add no table and do not alter scoring or pip traversal. Expected tier: 2; the stale profile does not isolate this function and supplies no promised gain.

## Kill criterion and acceptance gates

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using version string `perf-accepted-qor-revert-40d3b873`. Run one detached B1 pair against the verified reference on an allowed core pair, seed 1, one thread, 100 MHz and a 1920 s cap. Record actual concurrency.

Before candidate metrics are read, the reference must reproduce complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal completion with expected timing-only exit 1, byte-identical complete FASM, identical report bytes and parsed report, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.
