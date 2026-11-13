# generate-ninja 0.0~git20260722.641ace93 for LoongArch64

## Package

- Debian package: `generate-ninja` `0.0~git20260722.641ace93-3` (metadata-only packaging revision)
- Architecture: `loongarch64`
- Executable: `/usr/bin/gn`; the package provides the virtual package `gn`.
- Runtime dependency: `libc6 (>= 2.28)`.
- Candidate: `../binaries/generate-ninja/loongarch64/generate-ninja_v0.0~git20260722.641ace93-3_loongarch64.deb`
- SHA256: `8591aa2bcb07cfec714132b125656b03fcafb2a0131f7f8517f673adc6140383`
- Installed-Size: `12928` KiB

## Source and build

- Upstream: https://gn.googlesource.com/gn
- Exact commit: `641ace93dd9560e75e7add0d08f77b446fbb3b78`
- Git tree: `90329c7841d68716991cf2c247bcd9906c884c91`
- Verified source archive SHA256: `f1c0a45bf617a2a86b1e498b3837bc1a9855bad0fe70c48d664395bbe1bd1e08`
- Built in the existing Codex/V8 Loongnix 20 container (glibc 2.28), using the verified private GCC 13.4.0 and Python 3.10.16 copies.
- Compilation ran under the shared host lock with GN Ninja calls capped at `-j 2`. The only warning demoted from error was GCC's `-Wcomment`, using `CXXFLAGS=-Wno-error=comment`; the warning remains visible.
- No GN source file, RustyV8 source, or lock file was changed. GN's pristine export contains no `.git` metadata; `gn --version` therefore prints `0 (UNKNOWN)`. The package includes the full commit/tree/archive pin in `/usr/share/doc/generate-ninja/SOURCE-COMMIT`.

## ABI and validation

The binary is LoongArch ELF and requests `/lib64/ld.so.1`. Its highest referenced GLIBC symbol is `GLIBC_2.27`; dependencies resolve on the target container's glibc 2.28. It has no dynamic libstdc++ or libgcc dependency. The package requires libc6 2.28 to record the supported target baseline.

The packaged executable ran after extraction from the `.deb` inside the target container and reported `0 (UNKNOWN)`. GN generated a sample Ninja project, which Ninja then compiled and linked with two jobs; the resulting sample printed `Hello, world`. Logs and full build evidence are in the dedicated task cache at `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/`.

## Debian packaging reference

