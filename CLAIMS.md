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
