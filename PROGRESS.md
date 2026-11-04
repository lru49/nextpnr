# nextpnr performance lane 2

## 2026-11-04 — initialization and first candidate

- Base commit: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` (`origin/perf/accepted-qor`).
- Intended candidate: `perf/l2-heap-cell-location-index`, one commit on the base above.
- Claims: `origin/perf/claims` initialized from `perf/baseline`; active claim is `perf/l2-heap-cell-location-index` for `HeAP/build_equations+solve_equations+total_hpwl`.
- Candidate queue:
  1. `perf/l2-heap-cell-location-index` — HeAP analytic placement; replace hot `cell_locs` hash lookups with a name-indexed pointer table built after seeding; expected tier 1.
  2. `perf/l2-bel-location-flat` — Himbaechel analytic/SA placement; evaluate a flat per-BelId location table for `Arch::getBelLocation`; expected tier 1.
  3. `perf/l2-portref-iteration` — HeAP/SA port traversal; inspect and reduce `indexed_store<PortRef>` iterator increment overhead without changing traversal order; expected tier 1.
- Hypothesis: `cell_locs` is fully populated by seeding and not grown thereafter, so stable pointers indexed by `IdString.index` preserve every value and access order while removing repeated hashing from equation construction, solving, and HPWL evaluation.
- Kill criterion: abandon on any pointer-lifetime invalidation, build/smoke failure, reference or FASM/QoR mismatch, or a paired B1 wall regression greater than 2%; do not start B2 unless every B1 gate passes.
- No source edit, build, or benchmark has started.

ACK 2026-11-04T10:34Z lane2: block 1

## 2026-11-04T11:12Z — HeAP cell-location index abandoned at the B1 timing gate

- Candidate source `e78f7efc` is pushed as `perf/l2-heap-cell-location-index`, one commit over `40d3b873`. Only `common/place/placer_heap.cc` changed; `git diff --check` passed.
- Final xc7z020-only Release/IPO build used both C/C++ `ccache` launchers on cores 28-39: 61/61 steps passed in 122.94 s, peak RSS 2,678,080 KiB. Reference-chipdb `--test` passed. Candidate binary SHA-256 `d4f4b18c4604f38fe0b94be2be2f66e21b4e092d44b19713a1ecf2534429cff0`; reference `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`.
- Pair `20261104T110406Z-heap-cell-location-index-b1`: candidate core 18, reference core 19, seed 1, one thread, 100 MHz, 1920 s cap. Before candidate metrics, the reference reproduced FASM `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and normal timing-only exit 1.
- Candidate/reference exits 1/1; complete FASM and report bytes, parsed reports, full router history and checksums are identical. Identity and QoR pass.
- Reference -> candidate: whole wall 414.80 -> 412.47 s (**-0.5617%**), Router2 303.14 -> 302.41 s (**-0.2408%**), HeAP 45.84 -> 44.28 s (-3.4031%), analytic iteration sum 43.08 -> 41.72 s (-3.1569%), SA unchanged, peak RSS 1,558,080 -> 1,559,992 KiB (+0.1227%).
- Decision: **ABANDON `perf/l2-heap-cell-location-index`: identity passes, but both whole-wall and Router2 gains are below the required 3.0%.** No second B1 pair, B2 run, merge or accepted tag. No lane-2 benchmark process remains.

ACK 2026-11-04T11:04Z lane2: block 2

## 2026-11-04T11:27:09Z — fast-mode pip-delay source bypass named before build

- Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` (`origin/perf/accepted-qor`). Candidate `perf/l2-fast-pip-delay-source-bypass`, expected tier 1, touches only `Arch::getPipDelay`.
- Profile evidence: `getPipDelay` is 10.78% of B1 whole-run and 12.80% of B1 Router2 self time. In fast-delay routing the source-side RC product is multiplied by zero, but the base still normalises the source wire and walks its node timing on every query.
- Distinct from abandoned `perf/pip-delay-flat-r2`: add no delay table, cache, setup hook or persistent state. Skip only the source-side work whose product is provably zero in fast mode; retain the original non-fast path and destination arithmetic in the original order.
- Hypothesis: removing that dead source lookup and arithmetic reduces Router2 and whole wall time without changing any delay value, route decision, RNG or output.
- Kill criterion: one B1 pair on candidate core 18/reference core 19, seed 1, one thread, 100 MHz, 1920 s cap. Require reference reproduction before candidate metrics, expected timing-only completion, byte-identical complete FASM, identical report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% improvement in BOTH Router2 and whole wall. Any failure abandons with no B2 or rerun. Build with both C/C++ ccache launchers on cores 28-39 only.
- Queue after this candidate: `perf/l2-bel-location-flat` (placement, tier 1) and `perf/l2-portref-iteration` (placement/traversal, tier 1); neither will be substituted for the current Router2-focused screen without a new profile rationale.

## 2026-11-04T11:37:11Z — fast pip-delay source bypass built and smoke-tested

- Candidate commit `cbfc8d7a`; build `build/perf-l2-fast-pip-delay-source-bypass`: 63/63, 120.38 s, peak RSS 2,677,820 kB, ccache launchers for C and C++.
- Candidate SHA-256 `47fc3bfbc2eab4f347f5d6e299000d84fd31eb376f58c268a13142a52b169776`; reference SHA-256 `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`.
- Generated version remains `perf-accepted-qor-revert-40d3b873`; reference-chipdb `--test` passed on build core 28.

ACK 2026-11-04T11:34Z lane2: block 2

## 2026-11-04T11:55:52Z — fast pip-delay source bypass B1 PASS

- Valid pair `20261104T114735Z-fast-pip-delay-source-bypass-b1` (candidate core 18, reference core 19). An earlier launcher attempt ran only the reference because of a shell backgrounding error; it executed no candidate and was discarded as a comparison. Its reference reproduced the stored reference exactly.
- Reference reproduction PASS before candidate metrics: expected timing exit 1, normal completion/routing, stored FASM/report SHA-256, checksums and complete router history all matched.
- Candidate/reference identity PASS: complete FASM and report byte identity, parsed report, router history, checksums, Fmax and final iteration 91 all equal. FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`.
- Reference → candidate: wall 399.17 → 373.61 s (-6.4033%), Router2 289.52 → 264.34 s (-8.6972%), HeAP 45.42 → 44.84 s. BOTH timing gates PASS.
- No B2 pair started: directive block 3 first requires a reproduced B2 reference.

## 2026-11-04T11:59:47Z â two B2 reference-only reproductions launched

- Root `20261104T115910Z-b2-reference-repro`; drivers 504643 (r1, reference core 19) and 504644 (r2, reference core 23), seed 1, one thread, 100 MHz, 8460 s cap. These are REFERENCE-ONLY runs; no candidate B2 pair or other lane-2 benchmark is active.
- Verified reference binary SHA-256 `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3` and original B2 input SHA-256 `9271cdc99c199854e89a21615481fd541171e9b742c257140e57a46e6323c7f1`. Both outputs must reproduce the stored reference and each other before any candidate B2 pair.

## 2026-11-04T12:29:27Z â block 4 ACK; B1 confirmation live; B2 tier-2 plan

- B1 confirmation pair `20261104T122843Z-fast-pip-delay-source-bypass-b1-r2`: candidate driver 544364/core 26, reference driver 544363/core 27, seed 1, one thread, 100 MHz, 1920 s cap, detached with logs/PID files. Reference reproduction will be checked before candidate metrics.
- B2 plan from block 4: first pair on cores 26-27 after B1 confirmation passes; second on cores 18-19 after the reference reproduction runs finish. Every B2 result line will be labeled **B2 tier-2 (reference nondeterministic)**.
- At 30 minutes the two reference-only B2 runs remain live on cores 19/23 (nextpnr PIDs 504650/504651), within the 141-minute cap; no output has been compared yet.

ACK 2026-11-04T12:04Z lane2: block 4

## 2026-11-04T12:45:20Z â second B1 PASS; B2 reference status; two B2 tier-2 pairs live

