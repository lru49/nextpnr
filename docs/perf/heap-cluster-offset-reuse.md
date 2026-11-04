# HeAP cluster-offset reuse

## Status

ABANDONED after B1 r1: placement and routing outputs were identical, but both wall time and Router2 time regressed. No B2 run or bitstream was launched.

## Implementation

- Branch: `perf/l2-heap-cluster-offset-reuse`
- Candidate commit: `0c9ae750f4bede309c7f0a4ec1a7338b0d35c478`
- Base: accepted tier-2 commit `5d36ce50`
- Candidate binary SHA-256: `49581f7fc3f091b9fa825d0f9d77e75e971b798e288bb422e6b3ab43d5a2f33c`
- Reference binary SHA-256: `881d8b4f66bba9375067c814b3eb223b888ed2bcc4bee3c16786560e100d36b7`

The change caches each clustered child's `getClusterOffset` result during single-threaded `setup_solve_cells`, then reuses the existing `cell_locs` lookup while stamping equations. The cache is read-only during parallel X/Y equation builds.

## B1 r1

Result root: `/home/ubuntu/kimi/nextpnr-perf-lane2/bench/results/20261104T162736Z-heap-cluster-offset-reuse-b1-r1`.

- Candidate PID 1041993 on core 26; reference PID 1041992 on core 27; seed 1, one thread, timeout 1920 s.
- Reference reproduction against `20261104T143323Z-portref-iteration-b1/reference`: PASS. Raw FASM, report, complete placement/router histories, and checksums matched.
- Candidate/reference identity: PASS. FASM payloads, reports, placement histories, the complete Router2 iteration history, and checksums matched. Both exited 1 for the expected 100 MHz timing failure after otherwise normal completion.
- Wall time: reference 414.23 s, candidate 416.01 s, candidate/reference change +0.4297% (slower).
- Router2 time: reference 298.67 s, candidate 300.83 s, change +0.7232% (slower).
- Fmax: identical at 41.830501556396484 MHz for `aclk`; both fail the intentionally tight 100 MHz constraint.
- Timing gates: FAIL; neither wall nor Router2 achieved the required at-least-3% improvement.

Machine-readable evidence is in `reference-reproduction.json` and `comparison.json` under the result root. Recorded 2026-11-04T16:42:55Z.
