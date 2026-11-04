# Router2 visit generations

## Status

ABANDONED after B1 r1: outputs were identical, but replacing dirty-wire resets with per-search generations regressed both wall and Router2 time. No B2 run or bitstream was launched.

## Implementation

- Branch: `perf/l2-router2-visit-generation`
- Candidate commit: `f79047663910a06f0545e48745610b750d4b87cf`
- Base: accepted tier-2 commit `5d36ce50`
- Candidate binary SHA-256: `ca61f2731af813d8e7577981b9076ddbe803bdf081e071589e14ec501ece005a`
- Reference binary SHA-256: `881d8b4f66bba9375067c814b3eb223b888ed2bcc4bee3c16786560e100d36b7`

The change replaces `dirty_wires` clearing with a globally unique relaxed-atomic 64-bit generation for every search. Each wire records its generation; stale direction flags are ignored and cleared on first reuse. Generations are unique across threads and routing iterations, so they do not alter search ordering or cost comparisons.

## B1 r1

Result root: `/home/ubuntu/kimi/nextpnr-perf-lane2/bench/results/20261104T165208Z-router2-visit-generation-b1-r1`.

- Candidate PID 1088698 on core 26; reference PID 1088697 on core 27; seed 1, one requested thread, timeout 1920 s.
- Reference reproduction against the preceding accepted-tier-2 reference: PASS. Raw FASM, report, complete placement/router histories, and checksums matched.
- Candidate/reference identity: PASS. FASM payloads, reports, placement histories, complete Router2 iteration histories, and checksums matched. Both exited 1 for the expected 100 MHz timing failure after otherwise normal completion.
- Wall time: reference 397.45 s, candidate 402.38 s, candidate/reference change +1.2404% (slower).
- Router2 time: reference 280.57 s, candidate 285.59 s, change +1.7892% (slower).
- Fmax: identical at 41.830501556396484 MHz for `aclk`; both fail the intentionally tight 100 MHz constraint.
- Timing gates: FAIL; neither wall nor Router2 achieved the required at-least-3% improvement.

The added per-wire generation state and generation checks outweighed the eliminated reset writes in this workload. Machine-readable evidence is in `reference-reproduction.json` and `comparison.json` under the result root. Recorded 2026-11-04T17:03:08Z.