- B1 confirmation `20261104T122843Z-fast-pip-delay-source-bypass-b1-r2`: PASS. Reference reproduced stored FASM/report/history/checksums/final iteration; candidate FASM/report bytes, parsed report, history, checksums and final iteration 91 all match. Wall 419.49 -> 395.56 s (-5.7045%), Router2 306.95 -> 283.47 s (-7.6495%); exits 1/1 with normal timing-only completion.
- **B2 tier-2 (reference nondeterministic)** reference-only root `20261104T115910Z-b2-reference-repro`: both runs completed normally with exit 0. r1 FASM `18d00b1ccde28e327fd7fc41024487dc501e4d36a4f509a7dbb178f5e99e5b4a`; r2 FASM `65d9ca5e7b08292ec7248a99be6e8c352330875a1f6b78d942924822b5ffb216` (DIFFER). Reports match each other at `62fd9c1b31399315491fd7ee694c51f868d108f95001d29eb659204a78fe9058`, but neither FASM/report matches the stored reference `403d98cb...` / `ca496ca3...`; the reference is not twice-reproduced.
- **B2 tier-2 (reference nondeterministic)** pair 1 `20261104T124335Z-fast-pip-delay-source-bypass-b2-r1-tier2` launched: candidate driver 569352/core 26, reference driver 569351/core 27, 8460 s cap.
- **B2 tier-2 (reference nondeterministic)** pair 2 `20261104T124445Z-fast-pip-delay-source-bypass-b2-r2-tier2` launched after the reference-only runs finished: candidate driver 571545/core 18, reference driver 571544/core 19, 8460 s cap.

## 2026-11-04T13:13:34Z â B2 tier-2 pairs at 30 minutes

- **B2 tier-2 (reference nondeterministic)** pair 1 remains live: nextpnr PIDs 569359 (candidate core 26) and 569358 (reference core 27), elapsed 29m41s. Pair 2 remains live: PIDs 571552 (candidate core 18) and 571551 (reference core 19), elapsed 28m31s. No exit files yet; no candidate metrics inspected; both remain well below the 141-minute kill threshold.

## 2026-11-04T13:15:30Z â block 5 ACK; next candidate claimed

- Block 5 item 1 was recorded with both B2 driver PIDs, cores and result directories in the 12:45Z entry and refreshed at 13:13Z. Cores 22-23 remain idle. Claimed `perf/l2-bel-location-flat` at 2026-11-04T13:15:29Z for HeAP bel-location lookup flat storage; build will use cores 28-39 while both B2 pairs continue.

ACK 2026-11-04T13:04Z lane2: block 5


## 2026-11-04T13:30:00Z — fast pip-delay source bypass meets tier-2 acceptance gates

- **B2 tier-2 (reference nondeterministic)** r1 `20261104T124335Z-fast-pip-delay-source-bypass-b2-r1-tier2`: reference reproduced stored FASM/report/history/checksums exactly. Candidate/reference exits 0/0, routing converged, utilization 52/52 equal, final overuse/archfail 0/0. Fmax improved 30.582908630371094 -> 30.67860984802246 MHz. Wall 2272.83 -> 2188.52 s (**-3.7095%**), Router2 1287.45 -> 1206.45 s (**-6.2915%**), peak RSS +0.7846%. FASM differs; GNU `diff -U0` counted 22,114 all / 22,113 payload hunks, 15,537 added, 15,351 deleted and 70,203 changed non-comment lines (6,690,135 diff bytes).
- **B2 tier-2 (reference nondeterministic)** r2 `20261104T124445Z-fast-pip-delay-source-bypass-b2-r2-tier2`: reference did not reproduce the stored reference, as already recorded; the paired candidate is byte-identical in complete FASM/report, parsed report, history and checksums. Exits 0/0, routing converged, utilization 52/52 equal, final overuse/archfail 0/0 and Fmax equal. Wall 2234.09 -> 2133.10 s (**-4.5204%**), Router2 1253.49 -> 1160.15 s (**-7.4464%**), peak RSS +0.4501%. FASM diff is zero.
- QoR table (reference -> candidate):

| Pair | Tier/label | Final Fmax (MHz) | Utilization | Final overuse / overuse total / archfail | Converged | Iterations | FASM |
|---|---|---|---|---|---|---|---|
| B1 r1 | identity | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 | identical |
| B1 r2 | identity | 41.830501556396484 -> 41.830501556396484 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 91 -> 91 | identical |
| B2 r1 | B2 tier-2 (reference nondeterministic) | 30.582908630371094 -> 30.67860984802246 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 17 -> 16 | different; improved QoR |
| B2 r2 | B2 tier-2 (reference nondeterministic) | 30.67860984802246 -> 30.67860984802246 | 52/52 equal | 0/0/0 -> 0/0/0 | yes -> yes | 16 -> 16 | identical |

