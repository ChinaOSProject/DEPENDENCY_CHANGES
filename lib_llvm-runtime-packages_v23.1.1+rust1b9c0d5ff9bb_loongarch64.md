# LLVM runtime package candidates, 2026-11-11

Repository: https://github.com/llvm/llvm-project.git
Rust LLVM repository: https://github.com/rust-lang/llvm-project.git

Three runtime `.deb` candidates are complete and package-validated. They are not installed or published. Rust stage1/std owns the shared build lock; this packaging run did not acquire it or compile anything. Existing READY and consumer inputs are unchanged.

## Delivered candidates

| Package | Version | Bytes | SHA256 |
|---|---|---:|---|
| lld-21 | 21.1.8-0chinaos2 | 3048380 | `380f842770feec6bb13aa2bc283b3f032fa02c8ebfc0bfe311fffb1b88c18a94` |
| libllvm21 | 21.1.8-0chinaos2 | 60617642 | `e54242653a1ff5b100d8ccf7dc2c7c5633a20280eb19deefe48c428ab84cb6f0` |
| libllvm23 | 23.1.1+rust1b9c0d5ff9bb-0chinaos1 | 46218492 | `82ca9868f11a410fcbc063e952de520c7952b48d68aaae49462ec9834585e959` |

Target architecture is `loongarch64`, as returned by both host and existing container dpkg. Debian uses the architecture label `loong64`; this target distribution uses `loongarch64`. All files, temporary trees and package bytes are on ext4 UUID `5614614b-0d3f-4d00-b47b-d3ec9df1081f`. Conventional output paths are symlinks to the external disk; publish resolved `.deb` bytes.

- [lld-21](/home/laevatein/ChinaOSProject/binaries/lld-21/loongarch64/lld-21_v21.1.8-0chinaos2_loongarch64.deb)
  - Depends: `libc6 (>= 2.28), libgcc1 (>= 1:3.0), libllvm21 (= 21.1.8-0chinaos2), libstdc++6 (>= 8.3.0), zlib1g (>= 1:1.2.2)`
- [libllvm21](/home/laevatein/ChinaOSProject/binaries/libllvm21/loongarch64/libllvm21_v21.1.8-0chinaos2_loongarch64.deb)
  - Depends: `libc6 (>= 2.28), libgcc1 (>= 1:4.2), libstdc++6 (>= 8.3.0), zlib1g (>= 1:1.2.0)`
- [libllvm23](/home/laevatein/ChinaOSProject/binaries/libllvm23/loongarch64/libllvm23_v23.1.1+rust1b9c0d5ff9bb-0chinaos1_loongarch64.deb)
  - Depends: `clash-libgcc-s1 (= 13.4.0-1loong1), clash-libstdc++6 (= 13.4.0-1loong1), libc6 (>= 2.28), zlib1g (>= 1:1.2.0)`

Exact package/evidence manifest: [/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm-runtime-packages-20261111/HANDOFF-PACKAGES.json](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm-runtime-packages-20261111/HANDOFF-PACKAGES.json).

## Debian split and file ownership

References were retrieved on 2026-11-11 and cached with SHA256 in `evidence/debian-references.json`:

