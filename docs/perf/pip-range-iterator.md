# Himbaechel pip-range iterator caching

## Status

**ABANDONED on 2026-11-04.** The B1 pair preserved complete identity/QoR and improved Router2 by 3.7459%, but whole-wall improvement was only 2.3952%, below the required 3.0% in both metrics. No B2, rerun, merge or tag follows.

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

Candidate source `eb703476c5d71f3aebe35125fb00d9b167763d9c` is pushed on `perf/pip-range-iterator`. The Release/IPO build completed in 215.11 s with peak RSS 2,678,040 KiB; binary SHA-256 is `8882e8e50b5dbe7441fced3b95f232d8cd77f92e2c3715ae68280a8aeb715db2`. The architecture database integrity test (`--test`) passed with the reference chipdb.

## B1 screen: wall-time gate failure; abandon

The detached B1 pair started at 2026-11-04T09:19:10Z in `bench/results/20261104T091700Z-pip-range-iterator-b1`, with candidate core 6/reference core 7, seed 1, one thread, 100 MHz and a 1920 s cap. Both pip-delay B2 pairs had already finished, so actual total benchmark concurrency was 2 rather than the originally planned 6.

The reference reproduced every stored FASM/report/checksum/history/Fmax value before candidate metrics were inspected. Both runs then completed normally with the expected timing-only exit 1. Complete FASM, report bytes, parsed reports, full router history and checksums are identical; iteration 91, final route 0/0/0 and Fmax 41.83 MHz are unchanged.

| Metric | Reference | Candidate | Change |
| --- | ---: | ---: | ---: |
| Whole wall | 387.03 s | 377.76 s | **-2.3952% — FAIL** |
| Router2 | 281.64 s | 271.09 s | **-3.7459% — pass** |
| User time | 385.88 s | 376.67 s | -2.3868% |
| Analytic iteration sum | 42.66 s | 42.78 s | +0.2813% |
| HeAP | 45.24 s | 45.38 s | +0.3095% |
| SA | 53.46 s | 54.64 s | +2.2073% |
| Peak RSS | 1,558,076 KiB | 1,558,332 KiB | +0.0164% |

The prepublished kill criterion requires at least 3.0% improvement in both wall and Router2. Whole wall misses that gate, so the candidate is abandoned without a second B1, B2, merge or tag.
