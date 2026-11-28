# Codex CLI 0.172.0-1+chinaos1 LoongArch64 适配

## 源码与目标

- OpenAI 正式版 `rust-v0.172.0`，提交 `3d86e43b075adeba2193ec5c22ac92f55afd82e6`。
- `ChinaOSProject/codex` 的 `loong64` 提交 `c59efe92d18884435f82296b0e107c4d1af57fa9`。
- 目标为 Kylin V10 SP1 `loongarch64`，Rust 目标 `loongarch64-unknown-linux-gnu`，宿主 glibc `2.28`。
- `Cargo.lock` SHA-256 `55fc8f850ad1dab618730aeb284b6c51d100d7982c78a13f87e18ba9340e9d0b`。

## 构建与分包

- 使用已有旧 ABI `rustc 1.99.1-dev`、`Cargo 1.99.1-dev`、LLD `21.1.8`、GCC 和 Protobuf `3.21.12`。Cargo 使用 `--locked --offline --release`，关闭 LTO 并设置 16 个 codegen units。
- CLI 的 LoongArch 目标依赖使用 `-C code-model=large` 生成长距离调用，解决旧 ABI 短分支溢出，并关闭行表调试信息。配套程序沿用首次成功构建。
- 将 Rust 分配器跳板放在 `.text` 末端，靠近预构建标准库与 PLT，确保标准库的旧 ABI 短分支可达。
- 从 Kylin 发行包提取 OpenSSL 和其他开发文件至构建缓存。OpenSSL 多架构头文件在该目录内通过相对链接暴露给 `openssl-sys`。
- RustyV8 `152.2.0` 的两个提交 `350c5a859a9b3c84ab8701b3ad51e3b7699b30c0` 与 `90a65f887a0c0f048f3784afbcb320b373c8a2e0` 的根目录条目与 Git 对象完全一致。使用同一原生归档和对应绑定，Rust 源文件按锁定提交的 Git blob 校验。
- Code Mode host 和执行服务器各自分包，二进制位于上游 CLI 布局的同一个 `bin/` 下。主包以精确版本依赖配套包。响应 API 代理作为独立工具包。`rg` 由 `ripgrep` 包提供。Codex 专用 bubblewrap 位于主包的资源目录，文件字节与编译时摘要一致。
- 运行时共享库依赖通过目标机 `dpkg-shlibdeps` 生成。参考 Debian 发行的 `ripgrep` 与 `bubblewrap` 分包，Codex 专用 bubblewrap 使用私有路径。
- 本次交付 CLI 及其构建矩阵中的原生配套程序。上游 Linux 语音运行时的构建入口仅配置 `x86_64` 和 `aarch64`，LoongArch64 语音运行时需要独立移植。

## 产物

| 文件 | 字节数 | SHA-256 |
| --- | ---: | --- |
| `codex-code-mode-host/loongarch64/codex-code-mode-host_v0.172.0-1+chinaos1_loongarch64.deb` | 23065356 | `8d8c1117e37e644158e0eab9010fa96c64220aaec7799533292b5f1d9edc12a8` |
| `codex-exec-server/loongarch64/codex-exec-server_v0.172.0-1+chinaos1_loongarch64.deb` | 23062952 | `aa0857829f9f7edce83a84adea940aaabe8eb59ab219e2c5c60c711e100eaf2b` |
| `codex-responses-api-proxy/loongarch64/codex-responses-api-proxy_v0.172.0-1+chinaos1_loongarch64.deb` | 2936940 | `b9789c6d735217668f4f5b303a1fe2ffed2c9b2377dec1968e60b9e754509d4c` |
| `codex/loongarch64/codex_v0.172.0-1+chinaos1_loongarch64.deb` | 105341256 | `ade5ed0f25cfbe4a81c3d83a0b0837dfe42283a0a12e1bc0824585ad34a71b48` |

## ABI 与验收

所有下列 ELF 均为 LoongArch64 旧 ABI `0x3`，解释器为 `/lib64/ld.so.1`。

| 程序 | 最高 GLIBC 符号版本 | 动态依赖 |
| --- | --- | --- |
| `codex-code-mode-host` | `2.28` | `ld.so.1`, `libc.so.6`, `libcrypto.so.1.1`, `libdl.so.2`, `libgcc_s.so.1`, `libm.so.6`, `libpthread.so.0`, `libssl.so.1.1` |
| `codex-exec-server` | `2.28` | `ld.so.1`, `libc.so.6`, `libcrypto.so.1.1`, `libdl.so.2`, `libgcc_s.so.1`, `libm.so.6`, `libpthread.so.0`, `libssl.so.1.1`, `libutil.so.1` |
| `codex-responses-api-proxy` | `2.28` | `ld.so.1`, `libc.so.6`, `libcrypto.so.1.1`, `libdl.so.2`, `libgcc_s.so.1`, `libm.so.6`, `libpthread.so.0`, `libssl.so.1.1` |
| `codex` | `2.28` | `ld.so.1`, `libbz2.so.1.0`, `libc.so.6`, `libcrypto.so.1.1`, `libdl.so.2`, `libgcc_s.so.1`, `libm.so.6`, `libpthread.so.0`, `libssl.so.1.1`, `libutil.so.1` |
| `bwrap` | `2.28` | `ld.so.1`, `libc.so.6`, `libcap.so.2`, `libgcc_s.so.1`, `libpthread.so.0` |

- 上游打包测试 27 项通过。已解包验证 CLI 版本与各入口，验证 Linux 沙箱执行及写入限制、Code Mode gRPC JavaScript 执行、app-server stdio 初始化。
- 在目标机 `apt-get -s install` 联合解析四个包的依赖。

## 参考

- [上游正式发布](https://github.com/openai/codex/releases/tag/rust-v0.172.0)
- [Codex 发行构建与布局](https://github.com/openai/codex/blob/rust-v0.172.0/.github/workflows/rust-release.yml)
- [Debian 10 bubblewrap 0.3.1-4 打包源码](https://archive.debian.org/debian/pool/main/b/bubblewrap/bubblewrap_0.3.1-4.debian.tar.xz)
- [Debian rust-ripgrep 源码包及 `ripgrep` 分包](https://packages.debian.org/source/trixie/rust-ripgrep)
- [Debian Policy 共享库](https://www.debian.org/doc/debian-policy/ch-sharedlibs.html)
