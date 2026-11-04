# Router2 visited-check PerWireData passing r1

## Status

**CLAIMED; implementation and build pending.** Candidate `perf/visited-check-wiredata-r1` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. This is a semantics-preserving performance candidate, expected tier 1.

## Hypothesis and scope

The abandoned `visit-data-access-r1` only introduced a local reference inside `was_visited_*`; it still performed one `flat_wires.at(wire)` per check and regressed both timing metrics. This distinct follow-on passes the `PerWireData` objects already resolved at the four hot exploration call sites (`curr_data` or `nwd`) to the visited checks, eliminating those lookups without changing the backtrace loops or the abandoned route-backtrace candidate. Preserve short-circuit comparison order, queue/cursor order, costs, RNG, dirty-wire insertion and every route decision. The stale profile does not promise a gain; B1 is the decisive screen.

## Kill criterion and acceptance gates

Build once on cores 40-63 with Release/IPO, xc7z020 only, both C/C++ compiler launchers set to `ccache`, and the pinned generated version. Record source/binary hashes, build wall and verified version.

Run one detached B1 pair only when its assigned cores are free, with reference reproduction before candidate metrics. Require expected timing-only exit 1, normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% paired improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag. B2 performance testing remains blocked while the reference is nondeterministic.
