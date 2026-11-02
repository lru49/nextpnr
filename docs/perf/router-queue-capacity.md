# Router queue capacity retention

Source revision: `d56c470f523846ddfb5272c724cdac70b1781a51`.

## Change

- Derive the forward/backward queue type from `std::priority_queue` to expose the protected underlying vector.
- Clear that vector between arc searches instead of destroying and swapping the queue, retaining its allocation.
- Reserve `flat_wires.size()` entries in both queues when each routing thread context is created.

The patch changes allocation behavior only. It does not change queue elements, comparison, ordering, random-number consumption, search limits, or routing updates, so report JSON and FASM are expected to be identical. Maximum RSS is an explicit acceptance metric because up-front reservation can increase memory use.

## Build and validation

Release build with IPO, the same xc7z020 target and chipdb inputs as the unpatched companion build, and `CURRENT_GIT_VERSION=d56c470f`. Benchmark B1/B2 with seed 1, one worker thread, and a 100 MHz target on the same core as the companion build.

Results: pending.
