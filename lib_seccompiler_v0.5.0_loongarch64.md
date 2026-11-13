# seccompiler 0.5.0 LoongArch64 依赖适配报告

适配日期：2026-10-29；Debian/Ubuntu 打包审查更新：2026-11-07。类型：Rust 库，供 Codex CLI 使用。`-dev` 包判定依据见本报告的 Debian/Ubuntu 审查，不以“无应用 .deb”作为打包豁免理由。

## 来源和提交

- 被适配仓库 GitHub 地址：https://github.com/rust-vmm/seccompiler
- 官方基线：`v0.5.0`，提交 `c3cf77d65815037931ae5bc2fca010713defdc8c`。
- Codex 官方 `Cargo.lock` 中 seccompiler 0.5.0 的 crate checksum：`a4ae55de56877481d112a559bbc12667635fdaf5e005712fd4e2b2fa50ffc884`。专用容器内下载的原始 crate 已校验相符。
- 分支：正式上游版本 → `debian10`（保持基线）→ `loong64`。
- 架构支持提交：`9e0a1f66e49c148ceb42b5d4b302d3ec84937236`，`feat: Add LoongArch64 support with architecture and seccomp tests`。
- 独立测试兼容提交：`8bb8d11bbd729d6b1a64f2aa1ca466cc802f043b`，`test: Check TSYNC syscall failures without assuming a cleared errno`。
- 最终源码工作区干净。源码提交只包含必要支持、生成表、相关说明和测试，未加入 Containerfile、构建缓存、产物或本机路径。

## 必要改动

新增 `TargetArch::loongarch64`、不区分大小写的字符串解析及 `AUDIT_ARCH_LOONGARCH64 = 0xc0000102`。架构检查仍在放行 syscall 前执行，错误架构仍返回 `SECCOMP_RET_KILL_PROCESS`，没有关闭或绕过 sandbox。

依据 Linux 6.12 生成独立的 LoongArch64 JSON syscall 表，共 314 项，并扩展现有生成和校验工具。没有复用其它架构的私有 syscall。常量来源为 Linux 官方 UAPI：

- https://github.com/torvalds/linux/blob/master/include/uapi/linux/audit.h
- https://github.com/torvalds/linux/tree/v6.12/arch/loongarch
- Linux 6.12 源码包 SHA256：`b1a2562be56e42afb3f8489d4c2a7ac472ac23098f1ef1c1e40da601f54625eb`。镜像下载后与 kernel.org 发布的 `sha256sums.asc` 核对通过。

TSYNC 原测试在成功的 `getpid()` 后要求 errno 为零；本机实际保留了先前 errno=2，导致过滤安装前的断言失败。成功调用不保证清零 errno。修正后的测试使用 `libc::syscall(SYS_getpid)`，成功时检查 PID，拒绝时同时严格检查返回 -1 和 EPERM；保留跨线程同步过滤检查，没有削弱安全断言。

## 验证结果

原始 crate 的默认和 JSON 基线构建均成功。适配后运行结果如下，全部通过、没有忽略失败：

| 检查 | 默认配置 | 启用 json |
| --- | ---: | ---: |
| 库单元测试 | 13 | 17 |
| 架构安全集成测试 | 3 | 4 |
| 原有集成测试 | 11 | 11 |
| JSON 集成测试 | 0 | 4 |
| TSYNC 跨线程测试 | 1 | 1 |
| 文档测试 | 5 | 5 |
| 合计 | 33 | 42 |

新增测试对四种架构的 BPF 架构检查指令作精确断言，并在本机隔离子进程内验证：正确架构允许 getpid、拒绝 getppid 返回 EPERM、KillProcess 触发 SIGSYS，以及其余三个错误目标架构均触发 SIGSYS。JSON 生成结果与同等显式策略相同，原生 JSON 策略也经过实际执行。

Linux 6.12 的 x86_64、aarch64、riscv64、loongarch64 四套 syscall 表均重新生成比对通过。`cargo fmt --check` 通过。交付的合并补丁在原始 0.5.0 crate 上执行 `git apply --check` 成功；原始缓存来自 NTFS，因此该项检查出现文件权限模式警告，补丁本身没有引入权限变更。

已实际执行的 10 个测试 ELF 均检查为：ELF64 little-endian、`EM_LOONGARCH=258`、`EI_ABIVERSION=0`、flags `0x3`、loader `/lib64/ld.so.1`，最高 glibc 符号需求为 `GLIBC_2.28`。这验证了本机旧 ABI 测试链接产物；本报告不宣称已在其它物理架构上执行。

## 隔离环境和复现

全部编译、链接、内核头生成和运行测试使用 seccompiler 专属 Podman 镜像 `localhost/seccompiler-build:0.5.0-buster-loongarch64-v0` 和容器 `seccompiler-build-050`。基础镜像复用已有 Codex/buster 构建基座；专属镜像由独立临时容器提交生成，未修改 Codex 容器。基座 glibc 为 2.28，Rust SDK 为 1.98.1-v0，使用容器内 GCC 8.3 链接。rsync 只补入专属容器。宿主没有安装任何构建依赖或修改全局镜像策略。

容器内 `/work/source` 为测试源码，`/cache` 为项目专用外盘缓存。本次实际执行的命令为：

