# Router2 processed-sinks scratch reuse r1

## Status

**CLAIMED; source, build and B1 result pending.** Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Expected tier 1.

## Hypothesis and scope

`ThreadContext::processed_sinks` is a `pool<WireId>` used only for membership tests and insertion. `route_net()` currently calls `pool::clear()`, which releases the bucket vector; the next insertion allocates and rehashes a bucket array sized from the retained entry capacity. Add a pool operation that resets all existing buckets to `-1` and clears entries without releasing bucket storage, and use it only for `processed_sinks` at the per-net reset point.

This preserves set membership, insertion results, route/arc order, scoring, queue operations and RNG while removing repeated bucket allocation/initialization churn in Router2 scratch handling. It does not include the rejected router priority-queue capacity/reserve experiment, change generic `pool::clear()`, or touch other scratch containers. The profile does not isolate this cost and promises no gain.

## Kill criterion and validation

Build once from the stated base in Release/IPO mode for xc7z020 only, with both C/C++ launchers set to `ccache`, on cores 40-47. Verify ancestry, the minimal tracked diff, `git diff --check`, generated version and binary SHA-256.

Run one detached B1 pair on candidate core 2 and verified-reference core 3, seed 1, one thread, 100 MHz and a 1920 s cap, concurrently with the independent bind-no-resource pair on cores 10/11. Reference reproduction precedes candidate metrics and must match complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require expected timing-only exit 1 after normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, plus at least 3.0% improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag.
