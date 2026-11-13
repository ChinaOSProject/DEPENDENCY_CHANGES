# pnpm 12.16.1 for LoongArch

This package contains the native pnpm CLI built from the Rust workspace used by the upstream pnpm 12.16.1 release. The npm release metadata identifies `pnpm@12.16.1` and its `pnpm/pnpm` source directory; the Rust workspace identifies `pnpm/pacquet`. The Rust source `Cargo.lock` SHA-256 is `ef2eb07f977bf767709f259bfc58d81c3e92c3e181c10dccabc97d8eb341c1d1`. The build used Rust 1.99.0 with `cargo build --locked --offline --release --package pnpm-cli --bin pnpm --jobs 2`; the workspace toolchain file requests 1.97.0. The resulting program reports version `12.16.1`.

The Debian package is named `pnpm`, version `12.16.1-1`, architecture `loongarch64`. It installs `/usr/bin/pnpm` and license notices. Its `Depends` are `libc6 (>= 2.28), libgcc1 (>= 1:4.2)`. The ELF is LoongArch LP64 with flags `0x3`, interpreter `/lib64/ld.so.1`, and a maximum required GLIBC symbol version of `GLIBC_2.28`. Its `DT_NEEDED` entries are `libgcc_s.so.1`, `libpthread.so.0`, `libm.so.6`, `libdl.so.2`, and `libc.so.6`; the first is provided by `libgcc1`, and the remaining entries by `libc6` on the target baseline.

Debian's `node-corepack` 0.35.0-2 is the closest packaging comparison: it depends on Node.js and installs Corepack plus shims under `/usr/share/nodejs/corepack/shims/`. This package instead installs a native `/usr/bin/pnpm`; the Debian package file list has no overlap, so no `Conflicts`, `Breaks`, `Replaces`, or `Provides` are added. The upstream npm metadata declares `engines.node >=18`; the native Rust executable has no ELF dependency on Node and starts directly. Operations that execute JavaScript lifecycle scripts still need a suitable Node runtime from the environment; this package does not bundle Node.

The candidate was installed with `dpkg` into an isolated container root. `dpkg --verify pnpm` reported no changed files, and the installed command printed `12.16.1`. The successful install log is `logs/tool-deb-package-installcheck-v2510-attempt06.log`.

Artifact: `binaries/pnpm/loongarch64/pnpm_v12.16.1-1_loongarch64.deb` (15,528,264 bytes), SHA-256 `dcedc5997fbb580acf515d1cb93cb8f355c543303ad287844025b19e94b5456f`.

References: [upstream pnpm](https://github.com/pnpm/pnpm), [pnpm 12.16.1 npm metadata](https://registry.npmjs.org/pnpm/12.16.1), [Debian node-corepack control](https://sources.debian.org/src/node-corepack/0.35.0-2/debian/control), and [Debian node-corepack file list](https://packages.debian.org/sid/all/node-corepack/filelist).
