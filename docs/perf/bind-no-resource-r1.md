# Router2 bind/unbind no-resource lookup guard r1

## Status

**ABANDONED — B1 identity/QoR PASS; both timing gates FAIL.** Source `b79d47bb36cc48363a29ebebf32fccaf8566bfa7` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Expected tier 1.

## Build and launch record

The pushed diff is limited to `common/route/router2.cc` (2 insertions/2 deletions). Base ancestry and `git diff --check` passed. Configure completed in 1.00 s; the first build stopped before candidate compilation because the fresh worktree's Xilinx metadata submodule was uninitialized. After initializing the pinned `a4af910cac907f2cbd3a545f26f8e26573e860de` submodule, the Release/IPO xc7z020-only build completed on cores 40-47 in 193.58 s at `-j4`, maximum RSS 2,678,124 KiB, with both C/C++ `ccache` launchers. `--version` reports `perf-accepted-qor-revert-40d3b873`; binary SHA-256 is `39e9f4da2ef24e41c2f346c54986db550229d32fe5a8e22210e313719a4b7c66`.

B1 result dir: `bench/results/20261104T161500Z-bind-no-resource-r1-b1`. The detached driver started both runs at 16:18:19Z on candidate core 10/reference core 11, concurrently with the processed-sinks pair on cores 2/3. All hash guards passed. Reference reproduction precedes candidate comparison.

## B1 verdict

**ABANDON.** Candidate/reference cores **10/11**, actual concurrency **2 pairs = 4 processes**, no reference-only run. Both exited **1** at 16:25:27Z after routing and normal program completion. Reference-reproduction exit **0** passed every stored FASM/report/checksum/history/Fmax check before candidate comparison; comparison exit **0**.

Complete FASM and report bytes matched; parsed report/utilization/Fmax, all 91 router-history rows, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, final iteration 91 and overuse 0 matched. FASM SHA-256 is `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 is `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, and final aclk Fmax is 41.83 MHz for both.

Reference → candidate whole wall **428.45 → 428.34 s (-0.0257%)**; Router2 **312.17 → 312.90 s (+0.2338%)**. Both fail the required 3.0% improvement gate. User time 427.33 → 427.21 s; peak RSS 1,558,016 → 1,558,864 KiB. **No B2, rerun, merge or tag.** Evidence: `comparison.json`, `compare.log`, `reference-reproduction.json`, per-role logs/reports/FASM and `pair.json` in the result dir.

## Hypothesis and scope

`setup_resources()` populates `wire_to_resource` from resource-group destination wires before routing. On the Xilinx benchmark it remains empty, so `getResourceKeyForPip()` necessarily returns `GroupId()` and the resource-accounting paths in `bind_pip_internal` and `unbind_pip_internal` cannot be taken. Guard only those two per-pip resource-key/accounting paths with `!wire_to_resource.empty()`. Architectures with resources retain the original code.

Preserve wire binding/unbinding order, congestion counts, pip checks, resource accounting on nonempty architectures, route traversal, scoring, queue operations and RNG. This is distinct from abandoned `no-resource-fastpath-r1` (`route_arc` conflict checks) and `score-no-resource-r1` (scoring lookup); neither is included or rerun. The profile does not isolate this lookup and promises no gain.

## Kill criterion and validation

Build once from the stated base in Release/IPO mode for xc7z020 only, with both C/C++ launchers set to `ccache`, on cores 40-47. Verify ancestry, the minimal tracked diff, `git diff --check`, generated version and binary SHA-256.

Run one detached B1 pair on candidate core 10 and verified-reference core 11, seed 1, one thread, 100 MHz and a 1920 s cap, concurrently with the independent processed-sinks pair on cores 2/3. Reference reproduction precedes candidate metrics and must match complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require expected timing-only exit 1 after normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, plus at least 3.0% improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag.
