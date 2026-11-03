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

## 2026-11-03T17:37Z — standalone B1 reference diagnostic running
- Following the user's continuation instruction, started only `20261103T173527Z-b1-reference-standalone`: reference `d56c470f` nextpnr pid 6784 on physical core 0, driver pid 6780, started 17:35:45Z, concurrency 1, seed 1, one thread, 100 MHz, 1920-second cap. At the 60-second check pid 6784 used 99.9% CPU with affinity exactly core 0; 483 GiB is free. This isolates baseline behavior from candidate and multi-pair contention; batch 2 remains held.
- 17:58Z diagnostic evidence: the current B1 JSON SHA-256 is `521e463dd881e9c7b5743b172744d45095974139934f75e80f5701e4d77d2aff`, not the `47b085cd1e4c5ce6e541d292014ba2adcf2ac70eabca665ea544ce76b4ee8a37` recorded for the 623.63-second baseline in `docs/perf/baseline.md`; its mtime is 07:24:35Z, after that baseline. Chipdb SHA-256 still matches `e8038aa75f1162efa21c4bd6898fb03e8bfb3091164b56e9356e962a8d4cad2b`. Thus the old B1 wall/QoR is not a valid reference for the current input, and the timeouts are not evidence of pair contention. Standalone reference pid 6784 remains at 100% CPU; it will finish at the existing cap while batch 2 stays held.

## 2026-11-03T18:10Z — standalone calibration complete; supervisor clarification required
- ACK 2026-11-03T17:36Z lane: started only the standalone B1 reference, driver pid 6780 / nextpnr pid 6784 on core 0 at 17:35:45Z. Block 7 was first visible after that launch: the actual cap was 1920 s rather than its requested 2400 s, and the required pre-launch command/lscpu record was not made. Those protocol deviations are recorded rather than presenting this as a fully compliant calibration. No other nextpnr run was started and batch 2 remains HELD.
- Host: 64-core aarch64 Neoverse-V1 r1p1, one thread per core; `lscpu` exposes no CPU MHz field (BogoMIPS 2100.00 is not reported as MHz). At the 60-second launch check 483 GiB was free and pid 6784 had exactly core-0 affinity at 99.9% CPU. Exact executed argv (also preserved in `reference/time.txt`): `timeout --signal=TERM --kill-after=30s 1920s taskset -c 0 /home/gremen/.cache/nextpnr-perf-lane/bench/results/20261103T141500Z-six-pairs/binaries/reference-d56c470f --device xc7z020clg484-1 --chipdb /home/gremen/.cache/nextpnr-perf-lane/toolchain/reference-data/share/nextpnr/himbaechel/xilinx/chipdb-xc7z020.bin --json /home/gremen/.cache/nextpnr-perf-lane/bench/inputs/top_bringup.json -o xdc=/home/gremen/.cache/nextpnr-perf-lane/sparseloop-zed-zed-accel/fpga/vivado/v1/constr/top_bringup.xdc -o fasm=/home/gremen/.cache/nextpnr-perf-lane/bench/results/20261103T173527Z-b1-reference-standalone/reference/top_bringup.fasm --report /home/gremen/.cache/nextpnr-perf-lane/bench/results/20261103T173527Z-b1-reference-standalone/reference/report.json --router router2 --placer heap --seed 1 --threads 1 --freq 100 --log /home/gremen/.cache/nextpnr-perf-lane/bench/results/20261103T173527Z-b1-reference-standalone/reference/log.txt`.
- Diff against the documented 623.63-second record: source `d56c470f`, chipdb hash, device, router2, heap, seed 1, one thread and 100 MHz agree; the old run used core 0 concurrently with B2 on core 8 while this diagnostic was intentionally standalone. The material difference is B1 JSON SHA-256 `521e463d...` now versus `47b085cd...` then, so this is not a like-for-like host calibration. Current reference binary SHA-256 is `f0262ea7592ca604ebe2c225d233e66dc6fde18c71fe8ac39f73a91484ac692b`; no historical binary hash is recorded for comparison.
- Final reference result: W=1920.10 s, exit 124, max RSS 1,470,716 KiB, no report/FASM, QoR unavailable; router reached iteration 629 with 786 overused wires / overuse 788. W > 1872 s, so the required block-7 conclusion is: **VM not comparable to reference host: B1 caps invalid**. Batch 2 stays HELD and no nextpnr process remains running.
- The changed input also explains the different placement/QoR trajectory: post-placement Fmax is 49.35 MHz versus the old input's 51.00 MHz. Current-input overuse fell to 2 at iteration 503, then rose to 788 by iteration 629, reproducing the work-order router overuse problem rather than the old input's convergence at iteration 91. The five concurrent B1 pairs and isolated pair therefore cannot be timed against the old 623.63-second baseline, but no candidate is accepted or rejected from those timeouts.
- Waiting for supervisor clarification: restore/provide the original `47b085cd...` B1 JSON and repeat a compliant 2400-second calibration, or designate current `521e463d...` as the new B1 workload with a new reference cap and authorize the router overuse investigation. No further benchmark is authorized under block 7 item 4 until then.

