# Protobuf 3.21.12 — LoongArch ABI 1.0 / glibc 2.28

## Scope and source

Upstream repository: https://github.com/protocolbuffers/protobuf . Built the unmodified C++ runtime/compiler source to provide the proto3 optional syntax required by Codex CLI v0.166.0. The Buster repository protoc 3.6.1 rejected the actual `code-mode-protocol/src/grpc/*.proto` optional fields before compilation; the new installed protoc accepts that same input. No Codex or dependency source was changed for this tool.

The upstream orig archive was obtained from the [official Debian protobuf pool](https://deb.debian.org/debian/pool/main/p/protobuf/). Its SHA256 and size match the HTTPS-delivered `protobuf_3.21.12-3+deb12u1.dsc` Checksums-Sha256 field:

- `protobuf_3.21.12.orig.tar.gz`: 5,141,502 bytes; SHA256 `930c2c3b5ecc6c9c12615cf5ad93f1cd6e12d0aba862b572e076259970ac3a53`.
- Debian packaging reference `protobuf_3.21.12-3+deb12u1.debian.tar.xz`: 46,240 bytes; SHA256 `49cdad49801545cb797355af19fbd6b3627084c6549f629512faaa4e25dd5647`.

The Debian packaging archive was read only as a reference. No Debian patch was applied, and this is upstream 3.21.12 with local Debian revision `-1`, not Debian's `-3+deb12u1` security revision. All 2,248 regular files in the extracted source were compared against the original archive after the build and matched. The `.dsc` signature was not independently verified.

## Build isolation

All compilation, packaging, package installation and compiled smoke tests ran in the existing dedicated Codex Podman container `codex-v0.166.0-v8-directory-probe-20261107`, from the project image `localhost/codex-v0.166.0-sdk:rust-1.99.0-2`. The container uses Debian Buster/Loongnix glibc 2.28 and GCC/G++ 8.3. No new Rust container, image-policy override, host installation or host compilation was performed. Inputs, build objects, packaging trees and logs are under the dedicated ext4 cache UUID `5614614b-0d3f-4d00-b47b-d3ec9df1081f`.

The build held `/tmp/chinaos-heavy-build-20261105.lock` and used two jobs. CMake 3.13.4, OpenSSL development files, zlib development files and Python 3.7 were existing distribution packages installed only in this container; they were not rebuilt or relabeled.

CMake used Release `-O2`, PIC, `-z relro,-z now`, `/usr` prefix, `lib/loongarch64-linux-gnu` library directory, shared C++ runtimes and compiler, and zlib support. Upstream unit tests/examples, language bindings and static archives were not built. Installed CMake/pkg-config metadata reports `3.21.12.0` because upstream CMake appends its release component; `protoc --version` reports `libprotoc 3.21.12`.

## Debian package split

Package names, SONAME 32, Multi-Arch roles, headers/proto ownership, exact developer dependencies and the compiler/developer version constraint follow Debian bookworm's protobuf `debian/control`, `debian/rules` and `debian/*.install` from the reference above. All six packages have `Version: 3.21.12-1` and `Architecture: loongarch64`.

- `libprotobuf32`, `libprotobuf-lite32`, `libprotoc32`: actual shared runtime and SONAME symlink, Multi-Arch same, ldconfig activation, shlibs metadata.
- `libprotobuf-dev`: runtime headers/proto files, runtime linker symlinks and pkg-config metadata; exact runtime family dependencies, `zlib1g-dev`, `protobuf-api-32-0`, and the Debian compiler version Breaks constraint.
- `libprotoc-dev`: compiler headers/linker symlink and the full CMake imported-target SDK. Upstream CMake exports both the compiler and all three libraries together, so these export files are owned here with exact dependencies on `protobuf-compiler` and `libprotobuf-dev`. This avoids a circular dependency and ensures that every imported path exists. Debian's reference autotools install does not ship this CMake export family.
- `protobuf-compiler`: the actual versioned protoc ELF and upstream-installed `protoc` symlink, help-derived man page, Multi-Arch foreign; exact compiler/runtime family dependencies and a developer-package recommendation.

No empty language-binding, static-archive, common-data or GIR package was created. No generated installed payload was dropped: 149 source-install files/symlinks have one owner each. The library SONAMEs coexist with the pre-existing Buster `libprotobuf17`/`libprotoc17`. The compiler replaces the older `protobuf-compiler` through its normal package name.

External ELF dependencies were calculated using container `dpkg-shlibdeps` and actual staged SONAME mappings. They include glibc >=2.28, libgcc1, libstdc++6 and zlib for the full runtime. The oldworld repository supplies only an unversioned libstdc++ shlibs entry; a conservative `libstdc++6 (>=8.3.0)` constraint records the tested GCC 8 runtime rather than dropping its version requirement. No Rust, CMake, Python, OpenSSL or build-essential runtime dependency was added.

Each package includes the full upstream LICENSE and Debian reference copyright information, with the complete GPL-2/GPL-3/Apache-2.0 texts referenced by that reference document. Native code in this C++ build is the upstream BSD-licensed Protobuf implementation.

## Validation

The complete package family is installed and configured in the dedicated Buster container. Initial old-APT local-file pathname failure and alphabetic dpkg ordering against the old compiler are retained in logs. The final explicit order (runtimes, new compiler, developer packages) succeeded without force/ignore-dependency flags. `dpkg --audit`, each package's `dpkg --verify`, and `apt-get check` passed. This is an upgrade/use verification in the existing project container, not a claim of a newly created clean base.

- 179 installed regular files and 7 symlinks matched final package staging hashes/targets/modes; installed owner/group are root. All final package SHA256 values were checked.
- Four installed ELF files are LoongArch machine 258, flags `0x3`. Protoc uses `/lib64/ld.so.1`; maximum required GLIBC symbol version is 2.27 for all four. Dependency resolution passed with no `LD_LIBRARY_PATH` or `LD_PRELOAD`.
- `protoc --version`, both pkg-config modules and the installed CMake imported targets passed.
- The real Codex code-mode proto schema compiled to a 7,035-byte descriptor set.
- Generated C++ full and lite programs preserved the distinction between an absent optional integer and a present zero, and passed serialization/deserialization roundtrips.
- The installed compiler SDK imported the fixture and identified its optional field presence through the descriptor API.

These are focused installed-SDK checks; no upstream full-suite result is claimed. The small smoke programs are temporary test fixtures outside all source repositories and are not reusable distribution tools.

## Artifacts

Formal location: `binaries/{package}/loongarch64/{package}_v3.21.12_loongarch64.deb`, with matching `.deb.sha256` files.

| Package | Bytes | SHA256 |
| --- | ---: | --- |
| libprotobuf32 | 706928 | `cc3aab5cca1d546522d1694ee7adf8b468d143bf1c910c7880e90c3cd772adef` |
| libprotobuf-lite32 | 165368 | `3e02c2d982b2118068b8633553bfe0e126ff4b7f4b7eb8902d58c9aae0490898` |
| libprotoc32 | 721624 | `0e8ebeb871fd6c77bfb18548c480152a5f325dd2552fd604785709a71157c7d7` |
| libprotobuf-dev | 332200 | `4fc45030e2e4195832a6658128fa1d96e9537a3e69a426d10a8e1522e7bcdfdc` |
| libprotoc-dev | 68268 | `6524b5e6adea0a174cb88e3e64f1d3971fc050df11396de613726ed17e8fe1aa` |
| protobuf-compiler | 27864 | `120a8b6e7107ad487220be6f1774f0290ddc50ba90718bfbc914b10907e88853` |

## Evidence

Cache root: `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/`.

- `cargo-home/protobuf-tool-20261107/inputs/verified-inputs.json`
- `cargo-home/protobuf-tool-20261107/package-manifest.json` and `formal-deliverables.json`
- `cargo-home/protobuf-tool-20261107/installed-verification.json`
- `logs/main-protobuf3212-build.log` and `main-protobuf3212-build-state.json`
- `logs/main-protobuf3212-group-install-ordered.log`
- `logs/main-protobuf3212-installed-verification.log`
- Earlier package/installation attempts remain separately named in the same log directory; withdrawn metadata-only candidates are not release artifacts.
