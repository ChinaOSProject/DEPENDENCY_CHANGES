# esbuild 0.28.2 for LoongArch

This package contains the upstream `evanw/esbuild` 0.28.2 command-line program from commit `609683d892977362a0f99026cb74b96263d728a9`. The locked source files have SHA-256 `2d48dbed6151e653adc9ab5d8869cba012d167f13352e45880e59104e1e02b9b` (`go.mod`) and `f2881c3c2d9d03858770bf7713c82ff7564c0893685cf708e49a111e5fd66ab2` (`go.sum`). The module requires only `golang.org/x/sys v0.0.0-20220715151400-c0bba94af5f8`; the package ships no Go source or development library. It was built with the existing Go 1.27.1 SDK using `go build -p=2 -mod=readonly -trimpath -buildvcs=false -ldflags='-s -w -buildid=' ./cmd/esbuild`.

The Debian package is named `esbuild`, version `0.28.2-1`, architecture `loongarch64`. It installs `/usr/bin/esbuild` and complete license/copyright texts. The executable is a static LoongArch LP64 ELF with flags `0x3`; it has no `PT_INTERP` and no `DT_NEEDED` entries, so it adds no shared-library runtime dependency.

Debian sid packages the same upstream version as `esbuild` (program), `esbuild-wasm` (`Architecture: all`), and `golang-github-evanw-esbuild-dev` (`Architecture: all`). This delivery follows the program package boundary: the build produced only the native CLI, not the WASM distribution or a separately usable Go development tree. The target architecture is recorded as Debian's `loongarch64` value.

The candidate was installed with `dpkg` into an isolated container root. `dpkg --verify esbuild` reported no changed files; the installed command printed `0.28.2` and transformed a TypeScript smoke input into JavaScript. The successful install and transform log is `logs/tool-deb-package-installcheck-v2510-attempt06.log`.

Artifact: `binaries/esbuild/loongarch64/esbuild_v0.28.2-1_loongarch64.deb` (3,374,940 bytes), SHA-256 `2630da0d80ecefc40eb82d4c82d5b26d93a4ea87326f2b3d437077186f28fe6b`.

References: [upstream esbuild](https://github.com/evanw/esbuild), [Debian sid esbuild 0.28.2-1](https://packages.debian.org/sid/esbuild), [Debian sid esbuild-wasm](https://packages.debian.org/sid/esbuild-wasm), and [Debian sid esbuild Go development package](https://packages.debian.org/sid/golang-github-evanw-esbuild-dev).
