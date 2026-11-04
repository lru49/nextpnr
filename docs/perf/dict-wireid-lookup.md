# Flat WireId-to-resource lookup

## Hypothesis and scope

Candidate `perf/dict-wireid-lookup` at `dda46ae2f33d9593bda1501d4887816d1f3b0361` is based on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. The B1 profile attributes 16.5% of whole-run self time to `dict<WireId,int>::do_lookup`; Router2 repeatedly uses `wire_to_resource` to translate a wire into its chipdb resource index.

Under `ARCH_HIMBAECHEL`, the candidate replaces that dictionary with a flat `std::vector<int>` indexed through the base's existing `tile_wire_offsets`. Entries initialize to -1 and use first-resource-wins setup; other architectures retain the dictionary path. Only `common/route/router2.cc` changes. Routing order, costs and output generation are unchanged.

## Kill criterion and acceptance gates

One detached B1 pair runs with candidate on core 6 and reference on core 7, using seed 1, one thread, 100 MHz and a 1920 s cap. It requires normal routing completion, byte-identical complete FASM, identical report bytes and parsed reports, router history, checksums, Fmax, utilization and final overuse, plus at least 3.0% paired improvement in both Router2 time and whole wall time. Any failure abandons the candidate with no B2.

## Build

The source branch is pushed. After initializing the pinned metadata submodule and supplying the reference prjxray database and `HIMBAECHEL_XILINX_DEVICES=xc7z020` settings, the Release/IPO build completed 63/63 steps in 216.81 s with 2,678,144 KiB peak RSS. It was pinned to cores 40-63 at `-j2`; both C and C++ compiler launchers are `ccache`. Version/help smoke checks passed with the reference version string.

Candidate binary SHA-256 is `81bfd4d42f218e653f218be78119d4b06893d507b5c0064daa66c2be703e4ea8`; reference binary SHA-256 is `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`. The build tree is `nextpnr-dict-wireid/build/perf-dict-wireid-lookup`.

## B1 result and decision

Pair `20261104T083700Z-dict-wireid-lookup-b1` ran concurrently with two pip-delay B2 pairs, for six benchmark runs in total. Before candidate metrics were inspected, the reference reproduced its stored FASM/report hashes, checksums, router history and Fmax exactly. Both sides then completed routing normally with timing-only exit 1; comparison exit was 0.

| Metric | Reference | Candidate | Change |
| --- | ---: | ---: | ---: |
| Whole wall | 425.48 s | 435.14 s | **+2.2704%** |
| Router2 | 317.78 s | 325.93 s | **+2.5647%** |
| User time | 424.37 s | 434.02 s | +2.2740% |
| Analytic iteration sum | — | — | +0.7258% |
| HeAP | — | — | +0.7502% |
| SA | — | — | +2.0493% |
| Peak RSS | 1,558,048 KiB | 1,600,500 KiB | +2.7247% |

Complete FASM is byte-identical with SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`. Report SHA-256 is `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`. Report bytes and parsed reports, full router history and checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a` also match. Both sides have final Fmax 41.83 MHz, router iteration 91 and final overuse 0.

**ABANDONED:** identity and QoR pass, but both required timing metrics regress instead of improving by at least 3.0%. No second B1 pair, B2 pair, merge or accepted tag will be performed.
