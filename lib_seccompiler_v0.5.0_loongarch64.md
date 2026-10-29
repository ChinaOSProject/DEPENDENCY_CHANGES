# seccompiler 0.5.0 LoongArch64 依赖适配报告

日期：2026-10-29。类型：仅依赖类 Rust 库，供 Codex CLI 0.165.0 使用；不生成无独立用途的应用 .deb。

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

容器内 `/work/source` 为测试源码，`/cache` 为项目专用外盘缓存。现有环境下可复现：

```bash
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo fmt --all -- --check
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo test
/tmp/codex-build-20261029/podman exec -w /work/source seccompiler-build-050 /work/in-container.sh cargo test --features json
```

原始日志和临时环境位于 `/tmp/seccompiler-build-20261029/`：`cargo-tests.log`、`syscall-table-validation.log`、`abi-evidence.log`、`baseline-build.log`。下载、Cargo target 和补丁缓存位于 `/media/laevatein/VCLIB/ChinaOSProject-cache/seccompiler/0.5.0/`；未格式化或改动外盘既有文件。

## 可复用交付和状态

- [完整适配补丁](lib_seccompiler_v0.5.0_loongarch64.patch)，SHA256：`1e450a47751227bae98fa53ce8589590e758b2d1a7b52b01140c1d7fc9e71424`。在官方 v0.5.0 源码上执行 `git apply` 即可复现源码改动。
- 本地完整源码归档：`/media/laevatein/VCLIB/ChinaOSProject-cache/seccompiler/0.5.0/seccompiler-0.5.0-loongarch64.tar.gz`，SHA256：`55e884b8042af8ec1dbd54c24812438b150335862690cf317fdc6bd0fc3f0876`。
- 本库的适配和验证已完成。Codex 本体仍需选择新架构、以可复现方式集成此依赖，并完成其完整 CLI 构建与安全验证；不能将本库通过测试视为 Codex 已完成升级。
