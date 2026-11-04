# Router2 to-go cost PerNetData/PerWireData passing r1

## Status

**ABANDONED: B1 identity/QoR PASS, both timing gates FAIL; no B2, rerun, merge or tag.** Candidate `perf/togo-wiredata-reference-r1` source `2b4c340597c54b5b29338ddc03d9586b78b5a52f` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. This was a semantics-preserving performance candidate, expected tier 1.

## Hypothesis and scope

`get_togo_cost` currently resolves `PerNetData` from `net->udata` and `PerWireData` from an integer wire index on every call, although `route_arc` already has `nd` and either the seed wire or hot-expansion `nwd`. Pass those existing objects directly at all four call sites. Keep the existing `nd.wires.find(wd.w)`, source-use calculation, delay estimation, floating-point arithmetic, queue order, RNG and route decisions unchanged.

This is distinct from abandoned `route-cost-source-uses-r1`, which reused `fnd_wire` and shared source-use values between scoring and to-go estimation, and from abandoned `score-netdata-reference-r1`, which changed only arc scoring. No visited-check or route-backtrace change is included.

## Kill criterion and acceptance gates

Build once on cores 40-63 with Release/IPO, xc7z020 only, both C/C++ compiler launchers set to `ccache`, and the pinned generated version. Record source/binary hashes, build wall and verified version.

Run one detached B1 pair only when its assigned cores are free, with reference reproduction before candidate metrics. Require expected timing-only exit 1, normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% paired improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag. B2 performance testing remains blocked while the reference is nondeterministic.

## Build and B1 launch record

Source `2b4c340597c54b5b29338ddc03d9586b78b5a52f` is pushed with a 5-insertion/7-deletion diff. Configure/build completed on cores 40-63 in 1.00/105.84 s, with maximum build RSS 2,678,020 KiB. CMakeCache verifies Release, IPO ON, xc7z020-only, both C/C++ `ccache` launchers and the pinned generated version; `--version` reports `perf-accepted-qor-revert-40d3b873`. Candidate binary SHA-256 is `0b77afb060b799a2eb23f3870fb1fd876c75cf53765360b68cd337252676ff05`. Base ancestry and `git diff --check` passed.

Detached B1 driver 45939 started at 14:13:31Z in `bench/results/20261104T141200Z-togo-wiredata-reference-r1-b1`, with candidate nextpnr 45967 on core 6 and reference nextpnr 45964 on core 7. Seed 1, one thread, 100 MHz and a 1920 s cap are in force; all hash guards passed and reference reproduction precedes candidate metrics. The visited pair on cores 2-3 completed before this launch, so the assignments do not overlap and this is currently one pair/two processes.

## B1 verdict

The pair completed with expected timing-only exits 1/1 and reference-reproduction/comparison exits 0/0. Reference reproduction passed every stored constant before candidate metrics were inspected. Complete FASM/report bytes, parsed report, full router history, checksums, Fmax, utilization and final overuse matched at iteration 91. Whole wall changed from 406.39 to 408.01 s (+0.3986%) and Router2 from 299.54 to 301.38 s (+0.6143%): the candidate was slower and both 3.0% improvement gates failed. It is abandoned with no B2, rerun, merge or tag.
