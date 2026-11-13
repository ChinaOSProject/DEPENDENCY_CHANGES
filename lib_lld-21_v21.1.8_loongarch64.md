# LLD21 no-R CIE compatibility fix — verified candidate

Repository: https://github.com/llvm/llvm-project.git

LLVM21 no-R CIE fix and LLVM23.1.1 consumer: READY for wE std/stage2 input.

LLD: /cargo-home/llvm21-cie-20261111/candidate/bin/ld.lld
LLD SHA256: 6b68ffabfe2fb27898b6cfed989d6be89730f1b700482101bbbb97e1a2e1d5e0
LLVM_CONFIG: /cargo-home/llvm21-cie-20261111/llvm23-candidate/install/bin/llvm-config
CC: /cargo-home/llvm21-cie-20261111/bin/gcc13-cc
CXX: /cargo-home/llvm21-cie-20261111/bin/gcc13-cxx
LD_LIBRARY_PATH: /cargo-home/llvm21-cie-20261111/llvm23-candidate/install/lib:/cargo-home/rusty-v8-native-20261107/toolchains/gcc-13/lib:/cargo-home/rusty-v8-native-20261107/toolchains/llvm-21/lib

Use GCC13 libstdc++ (C++11 ABI=1), not a libc++ consumer configuration. Same compiler/old-as runtime fixtures were retained; only task-local linker selection changed. Loader /lib64/ld.so.1; GLIBC max 2.27. libLLVM23 and mixed binaries flags0x3; LLVM23-produced objects unchanged0x43. A pure-modern0x43 DSO is byte-identical to original LLD21 output and returns42 on real glibc2.28.

35/35 LoongArch tests; empty/z/zPL/zR/zPLR/zS record and relocation checks; duplicate CIE/eh_frame_hdr; real C++ throw/catch through old-as/modern frames with LSDA/personality/destructor; five real Rust wrappers; AssignGUID/version/layout; call/FP/pthread/LE/IE/GD TLS/dlopen/default+LD_BIND_NOW passed in container and host. Zero dynamic relocation writes into nonwritable LOADs and no TEXTREL. Runtime/layout expectations were preserved. Rust source/layout checks untouched.

Incremental patch: /mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm21-cie-20261111/logs/no-r-cie-fix-20261111/no-r-incremental.patch
SHA256: 52cef35c105bd2a9bd0ef123b209e5a26f809003e6cf73e74d9a2fdf8d4785a5
Complete official21 + baseline + fix: /mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm21-cie-20261111/logs/no-r-cie-fix-20261111/complete-candidate.patch
Exact records/logs/manifests: /mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm21-cie-20261111/HANDOFF-READY.json

Only standalone LLD ELF backend/common + standard driver built; LLVM21 libraries and other driver archives reused read-only. Existing LLVM23 objects only relinked to the new task prefix. No complete LLVM rebuild. Original20b refs/tree and23 source remain clean; original prefixes untouched. No commit/push/package. Shared lock released.

This is shared LLVM development input with2355 same-pin headers, not full -dev: static LLVM libraries, CMake exports, cov/profdata are absent. Keep original headers and private LLVM21/GCC13 runtimes. Rust std/stage2 has not been built by this task.

The previous blocker and reproducer evidence is retained below as history.

# LLD21 old-as CIE compatibility blocker

Repository: https://github.com/llvm/llvm-project.git
Version: 21.1.8 (`llvmorg-21.1.8`). This is a triage report; no new source patch, package or READY has been produced.

The existing private patched LLD21 misses CIE version3 with empty augmentation (implicit absolute FDE encoding). Its conversion checks nonzero fdeEncodingOffset, so the old-as no-R form is skipped. The original writable input .eh_frame becomes read-only in the synthetic output while RELATIVE dynamic writes remain and no TEXTREL is emitted. Old glibc2.28 correctly faults on those writes. This is not established by flags alone: all1890 LLVM library host objects are flags0x3, and the failed mixed PIE output is also0x3.

Existing source: `/home/laevatein/src/zig-0.16.0-container/llvm-project-21.1.8.src` (release archive, no Git metadata). Original archive: `/home/laevatein/src/zig-0.16.0-container/llvm-project-21.1.8.src.tar.xz`, SHA256 `4633a23617fa31a3ea51242586ea7fb1da7140e426bd62fc164261fe036aa142`.

Full existing patch: `/home/laevatein/ChinaOSProject/DEPENDENCY_CHANGES/llvm-21_v21.1.8_loongarch64.patch`, SHA256 `73ab90191ad3fbea05c626dc98073b41abb1e89f8aaf87b32d4bd72c4571b8bd`. Its read-only reverse dry-run with fuzz0 passes all hunks. Exact source-file hashes and a preserved copy of the full patch are in `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm-rust-pin-1b9c0d5ff9bb-20261111/logs/bfd-relink-20261111T092759Z`.

Minimal reproducer: `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm-rust-pin-1b9c0d5ff9bb-20261111/logs/bfd-rustlld-ehframe-20261111/plain.c` compiled by GCC13/task old-as wrapper into flags0x3 plain.o. Existing LLD21 shared link exits0; `diagnose-dlopen` exits139 in the real glibc2.28 container. BFD links and loads the same object, rewriting it to zR/PC-relative EH records. Existing rust-lld22 also loads the old-object DSO, but rejects real modern LLVM0x43 object e_flags67. These comparisons do not permit passing a runtime gate merely on linker exit code or0x3.

BFD-only relink of the actual LLVM23 library into a new candidate prefix succeeds and loads in both container and host. It removes all82524 read-only dynamic writes. The unchanged mixed PIE driver.o(old as)+LLVM le.o(modern) still gets3 such writes via private LLD21 and faults before main on both systems. Changing the driver compiler would hide the required compatibility case and is not used.

Next step: main Agent prepares a separate official LLVM21 worktree/branch with the complete existing patch, independently from the20b root and23 pin. Address the missing CIE form with correct EH metadata/relocation sizing and real loader/RELRO permissions. Retain modern and mixed-object behavior gates. No ELF post-processing, fake TEXTREL, header flag edit, mprotect bypass or Rust layout change is allowed. Reuse existing LLVM23 objects/library; no1874-step rebuild is needed.

Detailed machine-readable handoff and exact commands: `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/llvm-rust-pin-1b9c0d5ff9bb-20261111/logs/bfd-relink-20261111T092759Z/HANDOFF-NOT-READY.json`; `minimal-lld21-reproducer.json`; `relocation-permission-audit.json`; `modern-le-rustlld.json`.


## Source patch archive, 2026-11-13

The accepted source bytes are archived alongside this report as [the incremental no-R CIE fix](llvm-21_v21.1.8-no-r-cie.patch) and [the complete adaptation against official LLVM 21.1.8](llvm-21_v21.1.8-complete-oldworld.patch). Their SHA256 values are `52cef35c105bd2a9bd0ef123b209e5a26f809003e6cf73e74d9a2fdf8d4785a5` and `1f10982e79f454adeaf21b007b1a6c3c52efed0fa91c2c6c6269e316f33e4ef6`. These are copies of the already tested source changes; no source, compiler or linker was rebuilt. The exact runtime/linker and scoped LLVM23 development packages are described in the adjacent package reports. Source-repository publication remains separate from report and binary archival.
