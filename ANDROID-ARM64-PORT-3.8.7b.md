# Igel 3.8.7b — Android ARM64 NEON and Dotprod port

This tree is based on the supplied upstream Igel 3.8.7b source. Its new NUMA
support and 3.8.7b network-loading lifecycle remain intact.

## Android builds

Run `.github/workflows/android.yml` manually (`workflow_dispatch`). It builds
and uploads two self-contained ARM64 executables using Android NDK r27c and
API 24:

- `Igel-3.8.7b-android-arm64-neon`: portable ARM64 baseline with mandatory NEON.
- `Igel-3.8.7b-android-arm64-dotprod`: ARMv8.2-A Dotprod build using SDOT.

The Dotprod executable requires an ARM64 CPU implementing the Dotprod extension;
use the baseline NEON executable on older ARM64 devices.

## NNUE arithmetic and Dotprod eligibility

All three affine layers use signed 8-bit weights (`std::int8_t`). The feature
transformer clamps both accumulator perspectives to `[0, 127]`. Both clipped
activation functions also clamp their outputs to `[0, 127]`. Therefore each
activation byte can be reinterpreted as signed without changing its value, and
`SDOT` computes exactly the same `int32` dot product as the widening NEON
multiply/accumulate path. No weight quantization, rounding, saturation, or
network-file conversion is introduced by the Dotprod implementation.

The 16-bit feature accumulator and transformer remain on NEON; Dotprod is used
for the int8-by-int8 affine layers. The baseline NEON build keeps the original
widening multiply/accumulate implementation.

## Portability and workflow fix

- The 32-slot accumulator refresh-cache comparison is scalar and ISA-neutral;
  this avoids x86 `__m256i` references in ARM builds.
- AVX2-only helper declarations are compiled only when their matching x86
  backend is selected.
- Android transformer allocation uses `posix_memalign` for the 2 MiB-aligned
  transformer storage.
- CMake selects `armv8-a+simd` for baseline builds and
  `armv8.2-a+dotprod` only when `USE_DOTPROD=ON`.
- The NDK CMake toolchain is allowed to select its generic `clang++` driver;
  the workflow no longer compares that path to the API-specific compiler
  wrapper. The toolchain file carries the Android target and ABI settings.
- Fix4 removes brittle text matches against `CMakeCache.txt` and `build.ninja`
  that could fail after successful generation but before compilation. Each
  build step now traces commands and preserves the full verbose compiler log.
- The final ELF checks verify `AArch64`, Android's `/system/bin/linker64`
  interpreter, static C++ runtime linkage, and NEON instructions. The Dotprod
  executable must additionally contain an actual `SDOT` instruction.

## Verification boundary

The exact 3.8.7b source and Android workflow are packaged here. The NDK Actions
build must still confirm both final ELF files and SDOT disassembly. DroidFish
startup, NNUE load, search, NPS, and playing-strength tests must be performed
on-device before marking this port complete.
