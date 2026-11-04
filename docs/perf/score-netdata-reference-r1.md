# Router2 scoring per-net data reuse

## Status

**ABANDONED on 2026-11-04: B1 identity passed, both timing gates failed.** Candidate `perf/score-netdata-reference-r1` source `0485077c40b331eb03481d28028bfcf7f5bfeafd` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Candidate binary SHA-256 is `85ae79ee1b3b26ce6ee9f18e35803f0c692324287ea55e9653413168df7f79e5`. No B2, rerun, merge or tag is permitted.

## Hypothesis and scope

`route_arc` already resolves `const PerNetData &nd = nets.at(net->udata)`, but `score_wire_for_arc` repeats that checked lookup for every expanded next wire. Pass the existing reference into the scorer and use it in both forward and backward expansion. Change only `common/route/router2.cc`; preserve the source-use lookup, bound-wire/resource checks, exact score arithmetic and evaluation order, pip traversal, queue behavior and route decisions. Add no table. Expected tier: 1; the stale profile does not isolate this access and supplies no promised gain.

## Build

The source branch is pushed at `0485077c40b331eb03481d28028bfcf7f5bfeafd`; `git merge-base --is-ancestor` verifies the reference base and the final source diff contains only `common/route/router2.cc` (4 insertions, 5 deletions). The initial build attempt failed after 96.11 s because the first implementation used the nonexistent `NetData` type; commit `0485077c` corrected it to the existing `PerNetData` type. The corrected incremental build completed in 18.76 s after a 1.02 s configure.

The Release/IPO xc7z020-only Ninja build used cores 40-63 and both C/C++ compiler launchers set to `ccache`. Generated `common/version.h` and `--version` report `perf-accepted-qor-revert-40d3b873`. CMakeCache confirms Release, `USE_IPO=ON`, Himbaechel Xilinx-only, both `ccache` launchers and the required version string. The final source worktree and `git diff --check` are clean. Final binary: `nextpnr-score-netdata-r1/build/perf-score-netdata-reference-r1/nextpnr-himbaechel`, SHA-256 `85ae79ee1b3b26ce6ee9f18e35803f0c692324287ea55e9653413168df7f79e5`.

## B1 launch

B1 started at `2026-11-04T12:44:52Z` in `bench/results/20261104T124100Z-score-netdata-reference-r1-b1` under detached driver 23461 (PPID 1). Candidate nextpnr 23536 is pinned to core 2 and reference nextpnr 23533 to core 3, with affinity verified by `taskset`. Both use seed 1, one thread, 100 MHz and a 1920 s cap. Actual launch concurrency is one candidate/reference pair plus two reference-only B2 diagnostic runs (four nextpnr processes). Reference reproduction is scripted to complete before candidate comparison or metrics.

## Kill criterion and acceptance gates

Run one detached B1 pair against the verified reference, candidate on core 2 and reference on core 3, seed 1, one thread, 100 MHz and a 1920 s cap. Record actual concurrency and guard input, candidate binary, reference binary and chipdb hashes before launch.

Before candidate metrics are read, the reference must reproduce expected timing-only exit 1, normal routing/program completion, complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal completion with expected timing-only exit 1, byte-identical complete FASM, identical report bytes and parsed report, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.

## 2026-11-04 B1 result: ABANDONED

Reference reproduction passed before candidate comparison: expected timing-only exit 1, normal completion, complete FASM/report byte identity, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz. Candidate also exited 1 normally; comparison exited 0. Complete FASM/report bytes, parsed report, history, checksums, Fmax, utilization and final overuse zero were identical at the declared concurrency of 1 pair plus 2 reference-only diagnostics (4 processes).

Whole wall: **428.21 -> 425.45 s (-0.6445%)**. Router2: **310.07 -> 308.15 s (-0.6192%)**. Peak RSS: **1,548,556 -> 1,558,832 KiB (+0.6636%)**. Both timing gates failed. **ABANDONED: no B2, rerun, merge or tag.**
