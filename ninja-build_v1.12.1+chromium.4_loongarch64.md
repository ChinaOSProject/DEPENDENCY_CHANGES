# ninja-build 1.12.1 Chromium revision 4 for LoongArch64

## Package

- Debian package: `ninja-build` `1.12.1+chromium.4-2` (metadata-only packaging revision)
- Architecture: `loongarch64`
- Executable: `/usr/bin/ninja`
- Runtime dependency: `libc6 (>= 2.28)`.
- Candidate: `../binaries/ninja-build/loongarch64/ninja-build_v1.12.1+chromium.4-2_loongarch64.deb`
- SHA256: `88c23ec7f5fe0a3329dcb660956c50601f8c827c5b30bc5b08883a9b13604592`
- Installed-Size: `17016` KiB

## Source and build

- Upstream: https://github.com/ninja-build/ninja; Chromium mirror: https://chromium.googlesource.com/external/github.com/ninja-build/ninja
- RustyV8 Chromium pin: `3@1.12.1.chromium.4`
- Exact source commit: `2daa09ba270b0a43e1929d29b073348aa985dfaa`
- Git tree: `a2903da7f05959eba7e9d809bf29befba23a6610`
- Verified source archive SHA256: `f2fbfc3339b572e4d14835c7ec2a7763ce8a4f3cc497b278a6f4c628a76d188d`
- Official Chromium 3pp recipe commit: `ddc3c4902f996e5a97dcc56805eeb8ff57ae97d5`; recipe SHA256: `5e9d2a74f406a7671ee9848137695e915fe22140b28d3fb3d5b22be1eef1fa68`. The recipe declares no source patches for this host build.
- Built in the existing Codex/V8 Loongnix 20 container (glibc 2.28) using the verified private GCC 13.4.0 copy. The existing GCC `as-loong64` wrapper is used from a checksum-matched private copy; `AR` uses the installed target `ar` because the private GCC copy lacks `liblto_plugin.so` at the location expected by `gcc-ar` and this build does not enable LTO.
- Ninja bootstrap's original self-rebuild has no jobs option. The pristine `source/ninja` export remains unchanged; the writable build copy has a two-line build-only change adding `-j 2` to that self-rebuild. The diff and original/work-copy hashes are preserved in task evidence. GN's generated Ninja calls also pass through the explicit `-j 2` wrapper.
- A first, unbounded self-rebuild completed under the shared host lock before the default parallelism was noticed. The final Ninja and GN outputs were rebuilt with the self-rebuild explicitly capped at two jobs; no other compile could use the shared lock concurrently.
- Bootstrap reported that re2c is unavailable. It used the `.in.cc` generated sources included in the verified archive; no source regeneration was needed.

## ABI and validation

The binary is LoongArch ELF and requests `/lib64/ld.so.1`. Its highest referenced GLIBC symbol is `GLIBC_2.27`; dependencies resolve on glibc 2.28. It has no dynamic libstdc++ or libgcc dependency. The package requires libc6 2.28 for the supported target baseline.

`ninja --version` reports `1.12.1`. The packaged executable ran after extraction from the `.deb` inside the target container. GN generated a sample Ninja project; Ninja compiled and linked its six-step target graph using two jobs, and the resulting sample printed `Hello, world`.

## Debian packaging reference

Debian packages this tool as `ninja-build` and installs `/usr/bin/ninja`: [package details](https://packages.debian.org/trixie/ninja-build), [file list](https://packages.debian.org/trixie/amd64/ninja-build/filelist). This candidate follows that name and executable path. It includes the upstream README, Apache license, and exact source/recipe provenance. It does not include Debian's generated man page or HTML manual.

The `.deb` and build logs are in the dedicated task cache at `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/`. No package was installed into the container. No V8 or RustyV8 build was started.

## Metadata-only packaging revision -2 (2026-11-08)

- The previous `-1` candidate remains unchanged at `../binaries/ninja-build/loongarch64/ninja-build_v1.12.1+chromium.4_loongarch64.deb`; its SHA256 remains `f2268d73bff20ceb70f8c3246ddb03c6bb78363e875fb25d2ffa4bebb402fa3e`.
- New candidate SHA256: `88c23ec7f5fe0a3329dcb660956c50601f8c827c5b30bc5b08883a9b13604592`; sidecar: `../binaries/ninja-build/loongarch64/ninja-build_v1.12.1+chromium.4-2_loongarch64.deb.sha256`.
- `Installed-Size` is `17016` KiB. `DEBIAN/md5sums` covers all five regular installed files; its SHA256 is `ed7aa7c59d542374a226aeea0d20adc808a8eafb89663c013ec7395497debd4a`. The DEP5 `copyright` file SHA256 is `c6bddfb41f07cc4a36067d7a592dfbe66ba5cfeb7fb697ef4c38f06ac5ab7a19`.
- The `/usr/bin/ninja` DEP5 stanza now records both `Apache-2.0` and `GPL-3+ with GCC Runtime Library Exception`; the complete upstream Apache text, GPLv3 text, and GCC Runtime Library Exception 3.1 text are included. The GCC attribution records the source headers `libgcc/libgcc2.c` (1989-2023, Free Software Foundation, Inc.) and `libstdc++-v3/src/c++98/ios_init.cc` (1997-2023, Free Software Foundation, Inc.); any other notices in linked runtime components remain applicable. The GCC license basis is documented in the [GCC 13.4.0 license page](https://gcc.gnu.org/onlinedocs/gcc-13.4.0/libstdc%2B%2B/manual/manual/license.html), with full text from `COPYING3` (SHA256 `8ceb4b9ee5adedde47b31e975c1d90c73ad27b6b165a1dcd80c7c545eb65b903`) and `COPYING.RUNTIME` (SHA256 `9d6b43ce4d8de0c878bf16b54d8e7a10d9bd42b75178153e3af6a815bdc90f74`). The local GCC 13.4.0 source archive used to inspect those notices has SHA256 `9c4ce6dbb040568fdc545588ac03c5cbc95a8dbf0c7aa490170843afb59ca8f5`; upstream source release: [GCC 13.4.0](https://gcc.gnu.org/pub/gcc/releases/gcc-13.4.0/). The format follows [Debian copyright-format 1.0](https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/).
- Archive inspection confirmed root ownership for all control and payload entries; directories are 0755, `/usr/bin/ninja` is 0755, and documentation/control files are 0644. The packaged executable SHA256 is unchanged at `2ca7f92185a490b942fa1ee1feb7abc51744d59c88f2e7eb375583a7b7e30c07`. It remains ELF64 LoongArch `EXEC`, `e_flags=0x3 (LP64)`, interpreter `/lib64/ld.so.1`, highest GLIBC symbol `GLIBC_2.27`, and `DT_NEEDED` is `libm.so.6`, `libc.so.6`, and `ld.so.1`; it has no `DT_FLAGS_1/2/3`.
- This revision only repackages metadata and documentation; GN/Ninja were not rebuilt. No package installation smoke was run here; the main Agent will arrange it separately.
