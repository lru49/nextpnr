# nextpnr performance claims

Append one claim before building. Release it with DONE or ABANDONED.
CLAIM 2026-11-04T10:45:24Z lane2 perf/l2-heap-cell-location-index HeAP/build_equations+solve_equations+total_hpwl
CLAIM 2026-11-04T10:52:00Z lane1 perf/no-resource-fastpath-r1 Router2/route_arc+getResourceKeyForPip
ABANDONED 2026-11-04T11:12:14Z lane2 perf/l2-heap-cell-location-index B1 identity pass; wall and Router2 gains below 3 percent
ABANDONED 2026-11-04T11:18:00Z lane1 perf/no-resource-fastpath-r1 B1 identity PASS; wall +0.2999%, Router2 +0.0951%, both timing gates FAIL
CLAIM 2026-11-04T11:27:09Z lane2 perf/l2-fast-pip-delay-source-bypass Himbaechel/Arch::getPipDelay fast-mode source-RC bypass
CLAIM 2026-11-04T11:26:00Z lane1 perf/visit-data-access-r1 Router2/was_visited-access-reuse
CLAIM 2026-11-04T11:40:00Z lane1 perf/route-cost-source-uses-r1 Router2/score_wire_for_arc+get_togo_cost
ABANDONED 2026-11-04T11:46:00Z lane1 perf/visit-data-access-r1 B1 identity PASS; wall +1.2530%, Router2 +1.2417%, both timing gates FAIL
ABANDONED 2026-11-04T12:00:00Z lane1 perf/route-cost-source-uses-r1 B1 identity PASS; wall -0.4633%, Router2 -0.6800%, both timing gates FAIL
CLAIM 2026-11-04T12:01:00Z lane1 perf/reset-wires-reference-r1 Router2/reset_wires-access-reuse
CLAIM 2026-11-04T12:04Z lane1 b2-determinism-root-cause Router2 per-iteration (rip-up order, heap pops, hash iteration)
ABANDONED 2026-11-04T12:25:00Z lane1 perf/reset-wires-reference-r1 B1 identity PASS; wall -0.8022%, Router2 -0.9540%, both timing gates FAIL
CLAIM 2026-11-04T12:25:00Z lane1 perf/score-netdata-reference-r1 Router2/route_arc+score_wire_for_arc
CLAIM 2026-11-04T12:34Z lane1 b2-determinism-root-cause: suspect = threaded Router2 (NPNR_DISABLE_THREADS)
ABANDONED 2026-11-04T12:52:00Z lane1 perf/score-netdata-reference-r1 B1 identity PASS; wall -0.6445%, Router2 -0.6192%, both timing gates FAIL
CLAIM 2026-11-04T13:08:00Z lane1 perf/router2-existing-wire-bb-guard-r1 Router2/route_net+ripup_arc threaded existing-wire partition invariant
CLAIM 2026-11-04T13:15:29Z lane2 perf/l2-bel-location-flat HeAP bel-location lookup flat storage
CLAIM 2026-11-04T13:16:00Z lane1 perf/route-backtrace-reference-r1 Router2/route_arc forward+backward backtrace access reuse
DONE 2026-11-04T13:32:08Z lane2 perf/l2-fast-pip-delay-source-bypass tier2 accepted 5d36ce50; B1 wall -6.4033/-5.7045 Router2 -8.6972/-7.6495; B2 tier2 wall -3.7095/-4.5204 Router2 -6.2915/-7.4464
ABANDONED 2026-11-04T13:37:00Z lane1 perf/route-backtrace-reference-r1 B1 identity PASS; wall -0.9221%, Router2 -0.7558%, both timing gates FAIL
CLAIM 2026-11-04T13:37:00Z lane1 perf/visited-check-wiredata-r1 Router2/was_visited-access-reuse
ABANDONED 2026-11-04T13:54:00Z lane1 perf/router2-existing-wire-bb-guard-r1 B1 identity FAIL; iteration 91 -> 88, FASM/report/history/checksums differ; no B2 or rerun
CLAIM 2026-11-04T14:02:00Z lane1 perf/togo-wiredata-reference-r1 Router2/get_togo_cost PerNetData+PerWireData passing
DONE 2026-11-04T14:07:13Z lane2 perf/l2-bel-location-flat ABANDONED at B1 gate; r1 wall 0.0000 Router2 -0.0351, r2 wall +0.7450 Router2 +0.7680; exact outputs, no merge
CLAIM 2026-11-04T14:10:58Z lane2 perf/l2-portref-iteration HeAP foreach_port direct indexed_store active-slot traversal
ABANDONED 2026-11-04T14:18:00Z lane1 perf/visited-check-wiredata-r1 B1 identity PASS; wall -0.7199%, Router2 -0.8894%, both timing gates FAIL
ABANDONED 2026-11-04T14:23:00Z lane1 perf/togo-wiredata-reference-r1 B1 identity PASS; wall +0.3986%, Router2 +0.6143%, both timing gates FAIL
CLAIM 2026-11-04T14:23:00Z lane1 perf/thread-test-wire-inline-r1 Router2/thread_test_wire direct bbox comparisons at two hot call sites
DONE 2026-11-04T14:44:16Z lane2 perf/l2-portref-iteration ABANDONED at B1 timing gate; wall +0.0025%, Router2 +0.0854%; report/history/FASM payload equal, raw version comment differs; no merge
ABANDONED 2026-11-04T15:23:00Z lane1 perf/thread-test-wire-inline-r1 B1 identity PASS; wall -0.6830%, Router2 -0.3048%, both timing gates FAIL
