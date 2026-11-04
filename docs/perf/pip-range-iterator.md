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

Build once with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-63, using the reference version string so the complete FASM header can match. The candidate was named and published before implementation.

Candidate source `eb703476c5d71f3aebe35125fb00d9b167763d9c` is pushed on `perf/pip-range-iterator`. The Release/IPO build completed successfully; binary SHA-256 is `8882e8e50b5dbe7441fced3b95f232d8cd77f92e2c3715ae68280a8aeb715db2`. The architecture database integrity test (`--test`) passed with the reference chipdb.

## B1 screen: running

The detached B1 pair started at 2026-11-04T09:19:10Z in `bench/results/20261104T091700Z-pip-range-iterator-b1`, with candidate core 6/reference core 7, seed 1, one thread, 100 MHz and a 1920 s cap. Both pip-delay B2 pairs had already finished, so actual total benchmark concurrency is 2 rather than the originally planned 6. Driver PID is 50723; candidate wrapper/actual PIDs are 50745/50747 and reference wrapper/actual PIDs are 50742/50744. Reference reproduction must pass before candidate comparison is considered.
