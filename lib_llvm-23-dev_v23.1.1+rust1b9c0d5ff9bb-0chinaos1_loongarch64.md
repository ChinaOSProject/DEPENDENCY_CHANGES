# LLVM23.1.1 LoongArch development package

Repository: https://github.com/rust-lang/llvm-project.git
LLVM upstream repository: https://github.com/llvm/llvm-project.git

Version:23.1.1, exact revision1b9c0d5ff9bbe7634aead059efe6b11a7eeba145. Package version `23.1.1+rust1b9c0d5ff9bb-0chinaos1`, architecture `loongarch64` as reported by both host and existing container dpkg.

Shared and static CMake consumer compilation/runtime passed.

## Candidate artifact

- [llvm-23-dev_v23.1.1+rust1b9c0d5ff9bb-0chinaos1_loongarch64.deb](/home/laevatein/ChinaOSProject/binaries/llvm-23-dev/loongarch64/llvm-23-dev_v23.1.1+rust1b9c0d5ff9bb-0chinaos1_loongarch64.deb)
- SHA256: `9ea6c59f4194c3e142e7a4e7d24390d13258f56245425b13108ff486ae3b4905`
- Bytes: 107018822
- Exact handoff: [HANDOFF-DEV.json](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm23-dev-packaging-20261111/HANDOFF-DEV.json)

Large files reside on ext4 UUID5614614b-0d3f-4d00-b47b-d3ec9df1081f. The conventional binaries path is a symlink to external package bytes; publish its resolved payload. No package installation, commit, push or publication occurred.

## Contents and ownership

The package contains2355 same-pin installed headers,100 real component static archives plus the existing libLLVMTableGen.a (101 archives,393049122 bytes), llvm-config, llvm-tblgen and llvm-min-tblgen. No header/archive/tool was rebuilt. All copied headers and100 component archives match the existing verified SHA manifests; the TableGen archive has a separately recorded SHA.

