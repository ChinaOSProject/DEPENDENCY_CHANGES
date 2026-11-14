# float16 0.1.5 LoongArch stable-compiler adaptation

Date: 2026-11-12 (Asia/Shanghai). Repository: `https://github.com/Alexhuszagh/float16`.
Upstream revision: `2401a40bbdeac256070324ec954bf268bc8e7be7`; upstream tree: `6bd4312db2a9163be849911b685c314b535f09f5`.
Checkout: `/home/laevatein/ChinaOSProject/float16`, branch `debian10`.
On 2026-11-13, the reviewed two-file adaptation was committed as `9028edc088ab4419536a5aa0b30cc227fcb5eb03` on `debian10`; `loong64` was fast-forwarded to the same commit. On 2026-11-14, GitHub repository metadata confirmed the user's new `ChinaOSProject/rust-float16` fork has parent and source `Alexhuszagh/float16`. Both adaptation branches were then atomically pushed and independently verified at that exact commit. The fork's default `main` remained at `6066be1922fb528a23aacb4da5880160378c758c`, and local `main` retains the exact build baseline. The working tree is clean; no source, patch or package changed and no tests were rerun for publication. The unrelated Go fork `ChinaOSProject/float16` remains untouched.

## Problem and production change

On the verified native Rust `1.99.1-dev` snapshot (LLVM `23.1.1`), original float16 0.1.5 fails with E0658 for the unstable `stdarch_loongarch` types and conversion intrinsics. Its LoongArch module and dispatch previously compiled solely from the target architecture/CPU feature.

The production diff changes only `build.rs` and `src/binary16/arch.rs`:

- On LoongArch, a metadata-only target compiler probe compiles the complete existing LSX conversion module. For the `std` feature, it also checks `std::arch::is_loongarch_feature_detected!("lsx")`.
- The probe uses Cargo's `RUSTC`, `TARGET`, `CARGO_MANIFEST_DIR`, `OUT_DIR`, and `CARGO_ENCODED_RUSTFLAGS`. It passes `--crate-type lib --edition 2021 --emit metadata --target ...`; it neither links nor executes a target program.
- Only a successful probe emits `has_loongarch_intrinsics`. The module, explicit `target_feature="lsx"` dispatch, and runtime-detection dispatch all require that cfg.
- Missing probe inputs, source-write failure, compiler launch failure, or compiler rejection conservatively leave the cfg unset. The library then uses its existing pure Rust conversions. Expected probe diagnostics are captured rather than presented as a failed library build.
- A compiler that accepts those APIs retains the unchanged hardware module and existing CPU guards. No compiler feature flag, `RUSTC_BOOTSTRAP`, dependency version change, arithmetic change, rounding change, or new dependency was introduced.

The complete existing module is probed to avoid a false positive from checking only one intrinsic. x86/aarch64 paths remain unchanged. Current compiler rejection is a capability result even though its version number is newer than many stabilization guesses.

## MSRV and cross-compilation review

The package retains `rust-version = "1.60"`. The new build script uses standard library APIs available by Rust 1.60; `CARGO_ENCODED_RUSTFLAGS` is supplied by Cargo since 1.55. The probe source uses edition 2021, already required by this crate. Existing `rustc_version = "0.2"` remains unchanged.

The probe selects the target compiler via `RUSTC` and explicitly supplies Cargo's target and encoded target flags, including any sysroot/CPU options. It requires no target executable, linker, or target runtime. Outputs are limited to Cargo's `OUT_DIR`. Unsupported/custom target resolution conservatively falls back rather than enabling unsupported code.

An actual Rust 1.60 toolchain and a second cross-compilation target were not present in the authorized environment. The MSRV assessment is therefore an API review. Explicit `--target loongarch64-unknown-linux-gnu` was tested on the native LoongArch host. Execution of the successful hardware-probe path requires a compiler that accepts these APIs; the available compiler rejects them, so that future hardware path was preserved by inspection rather than executed.

## Isolation and source provenance

All compilation, tests, and package builds use the existing container `codex-v0.166.0-v8-directory-probe-20261107` through `/tmp/codex-v0.166.0-20261105/bin/podman-codex exec` and the root's verified `app-native-env-main-20261111.sh`. Every build/test holds `/tmp/chinaos-heavy-build-20261105.lock` on the host; Cargo jobs are limited to two. Dedicated task target directories are used.

Task root: `TASK/float16-adaptation-20261112`, where `TASK=/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110`.

The crate checkout's `Cargo.lock` is unchanged (Git blob `923dc8cd9d1c59208daa2b9747e4b3621630bdf4`). Test-copy locks select the exact cached application dependencies (cfg-if 1.0.4, rustc_version 0.2.3, semver 0.9.0, semver-parser 0.7.0); canonical application source/cache/lock files are not edited. The root recovered the exact upstream source; evidence is in `source-inputs/source-recovery.json`. Temporary tests and packaging scripts remain outside the dependency repository.

