# Router2 thread wire bounds-test inlining r1

## Status

**BUILT; B1 pending.** Candidate `perf/thread-test-wire-inline-r1` source `3060c4c6e154cc54782f2cf03635aefda8ca2095` is based directly on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. This is a semantics-preserving performance candidate, expected tier 1.

## Hypothesis and scope

Remove the trivial `thread_test_wire` helper and write its four bounding-box comparisons directly at the two hot forward/backward expansion call sites. Preserve the exact comparison and short-circuit order, `nwd`/`ThreadContext` objects, filtering, scoring, traversal, queue behavior, RNG and route decisions. The stale profile does not isolate this helper and promises no gain; the paired B1 screen is decisive.

## Kill criterion and acceptance gates

Build with Release/IPO, xc7z020 only, both C/C++ compiler launchers set to `ccache`, and the pinned generated version; new build invocations use cores 40-55 only. Record source/binary hashes, build wall and verified version.

Run one detached B1 pair on cores 10/11 with reference reproduction before candidate metrics, seed 1, one thread, 100 MHz and a 1920 s cap. Require expected timing-only exit 1, normal routing/program completion, byte-identical complete FASM/report, identical parsed report/history/checksums/Fmax/utilization/final overuse, and at least 3.0% paired improvement in both Router2 and whole-wall time. Any failure abandons the candidate with no B2, rerun, merge or tag. B2 performance testing remains blocked while the reference is nondeterministic.

## Build record

Source `3060c4c6e154cc54782f2cf03635aefda8ca2095` is pushed directly over the required base with a 4-insertion/7-deletion diff. The recovered configure/build had already completed successfully in 1.06/109.40 s, with maximum build RSS 2,677,960 KiB; its logs record cores 40-63. A subsequent validation invocation on cores 40-55 completed in 0.02 s with `ninja: no work to do` and left the binary unchanged. CMakeCache verifies Release, IPO ON, xc7z020-only, both C/C++ `ccache` launchers and `CURRENT_GIT_VERSION=perf-accepted-qor-revert-40d3b873`; `--version` reports the same pinned version. Candidate binary SHA-256 is `68c54448a2fa19a153170e0f3108d9e85e6439e2af75e6a2a6b982c1abde812b`. Base ancestry and `git diff --check` passed; only untracked build timing logs exist in the candidate worktree.
