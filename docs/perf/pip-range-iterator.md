# Himbaechel pip-range iterator caching

## Hypothesis and scope

Candidate `perf/pip-range-iterator` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` and changes only `himbaechel/arch.h`.

The old-base B1 profile attributes 1.87% whole-run and 2.25% Router2 self time to `Arch::getPipsUphill`, 0.73% Router2 to `getPipsDownhill`, 1.55%/1.85% to `chip_wire_info`, and 2.00%/1.65% to `chip_tile_info`. Those functions have other callers, so the figures are plausible upper bounds rather than an additive speedup prediction.

`UpdownhillPipIterator::operator++` currently resolves `*twi` and `chip_wire_info` to find the current pip count. `operator*` then repeats both resolutions for the same tile wire. The iterator will cache the current denormalised `WireId` and `TileWireDataPOD` pointer, refreshing them only when advancing to another tile wire. Constructor/end-sentinel behavior, empty-range skipping, node traversal order, cursor values and returned `PipId`s remain unchanged.

No design-sized table is added. Pip availability, delays, bindings, endpoint normalisation, wire locations, scoring and route decisions are outside the change.

## Kill criterion and acceptance gates

Run one detached B1 pair with candidate on core 6 and reference on core 7, seed 1, one thread, 100 MHz and a 1920 s cap, concurrently with the two existing pip-delay B2 pairs (total concurrency 6). Require:

- normal routing completion;
- byte-identical complete FASM;
- identical report bytes and parsed reports, router history, checksums, Fmax, utilization and final overuse;
- at least 3.0% paired improvement in both Router2 time and whole wall time.

Any identity/QoR failure, non-convergence or timing improvement below 3.0% in either metric abandons the candidate with no B2 or rerun. A passing screen requires further confirmation before acceptance.

Before candidate metrics are read, the reference must reproduce FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

## Build and status

Build once with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using the reference version string so the complete FASM header can match. The candidate is named and published before implementation; no build or benchmark result exists yet.