No host build dependency was installed, no new container was created, no security policy was changed, and no Magpie executable, desktop entry, process, or signal operation was invoked.

## Validation

| Check | Result | Evidence relative to task adaptation directory |
| --- | --- | --- |
| Original upstream native check | Expected failure, E0658, 16 unstable API errors | `logs/baseline-check.log` |
| Patched upstream unit tests plus direct vector test | 43 passed (42 upstream, 1 additional) | `logs/patched-compiled-tests.log` |
| Independent public conversion tests with std | 3 passed | `logs/patched-compiled-tests.log` |
| Independent public conversion tests with no default features | 3 passed | `logs/patched-compiled-tests.log` |
| Explicit LoongArch target, `+lsx`, no default features | 3 passed | `logs/patched-explicit-lsx.log` |
| Doctests with provided environment | Tooling limitation: E0514 | `logs/patched-tests.log` |
| Extracted-package offline std/no_std consumer | 3 + 3 passed, fresh Cargo home | `logs/package-consumer.log` |
| Isolated dpkg installation of all five packages and installed-source offline consumer | All five configured; 3 + 3 passed | `logs/package-install-consumer.log`, `package-content-audit.json` |

The independent test suite checks all 65,536 binary16 encodings against an exponent/fraction numeric reference, compares widened floating-point bits (including signed zeros), round-trips every finite/infinite encoding, checks NaN classifications, and checks known normal/subnormal/overflow/ties-to-even cases. Slice lengths 0 through 17 cover empty, exact-lane, and remainder cases with finite, zero, infinity, and NaN values. An external temporary unit module directly tests 4- and 8-lane conversions in both f32/f64 directions. Existing bf16 and slice unit tests pass as part of the upstream suite.

Doctests fail because the verified compiler snapshot contains only `bin/rustc`, while Cargo finds system rustdoc built with Rust 1.99.0. That rustdoc rejects 1.99.1-dev library artifacts. Compiled unit/integration tests pass when invoked explicitly; no alternate compiler was downloaded, installed, or substituted to hide this gap.

The emitted build-script outputs for std/no_std contain only the existing x86/aarch64 cfgs and omit `has_loongarch_intrinsics`, confirming that this compiler uses software conversion even with explicit `+lsx`.

## Debian/Ubuntu package correspondence and delivery

This is the actual float16 crate, not the separate `half` crate. The conventional source package is `rust-float16`; its binary source-development package is `librust-float16-dev`.

Official raw evidence is preserved under `package-audit/`, with URLs, response headers, status, byte counts, and SHA-256 values in `fetch-*.json`:

- Debian's current pool lists rust-float16 0.1.7-1, including a loong64 source-development binary: `https://deb.debian.org/debian/pool/main/r/rust-float16/`.
- The exact 0.1.5-2 Debian-origin source package remains in the official Ubuntu archive: `https://archive.ubuntu.com/ubuntu/pool/universe/r/rust-float16/rust-float16_0.1.5-2.dsc` and corresponding `.debian.tar.xz`.
- The exact signed `.dsc` names `librust-float16-dev`, architecture `any`, and upstream minimum Rust 1.60. Its declared SHA-256 for the Debian packaging archive matches the downloaded raw archive (`082b9802e6e5f454a261141b06f75ba2f7a989a49cf6e73fb0d560b9fd4ab529`). The signed text is preserved; no claim is made that its OpenPGP signature was independently verified.
- That packaging already carries a conservative `has_loongarch_intrinsics` gate that is never set. Our additional compiler probe permits future supported compilers to retain LSX.
- It also changes rustc_version 0.2 to 0.4. That unrelated dependency change is not imported: this adaptation retains the locked 0.2.3 and records its actual source-development closure.
- Historical Debian source metadata for rustc_version 0.2.3, semver 0.9.0, semver-parser, and cfg-if is preserved from `https://archive.debian.org/debian/pool/main/r/` in the audit directory. It confirms the conventional source-dev package names and default-feature dependency names. In particular rustc_version 0.2.3 requires semver 0.9; semver 0.9 metadata requires semver-parser 0.7. These are distinct version families, not replacements with newer incompatible crates.

The required default-feature source-development package set is:

| Binary source-dev package | Version | Required real-package dependencies |
| --- | --- | --- |
| librust-float16-dev | 0.1.5-0chinaos1 | librust-cfg-if-dev (= 1.0.4-0chinaos1), librust-rustc-version-dev (= 0.2.3-0chinaos1) |
| librust-cfg-if-dev | 1.0.4-0chinaos1 | none |
| librust-rustc-version-dev | 0.2.3-0chinaos1 | librust-semver-dev (= 0.9.0-0chinaos1) |
| librust-semver-dev | 0.9.0-0chinaos1 | librust-semver-parser-dev (= 0.7.0-0chinaos1) |
| librust-semver-parser-dev | 0.7.0-0chinaos1 | none |

