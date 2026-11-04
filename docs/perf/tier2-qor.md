# Tier-2 candidate evidence

This record uses the existing paired artifacts only; no benchmark was rerun. Values are **reference -> candidate**. Tier 2 requires equal or better QoR on every valid benchmark, final overuse 0, router convergence, and a recorded FASM difference. Tier 1 is not claimed for any candidate with a differing-FASM row in this record.

`1T` is the documented B1 timing-only exit 1: both sides fail the 100 MHz constraint with identical Fmax. It is not timing closure. The final route triple is `overused / overuse / archfail`. Utilization is reported as the number of equal report fields out of 52.

## Pip-binding index

- Candidate: `9299e4be8c99fdd61e92df47667d957d2c4963c2`
- Reference: `d56c470f523846ddfb5272c724cdac70b1781a51`
- Result: **tier-2 QoR gate passes**. All three B1 pairs are byte-identical; all three B2 pairs have identical report bytes and utilization but different FASM, history and checksums. Tier 1 is not accepted for the whole candidate.

| Pair | Design | Exit | Final Fmax (MHz) | Utilization | Final route triple | Converged | Iterations |
|---|---|---|---|---|---|---|---|
| P1 | B1 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| P2 | B1 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| P3 | B1 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| P4 | B2 | 0 -> 0 | 30.538082122802734 -> 30.538082122802734 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 23 -> 17 |
| P5 | B2 | 0 -> 0 | 30.538082122802734 -> 30.538082122802734 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 23 -> 18 |
| P6 | B2 | 0 -> 0 | 30.538082122802734 -> 30.538082122802734 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 23 -> 18 |

| Pair | FASM status | All / payload hunks | Non-comment added | Non-comment deleted | Non-comment changed |
|---|---|---:|---:|---:|---:|
| P1 | identical | 0 / 0 | 0 | 0 | 0 |
| P2 | identical | 0 / 0 | 0 | 0 | 0 |
| P3 | identical | 0 / 0 | 0 | 0 | 0 |
| P4 | different | 2,735 / 2,734 | 1,637 | 1,657 | 9,779 |
| P5 | different | 2,719 / 2,718 | 1,625 | 1,673 | 9,469 |
| P6 | different | 2,719 / 2,718 | 1,625 | 1,673 | 9,469 |

Evidence root: `/home/gremen/.cache/nextpnr-perf-lane/bench/results/20261103T161300Z-pip-binding-six-pairs/`.

- P1-P3: `b1-pip-binding-index-r1` through `b1-pip-binding-index-r3`.
- P4: `b2-pip-binding-index-r1`.
- P5-P6: `wave2/b2-pip-binding-index-r2` and `wave2/b2-pip-binding-index-r3`.

The first-wave B2 r2/r3 directories are excluded: both exits were 143, with no report or FASM. The completed wave2 replacements are the valid records.

## Reject-first router

- Candidate: `05231767371e64ff7a42b2226a76f20e4767ba26`
- Direct reference/parent: pip-binding `9299e4be8c99fdd61e92df47667d957d2c4963c2`
- Result: **tier-2 QoR gate passes**. B2 improves final Fmax by 0.460172%, utilization is unchanged, and FASM differs. R2 establishes an incremental B1 identity pass; R1 is only a payload-identity diagnostic.

| Pair | Design | Exit | Final Fmax (MHz) | Utilization | Final route triple | Converged | Iterations |
|---|---|---|---|---|---|---|---|
| R1 | B1 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| R2 | B1 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| R3 | B2 | 0 -> 0 | 30.538082122802734 -> 30.67860984802246 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 16 -> 16 |

| Pair | FASM status | All / payload hunks | Non-comment added | Non-comment deleted | Non-comment changed |
|---|---|---:|---:|---:|---:|
| R1 | version-comment replacement only; payload identical | 1 / 0 | 0 | 0 | 0 |
| R2 | identical | 0 / 0 | 0 | 0 | 0 |
| R3 | different | 27,548 / 27,547 | 19,126 | 18,761 | 87,532 |

Evidence root: `/home/gremen/.cache/nextpnr-perf-lane/bench/results/20261103T203700Z-router-reject-first/`; pairs are `b1-router-reject-first-r1`, `b1-router-reject-first-r2` and `b2-router-reject-first-r1`.

R3 report bytes, parsed report, history and checksums differ; report differences are Fmax and critical paths. The abandoned cumulative `ed9af634` result is separate and is not evidence for this candidate.

## Router pip-location reuse

- Candidate: `02411403c8b8ab8ecba0279802a20b6121fe0581`
- Reference/parent: router-stack `9a83eaa0ef4b4fe52ce8395a167f92318d7fc2c3`
- Result: **tier-2 QoR gate passes**. Both B1 pairs and B2 r1 are byte-identical; B2 r2 has identical report bytes and utilization but different FASM, history and checksums. Tier 1 is not accepted for the whole candidate.
- Disposition: **measured, not accepted: below the tier-2 gain floor**. B1 wall +0.4% / +1.4%; B2 wall -0.1% / -0.8%; required floor -3% on both B1 and B2. Integration was reverted by `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`; the original acceptance tag remains as a historical record.

| Pair | Design | Exit | Final Fmax (MHz) | Utilization | Final route triple | Converged | Iterations |
|---|---|---|---|---|---|---|---|
| L1 | B1 r1 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| L2 | B1 r2 | 1T -> 1T | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 |
| L3 | B2 r1 | 0 -> 0 | 30.538082122802734 -> 30.538082122802734 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 16 -> 16 |
| L4 | B2 r2 | 0 -> 0 | 30.538082122802734 -> 30.538082122802734 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 18 -> 16 |

| Pair | FASM status | All / payload hunks | Non-comment added | Non-comment deleted | Non-comment changed |
|---|---|---:|---:|---:|---:|
| L1 | identical | 0 / 0 | 0 | 0 | 0 |
| L2 | identical | 0 / 0 | 0 | 0 | 0 |
| L3 | identical | 0 / 0 | 0 | 0 | 0 |
| L4 | different | 3,155 / 3,154 | 2,288 | 2,326 | 9,852 |

Evidence roots: `/home/gremen/.cache/nextpnr-perf-lane/bench/results/20261104T001600Z-pip-location-reuse-b1/` (L1-L2) and `/home/gremen/.cache/nextpnr-perf-lane/bench/results/20261104T004300Z-pip-location-reuse-b2/` (L3-L4). L4 preserves report bytes, parsed report, Fmax, utilization and final overuse; only its routing outcome differs.

## FASM counting convention

Differences use GNU `diff -U0`, direction reference -> candidate. A non-comment line is one whose first non-whitespace character is not `#`. "All hunks" includes comments; "payload hunks" contain at least one changed non-comment line. Within each hunk, `changed = min(non-comment deletions, non-comment additions)`; unmatched remaining lines are counted as added or deleted. Changed lines are paired replacements, not double-counted.

Every valid run log contains `Routing complete.` and `Program finished normally.`. These are nextpnr comparisons only; no bitstream or board-level test was performed.