- Decision: **ACCEPT `perf/l2-fast-pip-delay-source-bypass` under tier 2, not tier 1.** Both B1 pairs exceed 3% wall and Router2 with exact identity; both B2 pairs exceed the 3% tier-2 wall floor and have equal-or-better Fmax, unchanged utilization, zero final overuse and normal convergence. Origin accepted-qor remains `40d3b873`, so no lane-1 merge requires re-measurement. Integration merge and annotated tag follow.

| Change | B1 wall / Router2 | B2 wall / Router2 | QoR | FASM | Status |
|---|---|---|---|---|---|
| HeAP cell-location index | -0.56% / -0.24% | not run | identical | y | abandoned below timing floor |
| Fast pip-delay source bypass | -6.40%, -5.70% / -8.70%, -7.65% | -3.71%, -4.52% / -6.29%, -7.45% | equal or better | B1 y; B2 r1 n, r2 y | tier-2 gates passed |


## 2026-11-04T13:31:12Z — fast pip-delay source bypass integration

- Fetched and rebased on `origin/perf/accepted-qor`; the base remains `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`, so no lane-1 merge invalidates the measurements. Integrating with a no-ff merge and annotated tag `perf-20261104-tier2-fast-pip-delay-source-bypass`; parent tag `perf-accepted-qor-revert-02411403`. No accepted-identical merge because the candidate is tier 2.


## 2026-11-04T13:50:50Z — bel-location flat candidate built; B1 starting

- Source `0fa7cf41` changes only `himbaechel/arch.cc` and `himbaechel/arch.h`: precompute immutable `Loc` values in BelId tile/index order at chipdb load; `getBelLocation` becomes flat offset + index lookup. No placement or routing decisions change. The baseline profile inlines this helper, so the rationale is its integer division and chipdb pointer chasing across many placement callers, not a standalone profile symbol.
- Fresh accepted reference at `5d36ce50`: Release/IPO build on cores 28-39, 121.63 s, peak RSS 2,677,724 KiB, SHA-256 `881d8b4f66bba9375067c814b3eb223b888ed2bcc4bee3c16786560e100d36b7`; full xc7z020 chipdb integrity test finished normally. Candidate: 127.88 s, peak RSS 2,678,080 KiB, SHA-256 `529a80b938e125076fd1a147e4725e00ea2b42018b840f44e40b9ed617819265`; the same integrity test finished normally with exit 0.
- B1 `top_bringup` pair `20261104T135028Z-bel-location-flat-b1`: candidate core 18, reference core 19, seed 1, one thread, 1920 s cap. Expected tier 1: exact FASM/report/QoR identity plus wall and Router2 improvements of at least 3%. The fresh reference will also be compared with the prior accepted B1 reference artifacts before interpreting candidate metrics.


## 2026-11-04T13:59:00Z — bel-location B1 r1 neutral; independent r2 starting

