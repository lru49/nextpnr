# nextpnr-perf-lane progress

Append-only record. Times are UTC.

## 2026-11-02 21:25 — baseline run started
- Checked out `d56c470f523846ddfb5272c724cdac70b1781a51` as `perf/baseline`.
- Built the profiling binary successfully (`Release`, `-O2 -g -fno-omit-frame-pointer`).
- Launched the stripped reference on B1 and B2, seed 1, one thread, 100 MHz.

## 2026-11-02 21:36 — reference B1 complete
- Wall 623.63 s; maximum RSS 1,471,152 KiB; exit 1 because the 100 MHz constraint failed.
- Router2 converged to zero overuse at iteration 91 in 516.35 s; the previously observed iteration-700 plateau did not reproduce with the current B1 JSON.
- Post-placement Fmax 51.00 MHz; final Fmax 41.83 MHz. FASM SHA-256: `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`.
- B2 is still running; its baseline and all profile data remain pending.

## 2026-11-02 21:50 — B1 profile complete
- Instrumented B1: 671.59 s wall, 557.15 s router2, 1,471,396 KiB maximum RSS, 665K samples, zero lost samples.
- Fmax, utilization, and all 91 router iterations match the reference. FASM payload after the version header is byte-identical; future compatibility builds will pin the reference version string.
- Recorded top-30 self-time symbols for whole run, analytic placement, simulated annealing, and router2 in `docs/perf/baseline-b1-profile-top30.txt`.
- Router profile leaders: `dict<WireId,int>::do_lookup` 19.68%, `Arch::getPipDelay` 12.80%, `dict<WireId,Loc>::do_lookup` 7.16%, and `score_wire_for_arc` 6.27%.

## 2026-11-02 22:40 — B2 baseline and profile complete
- Reference B2: 2814.37 s wall, 2,291,876 KiB maximum RSS, 857.63 s analytic iterations, 268.67 s SA, and 1650.34 s router2; exit 1 is timing-only.
- Instrumented B2: 2933.84 s wall, 1683.82 s router2, 2,291,836 KiB maximum RSS, and about 2 million samples.
- Both runs converged at router iteration 20 with post-place/final Fmax 36.24/31.76 MHz. Report JSON is identical and FASM payload after the version header is byte-identical.
- Recorded whole-run and per-phase top-30 symbols in `docs/perf/baseline-b2-profile-top30.txt`; router hash/delay lookups remain the leaders.

## 2026-11-03 00:10 — Router2 scoring fast path validated
- Branch `perf/router-score-fastpath`, implementation `326efc3f`; B1/B2 results are in `docs/perf/router-score-fastpath.md` on that branch.
- B1 companion-build wall 602.19 -> 553.27 s (-8.12%); Router2 494.45 -> 451.00 s (-8.79%); RSS +0.02%; 91 iterations and QoR unchanged.
- B2 companion-build wall 2666.79 -> 2563.22 s (-3.88%); Router2 1519.09 -> 1400.03 s (-7.84%); RSS +0.01%; 20 iterations and QoR unchanged.
- Both report JSON files and complete FASM files are byte-identical. Accepted on the topic branch, not yet merged to fork main. These are single-pair measurements; the patched B2 placement overlapped another validation, making its wall delta conservative.

## 2026-11-03 00:10 — Router2 queue-capacity candidate rejected
- Branch `perf/router-queue-capacity`, implementation `e68901e8`; results are in `docs/perf/router-queue-capacity.md` on that branch.
- B1 wall 603.82 -> 628.03 s (+3.98%); Router2 501.42 -> 519.38 s (+3.58%); RSS +0.02%; 91 iterations and QoR unchanged.
- Report JSON and complete FASM are identical, but the candidate regressed runtime and was abandoned without a B2 run.

## 2026-11-03 00:02 — Flat WireId index validation started
- Built and pushed `d538000d` on `perf/router-wire-index`; it replaces the hot `dict<WireId, int>` with tile offsets and a flat integer vector while preserving `flat_wires` order.
- Sequential B1 companion-build comparison is running on core 24; results are pending.

## 2026-11-03 00:10 — Optimization summary
| Change | B1 wall (s) | B2 wall (s) | QoR delta | FASM identical | Status |
| --- | ---: | ---: | --- | :---: | --- |
| Reference baseline | 623.63 | 2814.37 | baseline | n/a | measured concurrently |
| Router scoring fast path | 602.19 -> 553.27 | 2666.79 -> 2563.22 | none | yes | accepted on topic branch |
| Router queue capacity | 603.82 -> 628.03 | not run | none | yes | abandoned: runtime regression |
| Flat WireId index | running | pending | pending | pending | validation running |