```bash
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo fmt --check
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo build --locked --release
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo build --locked --release --features json
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo test --locked -- --test-threads=1
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo test --locked --features json -- --test-threads=1
```

本次执行时的逐项结果已核对并记录在上表。任务结束后，`/tmp/seccompiler-build-20261029/` 和其 Podman wrapper、容器运行目录已从当前环境清理，故临时 `cargo-tests.log` 等日志不再保留；外盘的 `target` 中保留了全部十个测试 ELF，ABI 复核结果见上文。重新运行时须先按本机 Podman 配置重建专用镜像与容器，再挂载本仓库、Rust SDK、缓存及源代码；不应复用其他项目容器。完整源码补丁、两份逻辑提交补丁、Cargo 缓存及 target 位于 `/media/laevatein/VCLIB/ChinaOSProject-cache/seccompiler/0.5.0/`。外盘其他既有缓存文件未被改动。

## Debian/Ubuntu Rust `-dev` 包审查（2026-11-07）

依据根 `AGENTS.md` §6 检查发行版中是否存在可对应本库的 Rust 源码开发包。Debian `bookworm`、`trixie`、`sid` 和 Ubuntu `noble`、`questing`、`resolute` 的官方包名检索均未找到 `librust-seccompiler-dev`；Debian Sid 2026-10-31 `Sources` 快照也已按每个 stanza 的 `Package:` 与 `Binary:` 字段逐项检索，没有任何包含 `seccompiler` 的匹配。该快照 SHA256 为 `da7676dff4c8db29787fb4f8bb338a98ecf8372948c8c7e7346261853e217a83`；主审提供的检索记录位于 `/mnt/chinaos-cache/ChinaOSProject-cache/main-review-20261107/debian-seccompiler-source-reference.json`，本次对压缩索引的独立复核同样得到 0 个匹配。

检索到的相近名字并非本 crate 的 Debian 打包：`librust-seccomp-sys-dev` 对应 `rust-seccomp-sys`，提供较底层的 libseccomp 绑定；`librust-libseccomp-dev` 对应 libseccomp 的 Rust 绑定。这两者都不是 `rust-vmm/seccompiler`，不能作为本库的包名、API 或 feature 依赖拆分依据。官方 `librust-seccompiler-dev` 名称检索入口：[Debian](https://packages.debian.org/search?keywords=librust-seccompiler-dev&searchon=names&suite=all&section=all)、[Ubuntu](https://packages.ubuntu.com/search?keywords=librust-seccompiler-dev&searchon=names&suite=all&section=all)。相近但不同的发行包可见 [Debian `rust-seccomp-sys` 打包控制文件](https://sources.debian.org/src/rust-seccomp-sys/0.1.3-1/debian/control/)、[Debian `librust-libseccomp-dev`](https://packages.debian.org/trixie/librust-libseccomp-dev) 和 [Ubuntu seccomp 包索引](https://packages.ubuntu.com/seccomp)。

本 crate 上游 `Cargo.toml` 声明的基础依赖是 `libc = ^0.2.153`；`json` feature 启用可选依赖 `serde = ^1.0.27`（含 `derive`）和 `serde_json = ^1.0.9`。源码没有 `src/bin` 或 `src/main.rs`，也没有声明独立二进制目标。由于 Debian/Ubuntu 没有可对应的独立发行包，发行版没有可遵循的本 crate feature/Depends 拆分。按 `AGENTS.md` §6“没有常见 Debian/Ubuntu 独立发行包对应”的例外，本次不制造 `librust-seccompiler-dev` 或其他虚构 `.deb`；也没有运行打包容器、安装依赖或重跑既有测试。若未来发行版正式收录该 crate，再按当时实际包控制字段更新分包和依赖。

## 可复用交付和状态

- 完整适配补丁随本报告同目录归档：[seccompiler-0.5.0-loongarch64.patch](./seccompiler-0.5.0-loongarch64.patch)，SHA256：`1e450a47751227bae98fa53ce8589590e758b2d1a7b52b01140c1d7fc9e71424`。该补丁基于可公开获取的 [官方 v0.5.0 标签](https://github.com/rust-vmm/seccompiler/tree/v0.5.0)，在原版源码上执行 `git apply` 即可复现。2026-11-13 已将 `debian10` 基线 `c3cf77d65815037931ae5bc2fca010713defdc8c` 和包含两个适配提交的 `loong64` 分支 `8bb8d11bbd729d6b1a64f2aa1ca466cc802f043b` 推送至 https://github.com/ChinaOSProject/seccompiler ，并核对远端 refs。fork 的默认分支未改动；两份按提交拆分的 patch 也保留在专属源码缓存中。
- [本地完整源码归档](/media/laevatein/VCLIB/ChinaOSProject-cache/seccompiler/0.5.0/seccompiler-0.5.0-loongarch64.tar.gz)，SHA256：`55e884b8042af8ec1dbd54c24812438b150335862690cf317fdc6bd0fc3f0876`。
- 本库的适配和验证已完成；按上文发行版索引证据，本次不生成 `.deb`。Codex 本体仍需选择新架构、以可复现方式集成此依赖，并完成其完整 CLI 构建与安全验证；不能将本库通过测试视为 Codex 已完成升级。
