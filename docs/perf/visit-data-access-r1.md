# Router2 visited-data access reuse

## Status

**NAMED BEFORE BUILD on 2026-11-04.** Candidate `perf/visit-data-access-r1` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`; source commit and binary hash are pending. No B2 run is permitted while the B2 reference remains nondeterministic.

## Hypothesis and scope

`was_visited` in `common/route/router2.cc` performs two checked `flat_wires.at(wire)` accesses for each visited-wire test. Reuse one local `const PerWireData &` for `visited_fwd` and `cost_fwd`. Apply the same semantics-preserving access reuse to the backward-direction branch. Do not change comparisons, queue order, route decisions, costs, arithmetic, pip traversal or bindings.

The candidate removes redundant access and bounds-check overhead in Router2's hot expansion path. The old-base profile does not isolate these accesses, so it supplies no promised gain. Expected tier: 1.

## Kill criterion and acceptance gates

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using version string `perf-accepted-qor-revert-40d3b873`. Run one detached B1 pair on candidate core 2 and the verified reference on core 3 (or the block-34 alternative cores 10/11), seed 1, one thread, 100 MHz and a 1920 s cap.

Before candidate metrics are read, the reference must reproduce:

- complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`;
- report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`;
- checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`;
- history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal completion with the expected timing-only exit 1, byte-identical complete FASM, identical report bytes and parsed report, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.