[Debian llvm-23-dev file ownership](https://packages.debian.org/sid/loong64/llvm-23-dev/filelist) is the baseline: headers live under `/usr/include/llvm-23/llvm` and `/usr/include/llvm-c-23/llvm-c`, with prefix include symlinks; archives, unversioned libLLVM.so and runtime symlinks live under `/usr/lib/llvm-23/lib`; CMake modules/exports live under `/usr/lib/llvm-23/lib/cmake/llvm`. Runtime SONAME files remain owned solely by the separate exact-version libllvm23 package. No duplicate library bytes are assigned to this development package.

This scoped SDK places its three existing configuration/TableGen executables under `/usr/lib/llvm-23/bin`; `/usr/bin/llvm-config-23` and `llvm-tblgen-23` are provided. Debian normally places llvm-config and general tools in llvm-23; here those useful existing tools are included with the SDK to provide a usable development package without requiring71 unbuilt general tools. `Breaks/Replaces: llvm-23 (<<23.1.1)` handles the former snapshot tool ownership. There is no empty llvm-23 or tools meta-package.

Only the LoongArch code-generation backend is built. The package excludes clang, llc, opt, llvm-cov, llvm-profdata and other unrelated general tools. This is a development SDK with the declared backend/tool scope, not a claim of the full Debian LLVM command-line suite.

## Original CMake metadata route

An independent metadata build reuses the original source revision, successful build CMakeCache capability results, and genuine C/C++/ASM compiler identification records. No compiler result is invented or forced. A compiler-launcher guard fails if a new compile is requested; configure exited0 with no guard invocation and no object produced. Compiler, old assembler, C++17, C++11 ABI=1, Release/assertionsOFF/EH-OFF/RTTI-OFF, shared LLVM-all and the LoongArch target match the real library build.

Configuration uses upstream `LLVM_DISTRIBUTION_COMPONENTS` listing the101 real static targets, LLVM, llvm-config, llvm-tblgen, llvm-headers and cmake-exports, with `LLVM_STRICT_DISTRIBUTIONS=ON`. `LLVM_INCLUDE_UTILS` and runtimes are OFF; the tools subtree is retained for LLVM shared/config targets, while unrelated `LLVM_TOOL_*_BUILD` options are OFF. `LLVM_INCLUDE_TOOLS=OFF` alone would exclude the subtree that defines the shared LLVM target, so the upstream whitelist is used.

Original CMake configure/generate and `cmake --install --component cmake-exports` produce unedited install-tree LLVMConfig.cmake, LLVMExports.cmake, LLVMExports-release.cmake and API modules. No imported target or path check is deleted. Installed exports contain exactly104 targets:101 static libraries, LLVM shared library and2 executable targets. All imported files exist. Three generated ABI configuration headers (llvm-config.h, abi-breaking.h, config.h) are byte-identical to the successful build.

The language-free consumer calls original `find_package(LLVM 23.1.1 EXACT CONFIG REQUIRED)` with default export/existence checks enabled. It imports104 LLVM targets plus ZLIB, confirms the LoongArch/RTTI/EH/assertion/shared configuration, and maps Core/Support/MC/LoongArchInfo successfully. Installed package metadata contains no build/cache absolute path. There is no LLVM_OMIT_EXPORTS_FROM_CONFIG override.

Exact commands and hashes: [configure-command.json](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm23-dev-packaging-20261111/configure-command.json), [generated ABI match](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm23-dev-packaging-20261111/evidence/generated-abi-config-match.json), [CMake import log](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm23-dev-packaging-20261111/evidence/cmake-import-check.log).

## Dependencies, relocatability and validation

Depends: `clash-libgcc-s1 (= 13.4.0-1loong1), clash-libstdc++6 (= 13.4.0-1loong1), libc6 (>= 2.28), zlib1g (>= 1:1.1.4), libllvm23 (= 23.1.1+rust1b9c0d5ff9bb-0chinaos1), libc6-dev, zlib1g-dev`. CMake is suggested at>=3.20 for CMake SDK consumers; direct llvm-config consumers do not require it. Existing private CMake3.28.6 is used for validation. Compiler/assembler/linker prerequisites remain external toolchains; they are not runtime libraries added mechanically to Depends.

The three tool copies use `$ORIGIN/../lib:$ORIGIN/../../clash-verge/runtime/lib` RUNPATH and the existing exact GCC13 runtime packages. Only those existing RUNPATH string byte ranges changed in independent package copies; the original executables and READY input are untouched. Headers, archives, ELF flags, relocations and segment permissions are not rewritten. The tool metadata audit retains GLIBC<=2.28, actual /lib64/ld.so.1, flags0x3 and no TEXTREL.

llvm-config23.1.1 resolves actual package include/lib/CMake directories; its --link-shared and --link-static libfiles all exist under the independent extracted union. All101 archives pass ar member inspection. TableGen tools start without LD_LIBRARY_PATH. All2508 payload files/links are checked against source SHA256 or recorded symlink targets; root uid/gid, non-overlap with runtime ownership, base file-list conflicts and isolated dpkg group dependency simulation pass. Compression is single-thread gzip1 at nice19; staged copies and archive verification streams are capped at3MiB/s.

## Consumer validation

The two small CMake consumers use the same GCC13+old-as/fixed-LLD driver, C++17, Module/Attributes and LoongArch TargetInfo APIs. One links imported LLVM, the other Core/Support/MC/LoongArchInfo static targets. Tests retain normal and LD_BIND_NOW execution, interpreter, GLIBC floor, actual shared-library resolution, and static-vs-shared NEEDED checks. No full LLVM or extra tool build is scheduled.

State: `PASSED`. [consumer-state.json](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm23-dev-packaging-20261111/consumer-state.json) and [lock runner log](/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm23-dev-packaging-20261111/consumer-lock-runner.log) record actual results. The runner queued at 11:55:14 UTC, acquired the shared flock at 11:58:03, and released it at 11:58:06 on 2026-11-11. Compilation used 2 jobs and 1 link job. Both shared and static consumers returned 0 with and without LD_BIND_NOW, reporting LLVM 23.1.1. Their interpreter is /lib64/ld.so.1, highest GLIBC requirement is 2.27, and neither contains TEXTREL. Shared linking resolved the independent extracted package library; the static executable has no libLLVM NEEDED entry.

The earlier runtime-only report accurately recorded the missing install exports at that time. This independent upstream metadata configuration resolves that development packaging gap while retaining all previously delivered runtime packages and source/READY inputs. Overall product publication remains with the main Agent.
