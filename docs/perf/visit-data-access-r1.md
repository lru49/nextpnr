# Router2 visited-data access reuse

## Status

**ABANDONED on 2026-11-04 at the B1 timing gate.** Candidate `perf/visit-data-access-r1` at `dd3000de0ce81caff8cb336ec0927a37c0989f10` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Candidate binary SHA-256 is `aadd14e2d7173eda4b0d23319b28ab26befe30c11391c5c2c2a68214a96acb8a`. Identity/QoR passed, but both required timing metrics regressed. No B2, rerun, merge or tag is permitted.

## Hypothesis and scope

`was_visited` in `common/route/router2.cc` performs two checked `flat_wires.at(wire)` accesses for each visited-wire test. Reuse one local `const PerWireData &` for `visited_fwd` and `cost_fwd`. Apply the same semantics-preserving access reuse to the backward-direction branch. Do not change comparisons, queue order, route decisions, costs, arithmetic, pip traversal or bindings.

The candidate removes redundant access and bounds-check overhead in Router2's hot expansion path. The old-base profile does not isolate these accesses, so it supplies no promised gain. Expected tier: 1.

## Kill criterion and acceptance gates

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using version string `perf-accepted-qor-revert-40d3b873`. Per block 35, run one detached B1 pair on candidate core 6 and the verified reference on core 7, seed 1, one thread, 100 MHz and a 1920 s cap.

The Release/IPO xc7z020-only build completed 63/63 steps in 214.75 s with peak RSS 2,678,000 KiB. Both compiler launchers and the generated version string were verified.

Before candidate metrics are read, the reference must reproduce:

- complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`;
- report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`;
- checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`;
- history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require normal completion with the expected timing-only exit 1, byte-identical complete FASM, identical report bytes and parsed report, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.

## B1 result

Pair `20261104T113600Z-visit-data-access-r1-b1` completed normally with expected timing-only exits 1/1 at launch concurrency 1 pair. The reference reproduced every stored identity value before candidate metrics were read. Candidate complete FASM and report SHA-256 matched the stored values; report bytes/parsed report, full router history, checksums, iteration 91, final overuse zero and Fmax 41.83 MHz all matched. Comparison exit was 0.

Reference -> candidate whole wall was 399.03 -> 404.03 s (**+1.2530%**, slower), and Router2 was 291.53 -> 295.15 s (**+1.2417%**, slower). Both metrics fail the required at-least-3.0% improvement. **Decision: ABANDONED on timing; no B2, rerun, merge or accepted tag.** Raw artifacts remain in `bench/results/20261104T113600Z-visit-data-access-r1-b1`.
