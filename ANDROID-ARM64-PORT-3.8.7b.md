# Igel 3.8.7b — Android ARM64 NEON port

This port is re-applied directly to the supplied upstream **Igel 3.8.7b** tree.
It is not a merge of the older 3.8.6 engine sources. The 3.8.7b search, NUMA
support, per-node NNUE replication, universal network format, and UCI options
remain the upstream 3.8.7b implementations.

## Android build

Run `.github/workflows/android.yml` manually with `workflow_dispatch`. It uses
Android NDK r27c (`27.2.12479018`), API 24, ARM64 (`arm64-v8a`), and the baseline
ARMv8-A NEON instruction set. The workflow downloads the exact upstream
`820573bc.universal` network named by the 3.8.7b source makefile (the older
`5adc00b8.standard` reference still present in parts of the README/other CI is
not used by this Android workflow), records its SHA-256, embeds it at build
time, checks the AArch64 ELF and
NEON instructions, and publishes only the UCI executable (plus its checksum).

## ARM64 portability changes

- Restored the 3.8.6 ARM NEON implementation in the 3.8.7b NNUE source: threat-
  aware accumulator deltas and PSQT updates, transformer output packing,
  unsigned-activation/signed-weight affine accumulation, and both clipped
  activation functions.
- Kept the 3.8.7b network-copy lifecycle (`NetCopy`, `ThreadNet`,
  `Evaluator::replicateNet()` and `Evaluator::useNet()`) intact. Android does
  not enable `NUMA_BINDING`, so it uses the upstream single-node path without
  Linux CPU-affinity calls.
- Made the NNUE intrinsic headers conditional: `<arm_neon.h>` for `USE_NEON`,
  `<immintrin.h>` for x86 builds.
- Restored 16-byte NEON SIMD width and Android-safe 2 MiB aligned allocation
  via `posix_memalign`.
- Corrected the 64-bit platform test to include `__aarch64__`, allowed the
  NEON architecture assertion, and reports `64 NEON` in the UCI identity.
- Added an Android-specific CMake branch so x86 AVX2/AVX-512/BMI2 compiler
  flags are not passed to the Android compiler.

## Fidelity and verification boundary

The NEON kernels use the same integer accumulator, clipping, and NNUE layer
operations as the upstream x86 code; they do not quantize or replace the
network. The `USE_NEON` implementation is baseline ARMv8-A SIMD, not Dotprod.

The supplied ZIP contains no `820573bc.universal` network file, and this local
runtime has neither GitHub network access nor Android NDK r27c. Host HCE
CMake/UCI startup/readiness passed, as did a full host compile of the NNUE
AVX2 path. A standalone AArch64 Clang intrinsic probe emitted the expected
NEON instructions; it is not a full engine cross-compile. Actual Android
cross-compilation, disassembly verification, Android runtime, NPS, and playing
strength remain pending the GitHub Actions run and device test. Do not mark
Igel 3.8.7b complete until those checks pass.
