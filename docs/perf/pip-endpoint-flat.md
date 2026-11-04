# Himbaechel flat pip-endpoint lookup

## Hypothesis and scope

Candidate `perf/pip-endpoint-flat` is based on `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`. Its source scope is only `himbaechel/arch.h` and `himbaechel/arch.cc`.

At the base, `Arch::getPipSrcWire` and `Arch::getPipDstWire` resolve `chip_pip_info` and call `normalise_wire` for every query. The old-base B1/B2 whole-run profiles attribute 0.77%/0.81% self time to `getPipSrcWire`, 0.56%/0.56% to `getPipDstWire`, 0.76%/1.01% to `chip_pip_info`, and 1.93%/1.25% to `normalise_wire`. The last two functions also have other callers, so these figures establish only a plausible upper bound, not a predicted gain.

The candidate precomputes normalised source and destination wires once after chipdb loading. Per-tile prefix offsets index one flat vector of source/destination `WireId` pairs, and only the two endpoint accessors change to direct lookups. The table is immutable after initialization and has no design-dependent or lazy state. Delay arithmetic, availability and binding logic, traversal order, costs and returned wire values are unchanged. This is not a reapplication of the abandoned pip-delay, pip-availability, pip-binding or router pip-location candidates.

## Kill criterion and acceptance gates

The first screen is one detached B1 pair with candidate on core 0 and reference on core 1, using seed 1, one thread, 100 MHz and a 1920 s cap. It requires:

- normal routing completion;
- byte-identical complete FASM;
- identical report bytes and parsed reports, router history, checksums, Fmax, utilization and final overuse;
- at least 3.0% paired improvement in both Router2 time and whole wall time.

Any identity/QoR failure, non-convergence or timing improvement below 3.0% in either metric abandons the candidate with no B2. A passing screen requires a second B1 pair and two B2 pairs under the existing tier gates before any merge or tag.

Before candidate metrics are read, the reference side must reproduce complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0` and checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a` from the preceding isolated B1 pairs.

## Status

Named before implementation and build on 2026-11-04. No benchmark result or acceptance claim exists yet.