- [Debian lld-21 file list](https://packages.debian.org/sid/loong64/lld-21/filelist)
- [Debian libllvm21 file list](https://packages.debian.org/sid/loong64/libllvm21/filelist)
- [Debian libllvm23 file list](https://packages.debian.org/sid/loong64/libllvm23/filelist)
- [Debian llvm-23-dev file list](https://packages.debian.org/sid/loong64/llvm-23-dev/filelist)

`lld-21` owns `/usr/lib/llvm-21/bin/lld`, its `ld.lld`, `ld64.lld`, `lld-link` and `wasm-ld` aliases, and only version-suffixed `/usr/bin/*-21` links. This is the fixed ELF driver; other driver archives were reused from LLVM21. No global compiler/linker default is replaced. A generated man page is not present.

`libllvm21` owns `/usr/lib/loongarch64-linux-gnu/libLLVM.so.21.1` and `libLLVM-21.so`. `libllvm23` owns `libLLVM.so.23.1` and `libLLVM-23.so` in the same multiarch directory. This matches the Debian runtime ownership. Each runtime library package has a real shlibs record, ldconfig trigger, `Multi-Arch: same`, upstream license and nonempty payload. Unversioned `libLLVM.so` is reserved for a future development package. No libc++ files are included: this exact LLVM23 pin was built against GCC13 libstdc++, C++11 ABI=1.

`lld-21` declares `Breaks/Replaces: llvm-21 (<= 21.1.8-1)` because the historical project package was monolithic and owns the same linker files. That old package must be split before a co-installable upgrade; it is not a qualifying independent libllvm21/lld-21 split.

The old 20b snapshot candidates under `/cargo-home/llvm-packaging-23-20261111` are preserved. Their libLLVM SONAME is `libLLVM.so.23.0git`. The new libllvm23 package has `Breaks` on pre-23.1.1 clang-23/libclang-cpp23/libclang1-23/llvm-23 so an upgrade cannot silently satisfy their broad dependency while removing the required old SONAME. Those snapshots are not interchangeable; the old Clang/tool family needs its matching runtime or a coordinated later split. No old candidate was overwritten.

LLD21 source base is official2078da43e25a4623cab2d0d60decddf709aaea28 plus the reviewed baseline patch SHA25673ab90191ad3fbea05c626dc98073b41abb1e89f8aaf87b32d4bd72c4571b8bd and no-R incremental patch SHA25652cef35c105bd2a9bd0ef123b209e5a26f809003e6cf73e74d9a2fdf8d4785a5. The package linker is the READY binary with only its RUNPATH copy changed. Reused libLLVM21 bytes retain SHA256beef662f6de8ca6878e3a1395e414a67c9ed56faafcbdcf4b9faef891dd9f823.

## Relocation, dependency and package validation

All work ran in the existing `codex-v0.166.0-v8-directory-probe-20261107` container. Staging copies and verification streams were capped at 3 MiB/s; packaging ran at nice 19 with gzip level 1 and one compression thread. No large tree copy, host build, dependency installation, new container or lock acquisition occurred.

Only the existing RUNPATH strings in independent copies changed:

- LLD21: `$ORIGIN/../../loongarch64-linux-gnu`
- LLVM23: `$ORIGIN/../clash-verge/runtime/lib`
- LLVM21 library: unchanged `$ORIGIN/../lib`

`evidence/runpath-copy-changes.json` records exact byte intervals and before/after SHA256. All bytes outside those string intervals are unchanged. ELF headers, flags, segments, relocations, permissions and source binaries were not changed. Each packaged ELF remains flags0x3, GLIBC max2.27, no TEXTREL; the linker interpreter remains `/lib64/ld.so.1`. Runtime dependency search paths have no absolute build/cache path. Embedded compiler diagnostic/debug source strings are retained; no stripping or ELF header rewriting was performed.

`dpkg-shlibdeps` computed the actual dependencies. The base shlibs map gives no versioned libstdc++ minimum, so the two LLVM21 packages additionally require the verified base version `libstdc++6 (>= 8.3.0)`. `evidence/explicit-base-dependency-floor.json` proves this control-only refinement left the fully verified `data.tar.gz` bytes unchanged. LLVM23 depends on the existing exact `clash-libstdc++6` and `clash-libgcc-s1` version13.4.0-1loong1; no duplicate GCC runtime files are owned by these LLVM packages.

Without `LD_LIBRARY_PATH`, the extracted LLD resolves extracted libLLVM21 and base GCC8 runtimes; extracted LLVM23 resolves the extracted existing private GCC13 runtime packages through its relative RUNPATH. Four linker driver version queries and an `LD_BIND_NOW` dlopen / LLVMGetVersion query return23.1.1. These are package relocation checks; the earlier consumer/layout/native ABI gate suite was not rerun.

All archive payload bytes/SHA256, symlink targets, root uid/gid and unique file ownership were checked. Isolated dpkg simulation with the three new packages and two existing GCC runtime packages passed against a copy of the base status database. The20 non-directory candidate paths also have zero conflicts with20250 paths recorded in the available base dpkg file lists. The final isolated database includes the base file-list format metadata and completes without warnings; this remains a simulation, not a system installation. No system install or maintainer script execution occurred. Original source ELF hashes and mtimes remain unchanged (`evidence/final-input-readonly-check.json`).

## Development inventory and precise gaps

The exact Rust LLVM pin has2355 installed same-revision headers. The shared library relink used100 existing static archives, totaling390318896 bytes, plus its existing libllvm.cpp object. The build also contains an additional generated `libLLVMTableGen.a` (2730226 bytes):101 available archives in total,393049122 bytes. No archive or header was fabricated or regenerated.

Each of these2456 inputs has a proposed `llvm-23-dev` owner and installed path in [development-inventory.json](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm-runtime-packages-20261111/evidence/development-inventory.json). Existing verified SHA manifests supply all2355 header hashes and100 relink archive hashes; the additional TableGen archive hash is separately recorded. Proposed header ownership follows `/usr/include/llvm-23/llvm/...` and `/usr/include/llvm-c-23/llvm-c/...`; archives belong under `/usr/lib/llvm-23/lib`.

No `llvm-23-dev` package is created yet. The install prefix lacks a relocatable installed LLVMConfig.cmake and LLVMExports configuration. The build-tree exports retain absolute paths and declare73 executable targets,71 of which are unbuilt, including llvm-cov and llvm-profdata. Existing llvm-config, llvm-tblgen and llvm-min-tblgen are inventoried for later tools packaging. We did not start all, cov, profdata, test or any other LLVM build to fill these gaps. A full LLVM23 tools/SDK package and a matching newer Clang family remain separate delivery work.

Root20b tree remains clean; main/debian10/loong64 still point to20b6ec66967ac2a8f932863c1abf251e5b17a843. Exact23 source remains clean. No Rust edit, layout change, source commit, push or publication occurred. Overall publication remains with the main Agent after all products pass.

Source publication update, 2026-11-13: https://github.com/ChinaOSProject/llvm-project now has the exact unmodified20b input on `debian10`/`loong64` and the exact unmodified RustLLVM23.1.1 input `1b9c0d5ff9bbe7634aead059efe6b11a7eeba145` on `debian10-rustllvm23`/`loong64-rustllvm23`. The accepted LLVM21 changes are separately committed and published on `debian10-llvm21`/`loong64-llvm21`, both at `de0645087733b0320e759ba7886430ae2a95c3cd`; their final14 source files match the accepted patches. Fork default branches and historical validation limits are preserved. No new compiler/linker build, Rust edit, layout change, ABI-header rewrite or Magpie action occurred during this source publication.
