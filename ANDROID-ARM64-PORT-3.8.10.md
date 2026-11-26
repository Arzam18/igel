# Igel 3.8.10 Android ARM64 NEON / DotProd port

## Source baseline

- Upstream source: Igel 3.8.10.
- Upstream snapshot commit: `d176e251b330daa8920e0cce96f583bd87c1353e`.
- Older Android source snapshot: `8dae53a77c8152224184892bdf12e43c3c7e96aa` (3.8.8).
- Android target: `arm64-v8a`, API 24, Android NDK r27c (`27.2.12479018`).
- Baseline ISA: ARMv8-A NEON (`-march=armv8-a+simd`).
- Optimized ISA: ARMv8.2-A DotProd (`-march=armv8.2-a+dotprod`).
- C++ runtime: static libc++.
- NNUE network: `820573bc.universal`, matching the network declaration in the new 3.8.10 makefile.

## Port design

The upstream 3.8.10 engine and network format remain authoritative. The older 3.8.8 Android implementation was used only as a reference for the ARM algorithms that the new upstream snapshot removed. The port restores architecture guards and AArch64 NEON kernels for accumulator delta updates, transformer feature output, dense NNUE affine layers, clipped activations, and square activations.

The 3.8.10 AVX2 sparse input-layout transformation and sparse propagation remain x86-only. Android retains the original row-major NNUE weights and uses a dense NEON affine path, so the serialized model is not rewritten. The 3.8.10 refresh cache, network-generation invalidation, NUMA-disabled Android single-network flow, and upstream search/history changes are preserved.

The NEON dot-product helper interprets activation bytes as signed only after the network's activation functions have clipped them to `[0,127]`; weights are signed int8. This makes `vdotq_s32` exact for those NNUE affine layers. Baseline builds use widening multiply-accumulate instead. DotProd is a separate binary and requires the ARMv8.2-A DotProd compiler feature.

## Workflow

`.github/workflows/android.yml` is manual-only (`workflow_dispatch`). It accepts a required `uci_date` build identity and an overridable network URL. It produces separate baseline NEON and DotProd artifacts, verifies AArch64 ELF identity, static libc++ linkage, architecture instructions, and records SHA-256 checksums.

## Validation performed and remaining work

Passed locally:

- Full x86-64 AVX2 host compile and link using GCC 14.2 with a one-byte fixture network. This is a compile/link check only; the fixture is not a valid NNUE network.
- Android-specific CMake branch fixture configuration; it defines `USE_NEON` / optional `USE_DOTPROD` and excludes x86-only build flags.
- Focused AArch64 intrinsic cross-compilation for baseline NEON and DotProd. Baseline disassembly showed NEON widening multiply/arithmetic and no SDOT; DotProd disassembly showed SDOT.
- Workflow YAML parsing, `bash -n` on all shell blocks, and build-date macro concatenation.

Still pending:

- Full Android NDK r27c compilation/link in GitHub Actions.
- Loading the actual `820573bc.universal` network, UCI runtime tests on Android, NPS, thermals and playing-strength testing.

These stages are distinct: host compilation and focused intrinsic tests do not prove a complete Android binary or a working on-device network load. The workflow is manual-only (`workflow_dispatch`).