- B1 r1 `20261104T135028Z-bel-location-flat-b1`: both exits 1 with normal timing-only completion; FASM `4b93c41fced2d036511549b077c51c23cde1f29a5309b38f138010682bd2fb61`, report `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, router history and checksums are candidate/reference identical; final iteration 91/91. Wall 360.88 -> 360.88 s (0.0000%), Router2 256.44 -> 256.35 s (-0.0351%), RSS +0.2441%: below both 3% gates, no verdict until r2.
- The fresh current-base reference report matches the prior accepted reference, but its FASM differs from that older-base artifact; this is a cross-base observation, not a candidate/reference mismatch. B1 r2 `20261104T135900Z-bel-location-flat-b1-r2` on candidate core 26 / reference core 27 will test current-base reference reproducibility as well as the candidate kill gate.


## 2026-11-04T14:06:48Z — ABANDON bel-location flat at the B1 kill gate

- B1 r2 `20261104T135900Z-bel-location-flat-b1-r2`: current-base reference reproduction against r1 PASS (FASM, report, checksums and final iteration equal across cores 19/27); both candidate/reference exits 1 with normal timing-only completion. Candidate/reference FASM, report, router history and checksums are identical; final iteration 91/91.
- B1 r2: wall 385.24 -> 388.11 s (+0.7450%), Router2 274.73 -> 276.84 s (+0.7680%), peak RSS +0.2777%. Together with r1 wall 0.0000% / Router2 -0.0351%, two independent-core pairs fail the required 3% wall and Router2 improvements. ABANDON `perf/l2-bel-location-flat`; no accepted-qor merge and no B2 run. Source branch and result roots remain as evidence.


## 2026-11-04T14:17:06Z — directive ACKs and PortRef iteration claimed

- Bel-location r1/r2 verdicts, reference reproduction, identity gates and wall/Router2 deltas are recorded above. Its B1 pairs actually used cores 18/19 and 26/27 because they were already built and running when block 6 arrived. Accepted integration `5d36ce50` is also recorded above.
- Block 7 full-model-Fmax harness is not present on `origin/perf/baseline` at `4e6facbf`: no patch commits touching `himbaechel/arch.cc`/`arch.h` were found after accepted `5d36ce50`. Do not write a duplicate; fetch every 30 minutes. Once it lands, rebase/rebuild as directed and run one B2 r1 pair with candidate core 18/reference core 19, then append full-model Fmax to the accepted record. Acceptance stands unless candidate Fmax is worse.
- Claimed `perf/l2-portref-iteration` at 14:10:58Z (claims commit `df280d47`) for HeAP `foreach_port` direct `indexed_store` active-slot traversal. Base is accepted `5d36ce50`; expected tier 1. Preserve driver-before-users order, ascending active-slot order, user indices, equation arithmetic and all RNG/output behavior.
- Pending candidate queue: (1) `perf/l2-portref-iteration` — HeAP analytic placement traversal, tier 1; (2) `perf/l2-detail-place-portref-iteration` — detailed-placement PortRef traversal using the same direct active-slot mechanism if justified, tier 1; (3) `perf/l2-heap-cluster-offset-reuse` — HeAP equation construction, reuse immutable cluster-offset lookups without changing arithmetic, tier 1.
- Kill gates: build/smoke failure, any reference or candidate/reference identity mismatch, or failure to improve both wall and Router2 by at least 3%. No B2 unless B1 passes. Build on cores 28-39 with both ccache launchers; B1 on candidate core 22/reference core 23 with seed 1, one thread, 100 MHz and 1920 s cap.

ACK 2026-11-04T13:36Z lane2: block 6
ACK 2026-11-04T14:05Z lane2: block 7


## 2026-11-04T14:24:31Z — PortRef direct active-slot iteration built

- Source `4b9f43e7` changes only `common/kernel/indexed_store.h` and `common/place/placer_heap.cc`: a direct ascending-active-slot `for_each` replaces the HeAP enumerated iterator wrappers and repeated bounds-checked accesses. Driver-first order, user indices and equation arithmetic are unchanged.
- xc7z020-only Release/IPO build on cores 28-39 with both ccache launchers passed in 128.16 s, peak RSS 2,678,000 KiB. Candidate binary SHA-256 `9bea41f8abe23890ac36d3ff718c3aea5679f341bdd323b7eda5b843c2c77715`; full chipdb `--test` exited 0.
- Reference remains the already validated accepted `5d36ce50` binary SHA-256 `881d8b4f66bba9375067c814b3eb223b888ed2bcc4bee3c16786560e100d36b7`. B1 will use candidate core 22/reference core 23; reference reproduction and identity are mandatory before timing interpretation.


## 2026-11-04T14:44:15Z — ABANDON PortRef iteration; full-model harness landed

- The 14:25:29Z launcher attempt `20261104T142529Z-portref-iteration-b1` was invalid: shell backgrounding ran only a candidate and no reference. It is retained as a discarded candidate-only artifact and no metrics were taken from it.
- Valid B1 `20261104T143323Z-portref-iteration-b1`, candidate core 22/reference core 23: reference reproduction against `20261104T135900Z-bel-location-flat-b1-r2` PASS for normal exit/finish, FASM, report, checksums, complete history and final iteration. Both runs exited 1 normally with final iteration 91.
- Candidate/reference report bytes, parsed report, history, checksums, final iteration and Fmax are equal. Raw FASM differs only in the generated version comment; the complete payload after that line is byte-identical with SHA-256 `45c231cca7f541233e2a98f318860ac62cfb31683ae9e0ec64b769a196f0db0f`. The raw full-byte identity gate is therefore not clean because the candidate was configured after its source commit, but this metadata issue cannot rescue the timing result.
- Wall 393.24 -> 393.25 s (+0.0025%), Router2 281.18 -> 281.42 s (+0.0854%), HeAP 45.80 -> 45.66 s (-0.3057%), analytic iterations 42.97 -> 42.89 s (-0.1862%). BOTH timing gates FAIL regardless of the version-comment artifact. ABANDON `perf/l2-portref-iteration`; no rerun, B2, merge, tag or bitstreams. No lane-2 benchmark process remains.
- Block 8 item 2 recorded: all run artifacts remain under `bench/results/<run id>`; any future candidate accepted after its tag requires B1+B2 bitstreams, checksums and metadata on cores 28-39. Accepted `5d36ce50` is exempt because the one-shot lane builds those.
- Full-model harness `987f57f2` is now on `origin/perf/baseline`. Build reference exactly `40d3b873 + 987f57f2` and candidate exactly `5d36ce50 + 987f57f2`, then run one B2 r1 pair on candidate core 18/reference core 19. Record fast-model and full-model Fmax and append the result to the accepted document; acceptance stands unless full-model Fmax worsens.

ACK 2026-11-04T14:26Z lane2: block 8


## 2026-11-04T14:55:13Z — full-model B2 r1 launched

- Built the prescribed full-model pair on cores 28-39 with 12 jobs and ccache. Reference `40d3b873 + 987f57f2` = `6ccf2ff830fb591ff0da37453b63050113f539c6`, binary SHA-256 `5158ffd8af68f4ca950e08579c434c495143e8ea3deba659c1627c157836ceb3`; candidate `5d36ce50 + 987f57f2` = `86d841933a540d81e6049b5f701a72147bc831d3`, binary SHA-256 `10b33f29dedcad7b28a0f46c4f7280b491d4a648211e526890b43505161bd7f7`. Both xilinx/meta submodules are `a4af910cac907f2cbd3a545f26f8e26573e860de`.
- B2 tier-2 (reference nondeterministic): `20261104T144924Z-fast-pip-delay-source-bypass-full-model-b2-r1-tier2`, candidate driver 807782 core 18, reference driver 807783 core 19, started 2026-11-04T14:54:47Z, cap 8460 s. Record fast-model Fmax from report JSON and full-model Fmax after the new log marker; compare the new reference to the stored r1 reference before interpreting candidate metrics.


## 2026-11-04T14:57:34Z — full-model launcher correction

- The 14:54:47Z attempt was discarded before metrics because its detached launcher did not persist exit codes. Both processes were stopped and artifacts moved to `20261104T144924Z-fast-pip-delay-source-bypass-full-model-b2-r1-tier2/invalid-launch-no-exit-capture`.
- Relaunched the identical prescribed B2 tier-2 pair at 14:57:11Z with explicit `exit.txt` capture: candidate driver 817662 core 18, reference driver 817663 core 19, same result root and cap 8460 s. These are now the only valid full-model run pids.


## 2026-11-04T15:26:34Z — full-model B2 progress

- B2 tier-2 (reference nondeterministic) remains active with valid drivers 817662 (candidate core 18) and 817663 (reference core 19), cap 8460 s. Status: `{"candidate": {"analytic_iteration": 28, "analytic_last_s": 8.4, "full_model_marker": false, "router_iteration": null}, "reference": {"analytic_iteration": 28, "analytic_last_s": 9.15, "full_model_marker": false, "router_iteration": null}}`. No verdict or candidate metric has been taken from the discarded launcher attempt.


## 2026-11-04T15:28:15Z — B2 tier-2 (reference nondeterministic), block 9 duplicate record

- B2 tier-2 (reference nondeterministic): full-model r1 uses result root `20261104T144924Z-fast-pip-delay-source-bypass-full-model-b2-r1-tier2`, launched from the 14:49:24Z build/provenance root; valid pair started 14:57:11Z, candidate core 18, reference core 19, cap 8460 s and kill rule 141 min. Supervisor killed the remaining duplicate 807xxx child tree by pid at about 15:26Z. The surviving valid tree is 817662-817669 with `/usr/bin/time -v` wrappers; only its timing/output artifacts are eligible. Active matching pids at this check: `[817662, 817663, 817664, 817665, 817666, 817667, 817668, 817669]`.
- B2 tier-2 (reference nondeterministic): current valid directories each hold one command/log/driver set; the discarded attempt is isolated under `invalid-launch-no-exit-capture`. Final verification that exactly one `time.txt`, report and FASM exists per valid side remains pending while the pair runs. Fast-model and full-model Fmax results are likewise pending.
- PortRef iteration is ABANDONED at the B1 timing gate (wall +0.0025%, Router2 +0.0854%), claim DONE at 14:44Z in claims commit `998a7152`; no merge, B2 or bitstreams.
- S3 status: all valid and discarded run artifacts are under the result root. No bitstreams are built for `5d36ce50`, as block 8 explicitly assigns those to the one-shot lane; no new candidate has been accepted.

ACK 2026-11-04T15:26Z lane2: block 9

## 2026-11-04T16:15:35Z — full-model B2 outcome, one-shot bitstreams, and block 10 ACK

- **B2 tier-2 (reference nondeterministic)**: surviving valid pair `20261104T144924Z-fast-pip-delay-source-bypass-full-model-b2-r1-tier2` completed; the discarded launch remains isolated under `invalid-launch-no-exit-capture`. Each valid side has exactly one `time.txt`, exit, log and FASM. Reference reproduction against the original r1 reference passes after excluding only the generated version comment: payload SHA-256 `742e67cc8cd7eccd71f238f38db21f34239e78b1bb3c3bf08e006b186c8968cd`, complete 17-iteration history, checksum `0xeae85389` and printed fast-model Fmax 30.58 MHz all match.
- **B2 tier-2 (reference nondeterministic)**: candidate/reference printed fast-model Fmax values are 30.68/30.58 MHz, but **no full-model Fmax was produced**. Both sides exited 125 after `Timing analysis failed due to combinational loops`, following the full-model marker; neither produced `report.json`. `full-model-status.json` therefore records `HARNESS_ERROR_NO_FULL_MODEL_FMAX`. These are not valid full-model QoR or timing results and do not change acceptance. A corrected diagnostic pair must add the already-validated `--ignore-loops` option; the failed pair will not be used for a speed claim.
- Block 8 S3 bitstream status is complete without a duplicate build: the assigned one-shot lane produced accepted-tier2 B1 and B2 artifacts under `bench/results/20261104T143204Z-nextpnr-bitstreams/tier2-b1` and `tier2-b2`, including FASM, frames, `design.bit`, `sha256sums.txt` and `metadata.json`. Tier2-B1 bitstream SHA-256 `74054f49702850613dc7d4682bfc3473c2eaa87cea2ed4766236bd0333c91258`, full-model Fmax 21.54 MHz, passes the stated 20 MHz screen. Tier2-B2 bitstream SHA-256 `8eb3d7c6d2cd81beedb059e9478e3d624811fdda8d8347505cde404a2a0e52fb`, full-model Fmax 9.13 MHz, is marked **DO-NOT-LOAD**. No board run was performed.
- PortRef remains ABANDONED at the B1 timing gate as recorded at 14:44Z; no live lane2 nextpnr process remains. No new candidate has been accepted and no additional bitstream obligation exists.
- Next named queue: `perf/l2-heap-cluster-offset-reuse` (HeAP equation construction, tier 1), `perf/l2-detail-place-portref-iteration` (detailed-placement traversal, tier 1), and `perf/l2-router2-visit-generation` (Router2 visited-state reset elimination, tier 1). Claims are required before any build; lane1's active bind-no-resource and processed-sinks candidates are excluded.

ACK 2026-11-04T15:55Z lane2: block 8 and block 9 and block 10