## 2026-11-03 00:25 — Flat WireId index passes B1
- `d538000d` wall 578.45 -> 492.63 s (-14.84%); Router2 480.84 -> 395.47 s (-17.75%); maximum RSS 1,471,088 -> 1,268,560 KiB (-13.77%).
- Router iterations (91), post-place/final Fmax (51.00/41.83 MHz), report JSON, and complete FASM are unchanged.
- B1 results committed as `0611ad3c`; sequential B2 validation is running on core 24.

## 2026-11-03 01:40 — Flat WireId index passes B2
- `d538000d` wall 2662.71 -> 2399.76 s (-9.88%); Router2 1521.34 -> 1260.58 s (-17.14%); maximum RSS 2,292,424 -> 2,106,284 KiB (-8.12%).
- Router iterations (20), post-place/final Fmax (36.24/31.76 MHz), report JSON, and complete FASM are unchanged. FASM SHA-256: `5464b504d810fdc3b50d6ee3f87e51f0a20cf1a16025cd5ec747951e4dd1f97b`.
- B1/B2 results committed as `ddd6a509`; the candidate is accepted on `perf/router-wire-index`.

## 2026-11-03 01:40 — Updated optimization summary
| Change | B1 wall (s) | B2 wall (s) | QoR delta | FASM identical | Status |
| --- | ---: | ---: | --- | :---: | --- |
| Reference baseline | 623.63 | 2814.37 | baseline | n/a | measured concurrently |
| Router scoring fast path | 602.19 -> 553.27 | 2666.79 -> 2563.22 | none | yes | accepted on topic branch |
| Router queue capacity | 603.82 -> 628.03 | not run | none | yes | abandoned: runtime regression |
| Flat WireId index | 578.45 -> 492.63 | 2662.71 -> 2399.76 | none | yes | accepted on topic branch |

## 2026-11-03T12:46Z — directive acknowledgement and recovery
- ACK 2026-11-03T11:56Z lane: recovered the canonical progress and am preparing the stacked accepted candidates and four profile-driven B1 candidates for six core-pinned pairs with 3x timeouts and per-pair FASM gates.
- Pip-delay-cache B1: abandoned: runtime > 3x reference. Stored pair `20261103T091100Z-b1-pip-pair` records candidate exit 143 after the 32-minute timeout; no nextpnr process remains in this lane namespace. Do not rerun this candidate.

## 2026-11-03T14:56Z — parallel-pair directive acknowledgements
- ACK 2026-11-03T14:09Z lane: started batch 1 at 14:15:20Z with B1 router stack on reference/candidate cores 0/1, B2 router stack on 2/3, B1 wire-location index on 4/5, B1 bel-bucket index on 6/7, B1 pip-binding index on 8/9, and B1 pip-availability cache on 10/11; all runs use seed 1, one thread, 100 MHz, per-run metadata, and the prescribed 1920/8460-second timeouts.
- ACK orchestrator notes 2026-11-03T13:30Z and 2026-11-03T13:59Z lane: retained the canonical append-only progress log and detached pid-based benchmark records, and am pushing every candidate branch tip.
- ACK 2026-11-03T14:40Z lane: acknowledging the 14:09Z batch-1 block above; batch 1 remains on its launched core layout and reference `d56c470f`, with no mid-batch changes and process control by pid only.

## 2026-11-03T15:38Z — supervisor relaunch recovery
- ACK 2026-11-03T15:07Z lane: batch 1 has no live pids; I am relaunching the six pairs on cores 0-11 with 1920/8460-second timeouts, per-run FASM and QoR gates, and progress updates at least every 30 minutes.
- ACK 2026-11-03T15:37Z lane: committing the canonical progress first, then starting B1/B2 router stack, B1 wire-location index, B1 bel-bucket index, B1 pip-binding index, and B1 pip-availability cache as six detached candidate/baseline pairs on cores 0-11; checking every pid after 60 seconds.

