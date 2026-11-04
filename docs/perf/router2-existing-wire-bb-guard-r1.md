# Router2 existing-wire bounding-box guard r1

## Status

**CLAIMED; not yet built.** This is a correctness/determinism investigation under `b2-determinism-root-cause`, not a performance claim. Candidate B2 results remain **B2 tier-2 (reference nondeterministic)** until the verified reference reproduces twice or a superseding directive arrives.

## Base and hypothesis

Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`, without any prior candidate patch. Branch/worktree: `perf/router2-existing-wire-bb-guard-r1` / `nextpnr-router2-bb-guard-r1`. Touched phase: Router2 `route_net`, protecting `ripup_arc`, existing-route seeding and visit state.

Threaded binning assigns nets using `nd.bb`, and `thread_test_wire` protects newly expanded wires against accesses outside a worker's `ThreadContext.bb`. An arc that fails in threaded mode is subsequently rerun serially and may succeed without a bounding box. On a later iteration, the net's existing `nd.wires` can therefore extend outside its newly assigned worker bbox. `route_net` currently rips up failed arcs and seeds from existing routing before any equivalent thread-boundary check, allowing shared `flat_wires` visit/congestion state to be accessed outside the partition.

Before any threaded route/rip-up mutation, scan the current `nd.wires`. If any existing wire lies outside the worker bbox, return failure without mutating shared state so the existing failed-net path reroutes the whole net serially after all workers join. Preserve scoring, route ordering, RNG, serial routing and all wire/resource semantics.

## Kill criterion and validation

- Build from the stated base with Release/IPO, xc7z020-only, C/C++ `ccache` launchers and the pinned generated version; verify source/binary/chipdb hashes and clean diffs.
- Run the permitted B1 reference/candidate identity screen. Reference reproduction precedes candidate metrics. Any abnormal completion, reference-reproduction failure or candidate identity failure kills this guard without a rerun.
- The determinism test is two normally completed candidate B2 repeats with byte-identical complete FASM/report outputs and matching histories/QoR. Failure to complete or any output difference kills this guard without a rerun and returns to rip-up order/hash-iteration investigation under the root-cause claim.
- Do not use the nondeterministic reference as a candidate-B2 byte-identity oracle. Do not claim or accept a performance improvement from this correctness guard; any integration requires all applicable supervisor gates.

The next named candidates remain `perf/route-backtrace-reference-r1`, `perf/visited-check-wiredata-r1`, and further rip-up/hash-iteration investigation under `b2-determinism-root-cause`.