Each package places its crate under `/usr/share/cargo/registry/NAME-VERSION`, includes upstream license texts and provenance, and provides Debian-style version/default-feature aliases. Float16 additionally provides its std aliases. Optional cfg-if core and semver serde/ci package aliases are not advertised because those closures are not delivered. Float16 keeps Debian's `Breaks: rustc (<< 1.60~)`.

The packages use the target system's actual architecture spelling `loongarch64` (`dpkg --print-architecture`), differing from Debian archive spelling `loong64`. They follow the conventional source-development package shape (`Multi-Arch: same`) and contain no ELF file, SONAME, rlib, or CLI; no native ABI/runtime dependency is added. Rust build dependencies belong to these source-dev packages and are not GUI runtime dependencies. The four dependency sources are unmodified and validated byte-for-byte against cached crate archives whose SHA-256 values equal the application lockfile. No dependency download or source adaptation is needed for this closure.

All five packages were built in the authorized existing container and configured together in the isolated task-local dpkg root `package-install-root/`. A consumer with a fresh Cargo home and dedicated target directory then generated its lock and passed all three independent conversion tests for both std and no_std using only the installed `/usr/share/cargo/registry` sources, offline. The extracted-package consumer passed the same matrix independently. No system dpkg database or host package installation was used. `package-content-audit.json` confirms all five installation states and the absence of ELF/compiled-library/executable content. Native ABI floors are therefore not applicable to these source-only archives.

The package definitions, source checksums, and reproducible scripts are in `package-staging/`, `package-manifest.json`, `float16-stage-packages.py`, `run-package-build.sh`, `run-package-consumer.sh`, and `run-package-install-check.sh`. An initial packaging retry exposed a self-entry in the generated `.cargo-checksum.json`; the packaging script now excludes that metadata file from its own hash map. Its failed log is preserved as `logs/package-consumer-initial-checksum-self-entry.log`. A first attempt to use an empty isolated dpkg database required creating dpkg's standard subdirectories; that setup failure is preserved in `logs/package-install-initial-empty-database.log`. These fixes concern only external packaging helpers; the accepted production source remained frozen.

No packaging exemption is claimed because an independent source-development package correspondence exists. All delivered files are listed in `artifact-manifest.json`; checksums are also in `output/SHA256SUMS`.

| Delivered artifact | Bytes | SHA-256 |
| --- | ---: | --- |
| `/home/laevatein/ChinaOSProject/binaries/librust-float16-dev/loongarch64/librust-float16-dev_v0.1.5-0chinaos1_loongarch64.deb` | 46796 | `b3251840b3f2a15e45436c5538eface52dda2e9bbefa9b9b49656e792a921602` |
| `/home/laevatein/ChinaOSProject/binaries/librust-cfg-if-dev/loongarch64/librust-cfg-if-dev_v1.0.4-0chinaos1_loongarch64.deb` | 10872 | `45b3f17883b409f1c4e0718892cf4ef5f7d851680f465565a04cfc727e754dca` |
| `/home/laevatein/ChinaOSProject/binaries/librust-rustc-version-dev/loongarch64/librust-rustc-version-dev_v0.2.3-0chinaos1_loongarch64.deb` | 11388 | `488067d10d79e6b94195b83aacf91e4c1db857ca0350a35e6a9fe6884764e0cf` |
| `/home/laevatein/ChinaOSProject/binaries/librust-semver-dev/loongarch64/librust-semver-dev_v0.9.0-0chinaos1_loongarch64.deb` | 17932 | `eff26144a2c43e47ff361293fa5c4fda770ce56a919f5477fa2dfdf86be188a5` |
| `/home/laevatein/ChinaOSProject/binaries/librust-semver-parser-dev/loongarch64/librust-semver-parser-dev_v0.7.0-0chinaos1_loongarch64.deb` | 11456 | `e5e098a374f0fa541fdf3a229f10bb94ede6b507bbd410802918d5abe30c5b6e` |


## Root integration and handoff

The root accepted the compiled tests and copied the exact frozen production diff into its own `app-path-override/float16-0.1.5`, retaining the original registry-normalized manifest. It owns application integration through Cargo's documented paths override and unchanged application lockfile; this agent did not modify the application source or registry cache. Production source files are frozen after that copy.

For review and independent copying, `source-production/` is the exact Git baseline plus only the two-file production diff, with the original source lock intact. `production.patch` and `production-manifest.json` identify the source/patch hashes. The separately generated directory-source checksum metadata under the staged/extracted package crate describes the adapted file bytes explicitly while retaining the upstream crate archive checksum for Cargo provenance.

The final GUI integration and installed runtime acceptance are recorded in `clash-verge_v2.5.10-1loong2_loongarch64.md`. The root completed the local branch propagation and source commit described above and owns remote publication. No successful test or consumer was rerun for the source commit.