Debian uses the package name `generate-ninja`, installs `/usr/bin/gn`, and provides virtual `gn`: [package details](https://packages.debian.org/trixie/generate-ninja), [file list](https://packages.debian.org/trixie/ppc64el/generate-ninja/filelist). This package follows those names and paths. Its libc floor is based on the actual LoongArch binary and target baseline rather than Debian's unrelated trixie architecture builds.

The package contains the upstream README, license, and exact source pin. It does not include a generated man page.

## Metadata-only packaging revision -2 (2026-11-08)

- The previous `-1` candidate remains unchanged at `../binaries/generate-ninja/loongarch64/generate-ninja_v0.0~git20260722.641ace93_loongarch64.deb`; its SHA256 remains `8ec8415f19a08bbd0b946a3c921520186ad9735c1bac2a31ce62b1f8b4f377c5`.
- The `-2` candidate SHA256: `eda02a8e5a25b480d2704b032944b7acd7af7c1a76472d92facfdc2a3750cf97`; sidecar: `../binaries/generate-ninja/loongarch64/generate-ninja_v0.0~git20260722.641ace93-2_loongarch64.deb.sha256`.
- `Installed-Size` is `12924` KiB. `DEBIAN/md5sums` covers all five regular installed files; its SHA256 is `6e9dcfce2a7bc1d0cc2473683cc8c52a6a66b194ad8ea917e84d0800da040154`. The DEP5 `copyright` file SHA256 is `c55060fafd32110648c2cee6c4c6db1b27d6d3db3f73b04778f303e2429adc04`.
- The `/usr/bin/gn` DEP5 stanza now records both `BSD-3-clause` and `GPL-3+ with GCC Runtime Library Exception`; the complete upstream BSD text, GPLv3 text, and GCC Runtime Library Exception 3.1 text are included. The GCC attribution records the source headers `libgcc/libgcc2.c` (1989-2023, Free Software Foundation, Inc.) and `libstdc++-v3/src/c++98/ios_init.cc` (1997-2023, Free Software Foundation, Inc.); any other notices in linked runtime components remain applicable. The GCC license basis is documented in the [GCC 13.4.0 license page](https://gcc.gnu.org/onlinedocs/gcc-13.4.0/libstdc%2B%2B/manual/manual/license.html), with full text from `COPYING3` (SHA256 `8ceb4b9ee5adedde47b31e975c1d90c73ad27b6b165a1dcd80c7c545eb65b903`) and `COPYING.RUNTIME` (SHA256 `9d6b43ce4d8de0c878bf16b54d8e7a10d9bd42b75178153e3af6a815bdc90f74`). The local GCC 13.4.0 source archive used to inspect those notices has SHA256 `9c4ce6dbb040568fdc545588ac03c5cbc95a8dbf0c7aa490170843afb59ca8f5`; upstream source release: [GCC 13.4.0](https://gcc.gnu.org/pub/gcc/releases/gcc-13.4.0/). The format follows [Debian copyright-format 1.0](https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/).
- Archive inspection confirmed root ownership for all control and payload entries; directories are 0755, `/usr/bin/gn` is 0755, and documentation/control files are 0644. The packaged executable SHA256 is unchanged at `117d9e609fde83d5b248516399ee1cd00d8d3ab953ebe580095ddb633400cd12`. It remains ELF64 LoongArch `EXEC`, `e_flags=0x3 (LP64)`, interpreter `/lib64/ld.so.1`, highest GLIBC symbol `GLIBC_2.27`, and `DT_NEEDED` is `libm.so.6`, `libpthread.so.0`, `libc.so.6`, and `ld.so.1`; it has `DT_FLAGS=STATIC_TLS` and no `DT_FLAGS_1/2/3`.
- This revision only repackages metadata and documentation; GN/Ninja were not rebuilt. No package installation smoke was run here; the main Agent will arrange it separately.

## Final local candidate: metadata-only packaging revision -3 (2026-11-09)

- Exact Debian version: `0.0~git20260722.641ace93-3`; architecture: `loongarch64`; dependency: `libc6 (>= 2.28)`; `Installed-Size`: `12928` KiB.
- Candidate: `../binaries/generate-ninja/loongarch64/generate-ninja_v0.0~git20260722.641ace93-3_loongarch64.deb`; SHA256: `8591aa2bcb07cfec714132b125656b03fcafb2a0131f7f8517f673adc6140383`; checksum sidecar: `../binaries/generate-ninja/loongarch64/generate-ninja_v0.0~git20260722.641ace93-3_loongarch64.deb.sha256`.
- This is a candidate retained locally for review; it has not been published or installed. The existing `-1` and `-2` packages remain unchanged, and the Ninja `-2` candidate is unchanged.
- No GN rebuild occurred. `/usr/bin/gn` is byte-for-byte the reviewed `-2` executable, SHA256 `117d9e609fde83d5b248516399ee1cd00d8d3ab953ebe580095ddb633400cd12`.
- The additional embedded notice comes from the exact GN source file `src/base/third_party/icu/LICENSE` at commit `641ace93dd9560e75e7add0d08f77b446fbb3b78` (source tree `90329c7841d68716991cf2c247bcd9906c884c91`). Its verified source-export path is `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rusty-v8-native-20261107/source/gn/src/base/third_party/icu/LICENSE`, SHA256 `5d85142a5609ad177a2d7a2e7cae060b886b8b42f25c5b9803cf0cb5ee04ad2f`. GN's `build/gen.py:673-674` compiles `utf_string_conversion_utils.cc` and `utf_string_conversions.cc`, both of which include `base/third_party/icu/icu_utf.h`; the corresponding Unicode 1991-2017 and IBM 1995-2016 copyright and license terms from this complete `LICENSE` file are present in `DEBIAN/copyright` under the `usr/bin/gn` DEP5 stanza. Validation confirmed that the embedded notice matches the source file byte-for-byte after DEP5 indentation is removed.
- The `-3` package integrity record is `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rusty-v8-native-20261107/evidence/generate-ninja-package-v3-validation.json`; it confirms matching package staging, root ownership, modes, all five regular-file MD5 entries, `Installed-Size`, and the unchanged GN binary hash. The paired GN `-3` / Ninja `-2` candidate manifest is `codex-v0.166.0-native-tools-candidate-manifest-20261109.json` in this report directory.
- The main Agent's queued `-2` installation and sample run remain the runtime evidence. This `-3` candidate received package and documentation integrity checks only; no installation, rebuild, suite, or V8 build was run.
