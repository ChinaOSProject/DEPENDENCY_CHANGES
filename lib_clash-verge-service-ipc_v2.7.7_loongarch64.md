# Clash Verge Service IPC v2.7.7 for LoongArch

日期：2026-11-07

上游仓库：[clash-verge-rev/clash-verge-service-ipc](https://github.com/clash-verge-rev/clash-verge-service-ipc)

上游版本：`v2.7.7`，提交 `3ed29700b093190b5e7addcaa892e77f172af544`

Loong64 适配提交：`eaaae99bb24c77155dbea95b3964f6d5be42b0f1` (`fix(service): secure IPC socket permissions by inode`)

分支状态：`main` 精确指向上游 v2.7.7；`debian10`、`loong64` 均在该上游提交之上。源码工作树干净。补丁只修改 `src/core/manager.rs`，包含 Linux 兼容实现和定点回归测试；v2.7.6 归档 refs 与已发布包未修改。

2026-11-13 源码发布：`debian10`、`loong64` 已推送至 https://github.com/ChinaOSProject/clash-verge-service-ipc ，两分支的远端提交均核对为 `eaaae99bb24c77155dbea95b3964f6d5be42b0f1`。fork 的默认分支保持原样；本轮没有重跑既有构建或测试。

## 问题与修复

目标设备为 LoongArch64、Linux `5.4.18-168-generic`、glibc `2.28`。v2.7.7 上游 `secure_core_ipc_socket` 使用 `fchmodat(..., AT_SYMLINK_NOFOLLOW)` 修改 core IPC socket 权限，在该旧基座上返回 `EOPNOTSUPP`，阻断 core 启动。原版定点复现记录在 `logs/v277-baseline-repro.log`，测试进程以预期失败码 `101` 退出，错误为 `Operation not supported (os error 95)`。

Linux 分支现在先用 `O_PATH | O_NOFOLLOW | O_CLOEXEC` 固定 socket inode，校验 fd 指向 socket 且 dev/ino 与 `symlink_metadata` 一致。root 的 chown 与 chmod 都作用于 pinned fd。chmod 优先尝试 `fchmodat(fd, "", 0600, AT_EMPTY_PATH)`；仅 `EINVAL`、`ENOSYS`、`EOPNOTSUPP` 进入 `/proc/self/fd/<fd>` 回退，其他错误直接返回。回退期间保持 fd 打开，完成后再检查路径 inode；procfs 不可用时失败关闭。非 Linux Unix 分支保留上游路径。

## 源码验证

v2.7.7 的原版复现与 patched 定点回归已在此前完成，本次没有重跑这些回归或完整 crate suite：

- root 和非 root 各运行 `owner_core_socket_is_private`、`core_socket_permissions_stay_on_pinned_inode`、`core_socket_rejects_replacement_before_pinning`、`core_socket_rejects_symlink`，均通过。记录在 `logs/v277-patched-owner-{root,nonroot}.log`、`logs/v277-patched-security-{root,nonroot}.log`；汇总状态 `logs/v277-patched-tests.status` 为 `0`。
- 在目标旧 kernel 上，外置观察 shim 记录 `AT_EMPTY_PATH` 的 libc 调用返回 `EINVAL`，随后 `/proc/self/fd/<fd>` fallback 成功；strace 显示 chown 与 fallback 针对同一 pinned fd。记录在 `logs/v277-real-kernel.stdout`、`logs/v277-real-kernel.strace`。
- 外置故障注入覆盖 `EPERM`、`EACCES` 及 procfs fallback 的 `ENOENT`、`EACCES`；这些注入路径均失败关闭，记录在 `logs/v277-EACCES.*`、`logs/v277-EPERM.*`、`logs/v277-proc-*.stdout`。这是故障注入结果，不是目标 kernel 上自然发生的 errno。
- v2.7.7 patched release 已构建，`logs/v277-release-build.status` 为 `0`。使用现有 Rust `1.99.0` LoongArch 容器工具链，release 构建最多 2 jobs；本次收尾没有重编源码。

## Debian 运行包

二进制包：`clash-verge-service`，版本 `2.7.7-1`，架构 `loongarch64`，`Installed-Size: 7636`。

包内文件：

- `/usr/bin/clash-verge-service`
- `/usr/bin/clash-verge-service-install`
- `/usr/bin/clash-verge-service-uninstall`
- `/usr/share/doc/clash-verge-service/copyright`（GPL-3.0 全文）

三个程序放在同一目录，因为上游 installer 从自身可执行文件旁查找 daemon。包不含 maintainer scripts、systemd unit 或可单独拆分的公共数据文件；服务 unit 由上游 installer 在管理员安装流程中生成。包未自动启用或启动服务。

Depends：`libc6 (>= 2.28), libgcc1 (>= 1:4.2), systemd`。`dpkg-shlibdeps` 对三个程序计算出 `libc6 (>= 2.28), libgcc1 (>= 1:4.2)`；额外的 `systemd` 来自 installer/uninstaller 对 `systemctl` 的运行时调用。目标容器中实装版本为 `libc6 2.28-10.lnd.38`、`libgcc1 1:8.3.0-6.lnd.vec.44`、`systemd 241-7.lnd.13`。

三个已安装 ELF 均为 ELF64 little-endian LoongArch、LP64 flags `0x3`，解释器 `/lib64/ld.so.1`，最高符号要求 `GLIBC_2.28`。已安装文件与 release 产物逐字节一致，`dpkg --audit` 与 `dpkg -V clash-verge-service` 无输出，安装状态为 `ii`。完整的 `readelf`、`ldd`、dpkg 元数据和逐文件 SHA-256 见 `logs/v277-verify-installed-package.log`。

许可证校验：包内 copyright 文件与 v2.7.7 源码 `LICENSE` 字节一致，均为 35,148 字节，SHA-256 `8b1ba204bb69a0ade2bfcf65ef294a920f6bb361b317dba43c7ef29d96332b9b`。

## 服务 smoke 与 systemd 边界

候选 `.deb` 已在本任务的独立 Podman 验证容器中通过 apt 安装。验证容器复用本任务已拥有的 Podman store 和原 `localhost/clash-verge-build:2.5.8-buster-loongarch64` 镜像，创建时使用 `--cgroups=disabled`；仅绑定本任务 `/work` 与只读源码 `/src`。旧 `service-ipc-build-2.7.6` 容器保持原状且没有活跃构建或服务进程（进程为 `/bin/sleep infinity`）。验证容器名为 `service-ipc-v277-verify-20261107`。

从已安装的 `/usr/bin/clash-verge-service` 实际启动 daemon 后：

- socket `/run/clash-verge-service/service.sock` 建立，模式为 `0666 root:root`；客户端必须提供 `X-IPC-Magic`。
- 随包构建的 `service-integration-driver probe` 成功，完成 magic 握手和协议兼容检查。
- `GET /version` 返回 `build_version=2.7.7`、协议 epoch `2`、revision `6`、最低客户端 revision `5`。
- daemon 收到 SIGTERM 后以状态 `0` 退出并释放 owner lock，IPC socket 随之移除。

容器的 PID 1 是 `sleep`。`systemctl --version` 显示 systemd `241`，但 `systemctl is-system-running` 返回 `offline`、退出码 `1`。`systemd-analyze verify` 对由上游 `resources/systemd_service_unit.tmpl` 渲染的 unit 静态校验通过；dpkg 安装后未生成或启用 `clash-verge-service.service`。由于容器没有 systemd PID 1，此处没有实际验证 installer 的 unit enable/start，也没有验证真实桌面会话下的管理员安装流程。此次 smoke 检查 IPC daemon、magic/version 协议和关闭清理，不启动 Mihomo core，也不代表 core Start/Stop 的端到端验证。记录在 `logs/v277-package-service-smoke.log`、`logs/v277-service-smoke-daemon.log`、`logs/v277-service-smoke-version.json`、`logs/v277-service-smoke-socket.txt`、`logs/v277-service-smoke-shutdown.txt`、`logs/v277-systemd-container-boundary.log`、`logs/v277-systemd-unit-static-check.log`。

## Debian 分包依据

分包结构参考 Debian Buster 的 [transmission 2.94-2+deb10u2 `debian/control`](https://sources.debian.org/data/main/t/transmission/2.94-2+deb10u2/debian/control)：将 daemon 与 GUI 分开，只有实际共享的数据才拆成 common 包；共享对象依赖由 `dpkg-shlibdeps` 生成。这里沿用上游实际命名，将 daemon 及必须相邻的 installer/uninstaller 放入一个运行包。没有把构建依赖写成运行依赖，也没有人为创建无实际用途的 `-common` 或库包。

## 产物与保留状态

最终候选包大小 `1,492,128` bytes，SHA-256：

`f64abe83d2cbd84185546025dda25c22bb4a1888e17b8f7ec1c5c86d1c6ac636`

本地交付候选位于 `../binaries/clash-verge-service/loongarch64/clash-verge-service_v2.7.7_loongarch64.deb`，已由主 Agent 完成审核，随对应二进制包统一归档发布。副本与原始包及新缓存候选逐字节一致；新缓存挂载为 ext4，UUID `5614614b-0d3f-4d00-b47b-d3ec9df1081f`，目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/service-ipc/v2.7.7/`。

v2.7.6 已发布包保留在原路径，SHA-256 仍为 `aad0c68c71c830af53d784c365a0db92c1766262f193007f0f6805bfda8e7000`。v2.7.6 报告、归档 refs、源码和既有日志均未改动。

Clash v2.5.10 实际编译 crate 清单仍等 wB 的编译输入证据；旧 crate union 与脚本保留原样，未据 Cargo.lock 推测或发布不完整的 `-dev` 清单。
