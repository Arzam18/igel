# Igel 3.8.8 Android ARM64 port

## Target

- Engine source identity: Igel 3.8.8 (`src/uci.cpp`).
- Android NDK: r27c, 27.2.12479018; API 24; ABI `arm64-v8a`.
- Standard build: ARMv8-A NEON (`armv8-a+simd`).
- Optimized build: ARMv8.2-A Dotprod (`armv8.2-a+dotprod`).
- C++ runtime: static libc++.
- Default network candidate: `6b953e78`, the standard network named in the official Igel 3.8.0 release notes. The workflow exposes `network_url` so a revision-specific network can be supplied; the engine startup check rejects an incompatible file.

## Port design

The 3.8.8 NNUE implementation remains the source of truth. The port adds architecture guards and ARM NEON implementations for transformer accumulator deltas, feature transformation, affine layers, and clipped activation functions. The per-thread accumulator refresh cache and 3.8.8 network-generation invalidation remain in place. Android continues to use the non-NUMA single-network path selected by `src/numa.h`.

The new 3.8.8 sparse first-layer AVX2 layout is retained only for x86 AVX2 builds. ARM builds do not reorder the serialized row-major weights and use a NEON dense first-layer dot product. This is mathematically equivalent but does not yet reproduce the x86 sparse-kernel optimization.

Dotprod uses signed `vdotq_s32`. Its inputs are the unsigned activations clipped to `[0,127]`, reinterpreted as signed bytes without changing their values; NNUE weights are signed `int8_t`. This makes each signed byte product exact. The baseline NEON build uses widening multiply-accumulate instead.

## Workflow validation

The workflow creates two separate artifacts and checks that both are AArch64 ELF64 binaries. It rejects `libc++_shared.so`, checks for NEON arithmetic in the baseline build, and requires an `sdot` instruction in the Dotprod build. It also records the fetched network checksum and the engine binary checksum.

## Validation status

Source-level implementation has been added, but a successful NDK cross-build, Android runtime test, NPS measurement, and strength test have not yet been completed in this environment. Those remain required after the GitHub Actions workflow runs. The default network comes from the official 3.8.0 release announcement, not from the uploaded 3.8.8 source itself. The workflow therefore exposes `network_url` as an override and performs a real engine-startup check to reject a network with an incompatible header. Confirm the correct network for this source revision before relying on the artifact.
