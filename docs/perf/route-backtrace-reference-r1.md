# Router2 backtrace per-wire access reuse

## Status

**CLAIMED; implementation and build pending.** Candidate `perf/route-backtrace-reference-r1` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. This is a semantics-preserving performance candidate, expected tier 1.

## Hypothesis and scope

The forward and backward successful-route backtrace loops repeatedly resolve the same `flat_wires.at(cursor)` element within one iteration. Reuse a local reference to that existing `PerWireData` object in each loop. Change only those accesses in `common/route/router2.cc`; preserve cursor order, pip binding order, queue contents, costs, arithmetic and all route decisions. The stale profile does not isolate backtrace overhead and supplies no promised gain.

## Kill criterion and acceptance gates

Build once on cores 40-63 with Release/IPO, xc7z020 only and both C/C++ compiler launchers set to `ccache`. Record the source and binary hashes, build wall and verified version.

Screen with one detached B1 pair on candidate core 2 and reference core 3, seed 1, one thread, 100 MHz and a 1920 s cap. Reference reproduction must complete before candidate metrics are read: expected timing-only exit 1, normal routing/program completion, complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal candidate completion with expected timing-only exit 1, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% paired improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag. B2 performance testing remains blocked while the reference is nondeterministic.
