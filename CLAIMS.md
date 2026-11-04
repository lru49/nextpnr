# nextpnr performance claims

Append one claim before building. Release it with DONE or ABANDONED.
CLAIM 2026-11-04T10:45:24Z lane2 perf/l2-heap-cell-location-index HeAP/build_equations+solve_equations+total_hpwl
CLAIM 2026-11-04T10:52:00Z lane1 perf/no-resource-fastpath-r1 Router2/route_arc+getResourceKeyForPip
ABANDONED 2026-11-04T11:12:14Z lane2 perf/l2-heap-cell-location-index B1 identity pass; wall and Router2 gains below 3 percent
ABANDONED 2026-11-04T11:18:00Z lane1 perf/no-resource-fastpath-r1 B1 identity PASS; wall +0.2999%, Router2 +0.0951%, both timing gates FAIL
CLAIM 2026-11-04T11:27:09Z lane2 perf/l2-fast-pip-delay-source-bypass Himbaechel/Arch::getPipDelay fast-mode source-RC bypass
CLAIM 2026-11-04T11:26:00Z lane1 perf/visit-data-access-r1 Router2/was_visited-access-reuse
