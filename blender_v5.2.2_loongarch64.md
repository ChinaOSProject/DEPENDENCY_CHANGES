Dependency: GCC 14.3 C++ runtime (`libstdc++.so.6`, `libgcc_s.so.1`)
Version: 14.3.0
Type: Bundled runtime libraries
Reason: Blender was built with GCC 14.3. The target system's default APT `libstdc++6` is GCC 8.3 and does not provide the `GLIBCXX_3.4.32` symbol required by the executable.
Root Cause: The executable and bundled native dependencies use the GCC 14 C++ ABI; the target's system C++ runtime has an older symbol set.
Changes: The Debian package includes GCC 14.3 `libstdc++.so.6` and `libgcc_s.so.1` under `usr/lib/blender/lib`. The executable uses the relative RUNPATH `$ORIGIN/lib`.
Compatibility Impact: The package uses its private C++ runtime and leaves the target's `glibc` and system library directories unchanged. The executable requires no GLIBC version newer than `GLIBC_2.28`. X11, xkbcommon, and PulseAudio runtime libraries are declared as APT dependencies.
Upstream Status: No GCC source changes or upstream patch. Runtime bundling is part of this package build.
Validation: `readelf --version-info` reports `GLIBC_2.28` and `GLIBCXX_3.4.32` as the highest executable requirements. `ldd` resolves all executable dependencies on Kylin V10 SP1. The extracted `.deb` launches without a custom `LD_LIBRARY_PATH`; a Blender 5.2.2 LTS smoke test subdivided a cube and rendered a PNG with Cycles CPU.
Project-side Changes: LoongArch-specific dependency build settings, unsupported optional backend selection, Wayland `lib64` layout, and `makesdna` link configuration are recorded in the `loong64` branch commits.
