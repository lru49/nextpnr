# Router scoring fast path

Source revision: `d56c470f523846ddfb5272c724cdac70b1781a51`.

## Change

- Compute `get_base_cost()` once per forward or backward expansion and reuse the value for both the accumulated delay and the arc score.
- Pass the already-indexed `PerWireData` into `score_wire_for_arc()` instead of looking up the same flat wire again.
- Replace `count()` followed by `at()` with one `find()` in the arc-score and to-go-cost net-wire lookups.

The patch does not change queue ordering, iteration limits, congestion updates, random-number consumption, or floating-point expressions. The removed calls are duplicate lookups of the same values, so placement, routing, QoR, and FASM are expected to be identical.

## Build and validation

Release build with IPO, the same xc7z020 target and chipdb inputs as the unpatched companion build, and `CURRENT_GIT_VERSION=d56c470f`.

Benchmarks use seed 1, one worker thread, and a 100 MHz timing target. Compare wall time, maximum RSS, router iterations and phase time, utilization, post-place/final Fmax, and complete FASM files against the unpatched build. A speedup is reported only with the companion-build comparison; the initial reference timings were concurrent and provisional.

Results: pending.
