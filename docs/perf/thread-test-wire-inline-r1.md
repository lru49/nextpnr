# Router2 thread wire bounds-test inlining r1

## Status

**ABANDONED; B1 identity/QoR PASS, both timing gates FAIL.** Candidate `perf/thread-test-wire-inline-r1` source `3060c4c6e154cc54782f2cf03635aefda8ca2095` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. This was a semantics-preserving performance candidate, expected tier 1. No B2, rerun, merge or tag.

## Hypothesis and scope

Remove the trivial `thread_test_wire` helper and write its four bounding-box comparisons directly at the two hot forward/backward expansion call sites. Preserve the exact comparison and short-circuit order, `nwd`/`ThreadContext` objects, filtering, scoring, traversal, queue behavior, RNG and route decisions. The stale profile does not isolate this helper and promises no gain; the paired B1 screen is decisive.

## Kill criterion and acceptance gates

Build with Release/IPO, xc7z020 only, both C/C++ compiler launchers set to `ccache`, and the pinned generated version; new build invocations use cores 40-55 only. Record source/binary hashes, build wall and verified version.

Run one detached B1 pair on cores 10/11 with reference reproduction before candidate metrics, seed 1, one thread, 100 MHz and a 1920 s cap. Require expected timing-only exit 1, normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% paired improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag. B2 performance testing remains blocked while the reference is nondeterministic.

## Build record

Source `3060c4c6e154cc54782f2cf03635aefda8ca2095` is pushed directly over the required base with a 4-insertion/7-deletion diff. The recovered configure/build had already completed successfully in 1.06/109.40 s, with maximum build RSS 2,677,960 KiB; its logs record cores 40-63. A subsequent validation invocation on cores 40-55 completed in 0.02 s with `ninja: no work to do` and left the binary unchanged. CMakeCache verifies Release, IPO ON, xc7z020-only, both C/C++ `ccache` launchers and `CURRENT_GIT_VERSION=perf-accepted-qor-revert-40d3b873`; `--version` reports the same pinned version. Candidate binary SHA-256 is `68c54448a2fa19a153170e0f3108d9e85e6439e2af75e6a2a6b982c1abde812b`. Base ancestry and `git diff --check` passed; only untracked build timing logs exist in the candidate worktree.

## B1 verdict — 2026-11-04T15:23Z

Result dir: `bench/results/20261104T151000Z-thread-test-wire-inline-r1-b1`. Candidate/reference ran on cores 10/11 at concurrency **1 pair = 2 processes**, seed 1, one thread, 100 MHz, with no other pair or reference-only run. Both exited 1 after routing and normal program completion; the reference reproduced every stored FASM/report/checksum/history/Fmax gate before comparison.

Identity/QoR passed: complete FASM and report bytes matched, parsed report/utilization/Fmax, all 91 router-history rows, checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`, final iteration 91 and overuse 0 matched. FASM SHA-256 is `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`; report SHA-256 is `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`; final aclk Fmax is 41.83 MHz for both.

Reference → candidate whole wall was **431.94 → 428.99 s (-0.6830%)** and Router2 was **318.24 → 317.27 s (-0.3048%)**. Both are below the required 3.0% improvement, so **ABANDON**. The analytic iteration sum improved by 3.0138%, but that is not a timing gate and does not change the verdict.
