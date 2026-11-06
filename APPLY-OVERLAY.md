# Apply Igel 3.8.6 Android ARM64 NEON overlay

1. Start with the original `Igel/igel-3.8.6/` source tree from `New-compiles.zip`.
2. Extract this ZIP into that repository root, preserving paths and allowing overwrite.
3. Commit the resulting files and run `.github/workflows/android.yml` manually (`workflow_dispatch`).

This overlay targets **Igel 3.8.6 only**. Do not apply it to 3.8.5 or another version.

Android compilation and device runtime still need confirmation from GitHub Actions and the phone.
