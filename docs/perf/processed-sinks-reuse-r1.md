# Router2 processed-sinks scratch reuse r1

## Status

**ABANDONED — B1 identity/QoR PASS; both timing gates FAIL.** Source `9a05fdeb192b1f8cd81fe6947426839b723f02f5` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Expected tier 1.

## Build and launch record

The pushed diff is limited to `common/kernel/hashlib.h` and `common/route/router2.cc` (6 insertions/1 deletion). Base ancestry and `git diff --check` passed. Configure completed in 0.99 s; the first build stopped before candidate compilation because the fresh worktree's Xilinx metadata submodule was uninitialized. After initializing the pinned `a4af910cac907f2cbd3a545f26f8e26573e860de` submodule, the Release/IPO xc7z020-only build completed on cores 40-47 in 193.85 s at `-j4`, maximum RSS 2,677,800 KiB, with both C/C++ `ccache` launchers. `--version` reports `perf-accepted-qor-revert-40d3b873`; binary SHA-256 is `e1cdde030b6657772bc4292aef842f1cef29db9659ced6a8c35b56a2c2c73035`.

B1 result dir: `bench/results/20261104T161500Z-processed-sinks-reuse-r1-b1`. The detached driver started both runs at 16:18:19Z on candidate core 2/reference core 3, concurrently with the bind-no-resource pair on cores 10/11. All hash guards passed. Reference reproduction precedes candidate comparison.

## B1 verdict

**ABANDON.** Candidate/reference cores **2/3**, actual concurrency **2 pairs = 4 processes**, no reference-only run. Candidate/reference exited **1/1** at 16:25:31/16:25:35Z after routing and normal program completion. Reference-reproduction exit **0** passed every stored FASM/report/checksum/history/Fmax check before candidate comparison; comparison exit **0**.

Complete FASM and report bytes matched; parsed report/utilization/Fmax, all 91 router-history rows, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, final iteration 91 and overuse 0 matched. FASM SHA-256 is `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 is `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, and final aclk Fmax is 41.83 MHz for both.

Reference → candidate whole wall **436.33 → 432.02 s (-0.9878%)**; Router2 **317.94 → 314.88 s (-0.9624%)**. Both fail the required 3.0% improvement gate. User time 435.24 → 430.92 s; peak RSS 1,558,080 → 1,558,820 KiB. **No B2, rerun, merge or tag.** Evidence: `comparison.json`, `compare.log`, `reference-reproduction.json`, per-role logs/reports/FASM and `pair.json` in the result dir.

## Hypothesis and scope

`ThreadContext::processed_sinks` is a `pool<WireId>` used only for membership tests and insertion. `route_net()` currently calls `pool::clear()`, which resets the bucket vector's size to zero while retaining its allocation; the next insertion resizes and initializes the bucket array from the retained entry capacity. Add a pool operation that resets all existing buckets to `-1` and clears entries without changing bucket size or releasing storage, and use it only for `processed_sinks` at the per-net reset point.

This preserves set membership, insertion results, route/arc order, scoring, queue operations and RNG while removing repeated bucket allocation/initialization churn in Router2 scratch handling. It does not include the rejected router priority-queue capacity/reserve experiment, change generic `pool::clear()`, or touch other scratch containers. The profile does not isolate this cost and promises no gain.

## Kill criterion and validation

Build once from the stated base in Release/IPO mode for xc7z020 only, with both C/C++ launchers set to `ccache`, on cores 40-47. Verify ancestry, the minimal tracked diff, `git diff --check`, generated version and binary SHA-256.

Run one detached B1 pair on candidate core 2 and verified-reference core 3, seed 1, one thread, 100 MHz and a 1920 s cap, concurrently with the independent bind-no-resource pair on cores 10/11. Reference reproduction precedes candidate metrics and must match complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require expected timing-only exit 1 after normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, plus at least 3.0% improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag.