## 2026-11-03T15:53Z — batch 1 recovery running
- Recovered the 14:15Z attempt before relaunch: all five B1 pairs hit the 1920-second cap on BOTH reference and candidate (exit 124, concurrency 12, no FASM/report), so they are inconclusive rather than candidate regressions. Its B2 router-stack pair completed on candidate/reference cores 3/2: wall 2855.52/3509.34 s (-18.63%), RSS 2,133,804/2,315,620 KiB, exit 0/0, router iterations 19/20, identical report/Fmax 30.54 MHz, but FASM mismatch (`ebb152b7...` vs `7ace2985...`), so the stack failed that run's semantics-preserving gate.
- Started `20261103T154908Z-six-pairs-recovery` at 15:50:41Z with 12 live nextpnr pids after 60 seconds, concurrency 12, driver pid 5485, and 472 GiB free. Candidate/baseline core and nextpnr pid pairs: B1 router stack 0/5582 vs 1/5584; B2 router stack 2/5580 vs 3/5592; B1 wire-location 4/5589 vs 5/5591; B1 bel-bucket 6/5590 vs 7/5586; B1 pip-binding 8/5587 vs 9/5588; B1 pip-availability 10/5593 vs 11/5594. Every run has command/launcher pid files and 1920/8460-second caps.
- Fork branch tips verified by `git ls-remote`: pip-binding-index `9299e4be`, router-wire-index `ddd6a509`, router-score-fastpath `9437fa79`; all were already pushed. Five independent Kimi audits found every candidate worktree clean and every binary executable.
- 16:12Z heartbeat: all 12 nextpnr pids remain live at 100% CPU after 21 minutes, no pair has finished, and 471 GiB is free. The five B1 pairs remain under their 1920-second caps; B2 continues under its 8460-second cap.
- 16:22Z B1 router stack (candidate/reference cores 0/1, concurrency 12): wall 1920.23/1920.27 s, peak RSS 1,266,892/1,470,976 KiB, exit 124/124, FASM match no (neither FASM produced), QoR delta unavailable. B1 run abandoned: runtime > 3x reference; the reference timed out too. B2 remains in flight.
- 16:22Z B1 wire-location-index (cores 4/5, concurrency 12): wall 1920.28/1920.27 s, peak RSS 1,520,348/1,470,716 KiB, exit 124/124, FASM match no (neither FASM produced), QoR delta unavailable. Abandoned: runtime > 3x reference; the reference timed out too.
- 16:22Z B1 bel-bucket-index (cores 6/7, concurrency 12): wall 1920.24/1920.27 s, peak RSS 1,269,244/1,471,096 KiB, exit 124/124, FASM match no (neither FASM produced), QoR delta unavailable. Abandoned: runtime > 3x reference; the reference timed out too.
- 16:22Z B1 pip-binding-index (cores 8/9, concurrency 12): wall 1920.28/1920.27 s, peak RSS 1,556,992/1,471,204 KiB, exit 124/124, FASM match no (neither FASM produced), QoR delta unavailable. This run is abandoned: runtime > 3x reference; the reference timed out too. The separately directed six-pair batch 2 remains queued.
- 16:22Z B1 pip-availability-cache (cores 10/11, concurrency 12): wall 1920.23/1920.27 s, peak RSS 1,267,180/1,471,284 KiB, exit 124/124, FASM match no (neither FASM produced), QoR delta unavailable. Abandoned: runtime > 3x reference; the reference timed out too.
- 16:48Z B2 router stack (candidate/reference cores 2/3, concurrency 12): wall 2855.96/3481.06 s (-17.957%), peak RSS 2,110,636/2,316,336 KiB, exit 0/0, FASM SHA-256 7ace29851ee494f4c8e4af634486daf16f6c37dd5c986f01e825d7628e25db72 on both (match yes), QoR delta 0 (final Fmax 30.54 MHz, final iteration 20, byte-identical report and equal router history). The previous mismatched candidate FASM is not reproduced. B1 was abandoned at its cap, so this is not a combined B1/B2 acceptance.
- 16:48Z batch 1 complete: baseline pid 5592 was live and finished normally at 16:48:42Z, so the B2 pair is valid. Six pair attempts took 3481 s: 6.21 pair attempts/hour; five attempts timed out on both sides and one completed. Post-batch VM load average was 0.19/0.83/2.83 with 483 GiB free.
- ACK 2026-11-03T16:36Z lane: lscpu reports 64 Neoverse-V1 cores with one thread per core (no SMT siblings); starting one pip-binding-index B1 contention pair on distinct physical cores 0/1 at concurrency 2 with a 1920 s cap. Batch 2 remains held until the baseline wall is known.
- 16:53Z contention check live: driver pid 15458; candidate nextpnr pid 15500 on core 0 and reference pid 15501 on core 1, both started 16:52:17Z and using 100% CPU at the 60-second check. Two nextpnr runs total; 482 GiB free. Batch 2 has not started.
- 17:21Z heartbeat: driver pid 15458 remains live; candidate pid 15500 (core 0) and reference pid 15501 (core 1), both started 16:52:17Z, remain at 100% CPU after 29 minutes. Two runs total, 481 GiB free, no FASM yet; batch 2 remains held pending the contention result.
- 17:24Z B1 pip-binding contention result (candidate/reference cores 0/1, concurrency 2): wall 1920.11/1920.10 s, peak RSS 1,556,996/1,470,720 KiB, exit 124/124, FASM match no (neither FASM produced), QoR delta unavailable. Run abandoned at the cap; this is not a candidate regression because the reference also timed out. Measured reference walls are 1920.27 s in each five-way-B1 portion of batch 1 and 1920.10 s in this isolated pair, versus the previously documented 623.63 s standalone reference. Per the 16:36Z directive, batch 2 is NOT started; no nextpnr runs remain, and the lane waits for the supervisor.
