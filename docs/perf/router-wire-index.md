# Flat WireId index for Router2

Branch: `perf/router-wire-index`

Baseline: `d56c470f523846ddfb5272c724cdac70b1781a51`.

## Hypothesis

`dict<WireId, int>::do_lookup` was the largest Router2 self-time item in both profiles (19.68% on B1 and 18.38% on B2). Himbaechel `WireId` already contains a tile and a tile-local integer index. Replace the hash dictionary with per-tile offsets into a flat integer lookup vector. Keep `flat_wires` in the original `getWires()` order so routing order and random choices do not change.

The setup pass supports sparse tile-local indices and initializes unoccupied entries to `-1`. The hot lookup retains the vector bounds check; valid routing wires still select exactly the same `PerWireData` entries as before.

## Validation

Compare the unpatched and patched Release/IPO builds sequentially on the same core with seed 1 and one thread. Require identical report JSON and complete FASM, unchanged QoR and iteration counts, and measure wall time and peak RSS on B1 before B2.

## B1 results

The unpatched and patched Release builds ran sequentially on core 24. Raw results are under `$BENCH_RESULTS/20261103T000226Z-nextpnr-perf-lane-{release-base-wire,router-wire}-b1-t1`.

| Metric | Unpatched | Flat index | Change |
|---|---:|---:|---:|
| Wall (s) | 578.45 | 492.63 | -14.84% |
| User CPU (s) | 577.07 | 491.49 | -14.83% |
| Max RSS (KiB) | 1,471,088 | 1,268,560 | -13.77% |
| Analytic iterations (s) | 43.57 | 44.12 | +1.26% |
| Simulated annealing (s) | 45.00 | 44.20 | -1.78% |
| Router2 (s) | 480.84 | 395.47 | -17.75% |
| Final router iteration | 91 | 91 | unchanged |
| Post-place Fmax (MHz) | 51.00 | 51.00 | unchanged |
| Final Fmax (MHz) | 41.83 | 41.83 | unchanged |

Both exits were timing-only. Parsed report JSON is identical and complete FASM is byte-identical (SHA-256 `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`). The candidate passes B1 with substantial runtime and memory improvements. B2 validation is running; this remains one measurement pair rather than a noise-bound estimate.