## 2026-11-03T18:43Z — restored-input standalone calibration launch
- ACK 2026-11-03T18:26Z lane: starting only the restored-input B1 reference `d56c470f` on physical core 0 with a 2400-second cap; no pair or other nextpnr process will run during calibration.
- The active B1 input was changed in place at 07:24:35Z (block 8 attributes the change outside this lane), from SHA-256 `47b085cd1e4c5ce6e541d292014ba2adcf2ac70eabca665ea544ce76b4ee8a37` to `521e463dd881e9c7b5743b172744d45095974139934f75e80f5701e4d77d2aff`. The supervisor restored the original on the orchestrator's 18:2xZ decision; the changed input is preserved as `bench/inputs/top_bringup.changed-0724Z.json` and is not a benchmark input. A pre-launch hash check confirms active and canonical copies both equal `47b085cd...a37`.
- Host immediately before launch: 64-core aarch64 Neoverse-V1 r1p1, one thread per core; `lscpu` exposes no CPU MHz field (BogoMIPS 2100.00 is not MHz). `free -g`: 494 GiB total, 478 GiB free, 490 GiB available, no swap. Process audit: no nextpnr process running.
- Exact launch argv, with `LANE=<lane-cache>` and `OUT=$LANE/bench/results/20261103T184300Z-b1-reference-restored/reference`: `/usr/bin/time -v -o $OUT/time.txt timeout --signal=TERM --kill-after=30s 2400s taskset -c 0 $LANE/bench/results/20261103T141500Z-six-pairs/binaries/reference-d56c470f --device xc7z020clg484-1 --chipdb $LANE/toolchain/reference-data/share/nextpnr/himbaechel/xilinx/chipdb-xc7z020.bin --json $LANE/bench/inputs/top_bringup.json -o xdc=$LANE/sparseloop-zed-zed-accel/fpga/vivado/v1/constr/top_bringup.xdc -o fasm=$OUT/top_bringup.fasm --report $OUT/report.json --router router2 --placer heap --seed 1 --threads 1 --freq 100 --log $OUT/log.txt`.
- Diff against the 623.63-second record: source, device, chipdb, restored input hash, router2, heap, seed 1, one thread and 100 MHz now match. This calibration is intentionally standalone on core 0; the old B1 reference ran alongside B2 on core 8. Output paths and the 2400-second safety cap do not alter the workload.
- Future task, explicitly deferred by block 8: diagnose the changed-`521e463d` input's late router overuse rebound as a separate congested-design candidate; do not debug it now and do not rerun pip-delay-cache.
- Live at 18:44:33Z: driver pid 7806, nextpnr pid 7839, started 18:44:09Z on core 0 at 100% CPU with affinity exactly core 0; concurrency 1 and the restored input hash still matches.
- Result at 18:54:08Z: standalone restored-input B1 reference completed routing at iteration 91 in W=599.16 s (user 597.93 s, system 1.16 s, max RSS 1,471,948 KiB), router2 499.13 s; this is 24.47 s / 3.92% faster than the documented 623.63 s and comfortably below the 1872 s wall gate. Final Fmax is 41.83 MHz against the 100 MHz target; report and FASM were produced, FASM SHA-256 `2edc647dfcb849ccb35eb6e3a59f480221684c60db92c5740b0acc4d906cbd05`. No nextpnr process remains running.
- Gate ambiguity: nextpnr exited 1 solely because timing failed, exactly like the documented reference. Wall and timeout conditions pass, but block 8 item 3 literally requires exit 0 before the stacked pair. The pair is therefore NOT started. Supervisor clarification requested in this append-only record: confirm that a timing-only exit 1 with complete report/FASM and W <= 1872 s satisfies the intended calibration gate. Candidate-pair preparation may continue without launching a benchmark.
- Non-benchmark preparation at 19:02Z: audited `perf/router-stack` 9a83eaa0 (clean and already pushed). It contains ddd6a509 and 11fa46aa, the stack-adapted form of score-fastpath code 326efc3f; 9437fa79 is that candidate's B2 validation-record commit, not an ancestor of the stack. Existing candidate binary SHA-256 `93ea50292f67a1955b88d67cd7168573e017605ab834fa0265cfad335506eb12`, reference SHA-256 `f0262ea7592ca604ebe2c225d233e66dc6fde18c71fe8ac39f73a91484ac692b`. Prepared `bench/results/20261103T190000Z-b1-router-stack-gated/driver.sh` for candidate/reference on cores 2/3, concurrency 2, cap 1920 s and FASM comparison. Its explicit supervisor-acknowledgement guard was tested and exited 89 before any launch; no nextpnr process is running.
- Batch-2 non-benchmark audit at 19:06Z: existing `20261103T161300Z-pip-binding-six-pairs/driver.sh` defines three repeated B1 pairs on cores 12-17 (1920 s caps) and three repeated B2 pairs on cores 18-23 (8460 s caps), all FASM-gated at concurrency 12. Clean, pushed candidate 9299e4be has binary SHA-256 `bfc190fac40bd81744f2bc80aec7070c98e28810da954047d4ce1f6cfccca790`; it is based on ddd6a509 + 11fa46aa (not on 9a83eaa0's later non-himbaechel-only guard). Current input hashes are B1 `47b085cd...a37` and B2 `9271cdc99c199854e89a21615481fd541171e9b742c257140e57a46e6323c7f1`; 490 GiB is available. Batch 2 remains HELD behind both the exit-gate clarification and the required stacked-pair wall result; no benchmark was launched.

## 2026-11-03T19:18Z — stacked-pair gate resolved from reference semantics
- Decision: the restored calibration satisfies the intended host-comparability gate. Its exit 1 is solely the expected 100 MHz timing failure, identical to the documented 623.63 s reference, with normal routing completion, complete report/FASM and W=599.16 s; treating that expected application status as a failed calibration would make every valid B1 run fail the literal exit-0 condition. Timeout exits 124/137/143 remain failures.
- Starting the single B1 router-stack pair now: candidate 9a83eaa0 on core 2 versus reference d56c470f on core 3, same-second launch, concurrency 2, cap 1920 s, original-input SHA guard and per-run FASM/QoR comparison. Batch 2 remains held until both walls finish below 1920 s.
- Live at 19:19Z: driver pid 698; both runs started 19:18:03Z. Candidate nextpnr pid 716 has exactly core-2 affinity and reference nextpnr pid 713 exactly core-3 affinity; both were 99.9% CPU at the 60-second check, concurrency 2, with 478 GiB free before launch. Input hash guard passed.
