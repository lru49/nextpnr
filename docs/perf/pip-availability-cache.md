# Xilinx pip availability cache

## Hypothesis and scope

`XilinxImpl::is_pip_unavail` accounts for 3.81% of B2 whole-run self time (6.70% of Router2) and 5.80% of B1 whole-run self time (6.96% of Router2) in the existing profiles. Candidate `perf/pip-availability-cache-r2` at `fb31bd8ebac3524a5b3f5b6c55fbb5f2fd44fa1e` caches immutable per-(tile-type, pip-index) classifications while retaining all design-dependent placement, BUFR/BUFGCTRL, site, LUT and constant-driver checks. It is based on reverted `perf/accepted-qor` head `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39` and changes only `himbaechel/uarch/xilinx/xilinx.cc` and `.h`.

The change must preserve routing order, costs, reports, checksums, QoR and complete FASM bytes. The B1 screen requires at least 3.0% improvement in both Router2 and whole wall time. Acceptance additionally requires two B2 pairs and the existing two-pairs-each wall/QoR gates.

## Build

The Release/IPO build used both C and C++ `ccache` launchers. After initializing the fresh worktree's pinned metadata submodule, the same build tree completed 61/61 steps in 128.38 s (133.14 s including the 4.76 s failed setup), with 2,677,660 KiB peak RSS. Candidate binary SHA-256 is `6e1d7b9b4d9b2c9d360c9dbdac0a64b6f3c8c574a1a115a37befabf0643c3bce`; reference binary SHA-256 is `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`.

## B1 screen

Artifacts: `bench/results/20261104T055114Z-pip-availability-cache-r2-b1`. Candidate/reference used cores 0/1, concurrency 2, seed 1, one thread, 100 MHz and a 1920 s cap. Both runs completed routing and exited 1 only because the design does not meet its 100 MHz timing target; comparison exited 0.

| Metric | Reference | Candidate | Change |
|---|---:|---:|---:|
| Whole wall (s) | 399.41 | 340.68 | -14.7042% |
| Router2 (s) | 291.99 | 231.34 | -20.7713% |
| SA (s) | 55.01 | 55.71 | +1.2725% |
| HeAP (s) | 45.57 | 46.73 | +2.5455% |
| Analytic iteration sum (s) | 42.96 | 44.07 | +2.5838% |
| Peak RSS (KiB) | 1,558,148 | 1,558,220 | +0.0046% |

Fmax is 41.83 MHz on both sides, router iteration is 91, and final overuse is 0. Report bytes and parsed reports, router history, checksums and complete FASM are identical. FASM SHA-256 is `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`.

**B1 screen: PASS. This did not establish acceptance; the subsequent B2 identity gate failed.**

## B2 identity gate and decision

Artifacts: `bench/results/20261104T060300Z-pip-availability-cache-r2-b2`. Both pairs used seed 1, one thread, 100 MHz, concurrency 4 and an 8460 s cap. r1 used candidate/reference cores 0/1 and r2 used cores 2/3. All runs and comparisons exited 0.

| Metric | r1 reference | r1 candidate | r1 change | r2 reference | r2 candidate | r2 change |
|---|---:|---:|---:|---:|---:|---:|
| Whole wall (s) | 2256.61 | 1998.65 | -11.4313% | 2236.23 | 1949.59 | -12.8180% |
| Router2 (s) | 1282.44 | 1021.96 | -20.3113% | 1267.24 | 979.05 | -22.7415% |
| Final router iteration | 17 | 19 | +2 | 17 | 19 | +2 |
| Fmax (MHz) | 30.582909 | 30.810944 | +0.228035 | 30.582909 | 30.678610 | +0.095701 |
| Final overuse | 0 | 0 | 0 | 0 | 0 | 0 |

Both pairs fail complete-FASM, report-byte, parsed-report, router-history and checksum identity. Reference FASM SHA-256 is `403d98cb07094b313a0a075284e12c8371ff32d444a87b99707f12ff98c5ba80` in both pairs; candidate hashes are `17a009ade34ef5cd246625da7c9a3d35d08eb314a3c24dc556a2bd25581da5a6` (r1) and `323664d4d0f0773302e7e0e0cbe6378fcd5c9c9f8dda8ee4b798b9da2441c23c` (r2). Reference final checksums repeat as `0xc55c66c4/0x6deffc74/0xeae85389`; candidate final checksums are `0xc55c66c4/0x6deffc74/0xe5ed6db8` and `0xc55c66c4/0x6deffc74/0xea90f316`. FASM diffs contain 104391 additions/104341 deletions (r1) and 108033/108325 (r2).

**Decision: ABANDON `fb31bd8e`. The wall improvements exceed 3%, but mandatory B2 FASM/report/history/checksum identity fails and candidate output differs across repeats. No second B1 pair, merge or tag is warranted.**
