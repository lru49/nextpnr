# Xilinx flat wire-location lookup

## Hypothesis and scope

Candidate `perf/wire-location-flat-r2` at `c06eacfd` is based on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. It minimally reapplies the source-only approach in `19d4e98d`, changing only `himbaechel/uarch/xilinx/xilinx.cc` and `.h`.

The existing B1/B2 profiles attribute 5.96%/4.87% of whole-run self time and 7.16%/8.55% of Router2 self time to `dict<WireId, Loc>::do_lookup`. `find_source_sink_locs`, `estimateDelay` and `getRouteBoundingBox` repeatedly query two hash dictionaries for source/sink locations.

The candidate replaces only those dictionaries with dense tile-wire-indexed `std::vector<Loc>` storage and tile prefix offsets. The existing `Loc` default `(-1,-1,-1)` remains the absent sentinel. BFS order, stored location values, delay expressions, fallback branches and bounding-box operations are unchanged. Direct indexing should remove hashing and lookup overhead without changing placement, routing or FASM.

The earlier 12-way B1 attempt of `19d4e98d` timed out on both candidate and reference and is inconclusive under directive block 6. This candidate uses a fresh isolated pair and makes no performance claim from that contended run.

## Kill criterion and acceptance gates

The first screen is one detached B1 pair with candidate on core 4 and reference on core 5, using seed 1, one thread, 100 MHz and a 1920 s cap. It requires:

- normal routing completion;
- byte-identical complete FASM;
- identical report bytes and parsed reports, router history, checksums, Fmax, utilization and final overuse;
- at least 3.0% paired improvement in both Router2 time and whole wall time.

Any identity/QoR failure, non-convergence or timing improvement below 3.0% in either metric abandons the candidate with no B2. A passing screen requires a second B1 pair and two B2 pairs under the existing tier gates before any merge or tag.

## Status

Source branch is pushed. Build and isolated B1 measurement are pending.
