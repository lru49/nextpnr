# Flat fast-mode pip-delay table

## Status

Named before implementation on 2026-11-04; no build or benchmark result yet.

## Base and branch

- Base: `40d3b873a9bbea1e0fbc399df5ee3c4eae3cbd39`
- Branch: `perf/pip-delay-flat-r2`
- Reference binary SHA-256: `c33194144895cd1a95098fe9ddfdfbb2c43a1c552e7c07e9b0ead979082fcdc3`

## Profile evidence and hypothesis

The old-base B1 profile attributes 10.78% whole-run and 12.80% Router2 self time to `Arch::getPipDelay`. During routing, `fast_pip_delays` makes the result independent of the mutable `drive_res` and `load_cap` dictionaries. The remaining chipdb timing data and normalized endpoint/node timing data are immutable.

Precompute the scalar delay of every pip once on entry to fast-delay routing, using the original arithmetic, and store it in a flat `std::vector<delay_t>` indexed by the base's existing `pip_tile_offsets`. A private helper retains the original uncached computation for non-fast-mode callers and for table initialization. Fast-mode `getPipDelay` then constructs the same `DelayQuad` from the cached scalar. The original implementation always returns `DelayQuad(total_delay)` or `DelayQuad(100)`, so no quadrant information is discarded.

This is a new implementation, not a rerun of `44471597`: it reuses the base's pip offsets and direct nested tile/pip initialization rather than adding a second offset table and iterating `getPips()`. Only `himbaechel/arch.h` and `himbaechel/arch.cc` are in scope. Route ordering, delay arithmetic, availability, bindings, costs, RNG and output generation remain unchanged.

## B1 kill criterion

Build once in Release/IPO mode with both C and C++ compiler launchers set to `ccache` and the reference version string. Run exactly one detached B1 pair on candidate core 4/reference core 5 with seed 1, one thread, 100 MHz and a 1920 s cap.

Before candidate metrics are inspected, the reference must reproduce:

- complete FASM SHA-256 `9885b5d11288445a488e53aa7e3059a1411fb886046d246a2d3289ad6cd30eb0`;
- report SHA-256 `48c54a90b33497a37ebe2a5e44d912c8e89b3552d840d5f0f9c3d9bed8fec2f0`;
- checksums `0x86b726c6/0xeb2c108e/0xeea7ec3a`;
- iteration 91, final route triple 0/0/0 and Fmax 41.83 MHz.

Abandon on a crash, timeout, non-timing error, non-convergence, any complete-FASM/report/history/checksum/QoR mismatch, or less than 3.0% paired improvement in either Router2 time or whole wall time. No second B1 pair or B2 run follows a failed screen. If it passes, acceptance still requires a second B1 pair and two B2 pairs under the existing tier gates.
