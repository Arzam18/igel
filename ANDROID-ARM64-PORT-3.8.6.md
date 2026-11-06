# Igel 3.8.6 — Android ARM64 NEON port

This tree uses the supplied upstream Igel 3.8.6 source as its base. The supplied
Igel 3.8.5 Android workflow and NEON implementation were used as the porting
reference; the 3.8.6 engine/search/NNUE changes are retained.

## Android build

Run `.github/workflows/android.yml` manually (`workflow_dispatch`). It targets
Android API 24 / ARM64, Android NDK r27c (27.2.12479018), ARMv8-A + NEON, and
uses the standard upstream `5adc00b8.standard` NNUE network embedded at build
time. The workflow verifies the AArch64 ELF, executable size, dynamic linkage,
and NEON instruction presence, then uploads one UCI executable.

## 3.8.6 ARM portability changes

- Restored `USE_NEON` compile selection and 16-byte SIMD width in `nnue.h`.
- Ported the 3.8.5 NEON kernels to the 3.8.6 accumulator, feature transform,
  NNUE input-layer and clipped-activation code.
- Kept 3.8.6's accumulator refresh-cache optimization; its 32-slot comparison
  is scalar/ISA-neutral on ARM.
- Restored Android-safe 2 MiB aligned allocation via `posix_memalign`.
- Made AVX2-only helpers and startup assertions conditional; the UCI banner
  reports `NEON` (not x86 `POPCNT`/`BMI2`) on the ARM64 build.
- Corrected the legacy platform-width macro to recognize `__aarch64__` as 64-bit.
- Added an Android-specific CMake branch so host `-mavx2` options are not used
  for the Android target.

## Verification boundary

This archive is a port candidate, not a confirmed Android build. A native x86-64 CMake Release build has passed. The Android GitHub Actions
build and device tests must still be run. Do not
mark Igel 3.8.6 successful until the produced binary passes UCI startup and
search on the target Android device.
