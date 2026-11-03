# Flat WireId index for Router2

Branch: `perf/router-wire-index`

Baseline: `d56c470f523846ddfb5272c724cdac70b1781a51`.

## Hypothesis

`dict<WireId, int>::do_lookup` was the largest Router2 self-time item in both profiles (19.68% on B1 and 18.38% on B2). Himbaechel `WireId` already contains a tile and a tile-local integer index. Replace the hash dictionary with per-tile offsets into a flat integer lookup vector. Keep `flat_wires` in the original `getWires()` order so routing order and random choices do not change.

The setup pass supports sparse tile-local indices and initializes unoccupied entries to `-1`. The hot lookup retains the vector bounds check; valid routing wires still select exactly the same `PerWireData` entries as before.

## Validation

Compare the unpatched and patched Release/IPO builds sequentially on the same core with seed 1 and one thread. Require identical report JSON and complete FASM, unchanged QoR and iteration counts, and measure wall time and peak RSS on B1 before B2.

Results: pending.
