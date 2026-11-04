# Router2 scoring no-resource lookup guard r1

## Status

B1 running under block 43. Source `01dd83e17a95a7f0ee5de00a526a1d86f94fd291` is pushed directly over the stated base with only the planned `score_wire_for_arc` guard (7 insertions/5 deletions).

Release/IPO xc7z020-only configure/build completed on cores 40-55 in 1.04/114.34 s with both C/C++ `ccache` launchers and maximum build RSS 2,677,980 KiB. Generated `--version` is `perf-accepted-qor-revert-40d3b873`; binary SHA-256 is `68f3e268c127c0351aedad1687c0f7fd85dfdd0e66db38ca39a8b8b9b00c2711`. Base ancestry, the tracked source diff and `git diff --check` passed.

The guarded B1 pair started at 15:39:57Z in `bench/results/20261104T153500Z-score-no-resource-r1-b1`. Detached driver 9643 runs candidate nextpnr 9672 on core 10 and reference nextpnr 9669 on core 11, seed 1, one thread, 100 MHz and a 1920 s cap. Launch concurrency is one pair (two nextpnr processes), with no reference-only run. All input/binary/chipdb guards passed; reference reproduction precedes candidate comparison.

## Base and hypothesis

Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`, without any prior candidate patch. Touched phase: Router2 `score_wire_for_arc`; expected tier 1.

`setup_resources` populates `wire_to_resource` once before routing. On Xilinx it remains empty, so the `wire_to_resource.find(wire)` in `score_wire_for_arc` always misses and both resource-cost terms remain zero. Guard that lookup and its dependent calculations with `!wire_to_resource.empty()`. Architectures with resources execute the original lookup and arithmetic unchanged.

This is deliberately distinct from abandoned `perf/no-resource-fastpath-r1`, which guarded the two per-pip resource-conflict blocks in `route_arc`. This candidate changes only the per-scoring-call wire/resource dictionary lookup; it does not rerun or modify that candidate, change resource accounting, scoring arithmetic, traversal, queue operations, bindings or RNG. Avoiding a guaranteed-missing dictionary lookup may reduce Router2 and whole-wall time; the stale profile supplies no promised gain.

## Kill criterion and validation

Build once from the stated base in Release/IPO mode for xc7z020 only, with both C and C++ compiler launchers set to `ccache`, pinned to cores 40-55, using generated version `perf-accepted-qor-revert-40d3b873`. Verify source ancestry, binary SHA-256, clean tracked diff and `git diff --check`.

Run one detached B1 pair on candidate core 10 and verified-reference core 11, seed 1, one thread, 100 MHz and a 1920 s cap, with no other pair. Before candidate metrics are read, the reference must reproduce complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`, report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, history `91/284240/0/0/0` and Fmax 41.83 MHz.

Require expected timing-only exit 1 after normal routing/program completion, byte-identical complete FASM and report bytes, identical parsed report, full router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 and whole-wall time. Any identity, QoR or timing failure abandons the candidate with no B2, rerun, merge or tag.
