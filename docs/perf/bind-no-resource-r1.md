# Router2 bind/unbind no-resource lookup guard r1

## Status

**CLAIMED; source, build and B1 result pending.** Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Expected tier 1.

## Hypothesis and scope

`setup_resources()` populates `wire_to_resource` from resource-group destination wires before routing. On the Xilinx benchmark it remains empty, so `getResourceKeyForPip()` necessarily returns `GroupId()` and the resource-accounting paths in `bind_pip_internal` and `unbind_pip_internal` cannot be taken. Guard only those two per-pip resource-key/accounting paths with `!wire_to_resource.empty()`. Architectures with resources retain the original code.

Preserve wire binding/unbinding order, congestion counts, pip checks, resource accounting on nonempty architectures, route traversal, scoring, queue operations and RNG. This is distinct from abandoned `no-resource-fastpath-r1` (`route_arc` conflict checks) and `score-no-resource-r1` (scoring lookup); neither is included or rerun. The profile does not isolate this lookup and promises no gain.

## Kill criterion and validation

Build once from the stated base in Release/IPO mode for xc7z020 only, with both C/C++ launchers set to `ccache`, on cores 40-47. Verify ancestry, the minimal tracked diff, `git diff --check`, generated version and binary SHA-256.

Run one detached B1 pair on candidate core 10 and verified-reference core 11, seed 1, one thread, 100 MHz and a 1920 s cap, concurrently with the independent processed-sinks pair on cores 2/3. Reference reproduction precedes candidate metrics and must match complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require expected timing-only exit 1 after normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, plus at least 3.0% improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag.
