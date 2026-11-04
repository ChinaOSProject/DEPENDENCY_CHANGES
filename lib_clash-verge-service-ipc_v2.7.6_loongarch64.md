# Clash Verge Service IPC v2.7.6 for LoongArch

日期：2026-11-04

上游仓库：[clash-verge-rev/clash-verge-service-ipc](https://github.com/clash-verge-rev/clash-verge-service-ipc)

上游版本：`v2.7.6`，基线提交 `b2a7427d6681c8d3780ab83a73f4dc21da2f22d6`

适配提交：`e1a6f6668ec8be74943752327cca44859061aefc` (`fix(service): secure IPC socket permissions by inode`)

源码：`debian10` 上单点修复后快进到 `loong64`；未 push。`src/core/manager.rs` SHA-256：`2c00f9bca260742cc2df0e6d04812f162776867554985d957c38ac1b05a2c67a`

主 Agent 归档的补丁：[clash-verge-service-ipc_v2.7.6_socket-permissions.patch](https://github.com/ChinaOSProject/DEPENDENCY_CHANGES/blob/main/patches/clash-verge-service-ipc_v2.7.6_socket-permissions.patch)

## 问题与修复

在 Linux 5.4 / glibc 2.28 上，旧实现用 `fchmodat(AT_FDCWD, path, 0600, AT_SYMLINK_NOFOLLOW)` 设置 core IPC socket 权限时，glibc 包装函数返回 `EOPNOTSUPP`，没有发起 chmod 系统调用，导致 core 启动功能阻塞。不能忽略该错误，也不能在路径可能被替换时直接 `chmod(path)`。

Linux 实现现在用 `O_PATH | O_NOFOLLOW | O_CLOEXEC` 固定 socket inode，再检查 fd 指向的对象是 socket，且 dev/ino 与此前的 `symlink_metadata` 结果一致。由于 `O_PATH | O_NOFOLLOW` 可以打开 symlink inode，代码还显式拒绝非 socket。root 执行的 chown 和后续 chmod 都作用于同一固定 fd。

chmod 先调用 libc `fchmodat(fd, "", 0600, AT_EMPTY_PATH)`。仅 `EINVAL`、`ENOSYS`、`EOPNOTSUPP` 会尝试 `/proc/self/fd/<fd>`；`EACCES`、`EPERM` 等其他错误直接返回。fd 在回退期间保持打开，procfs 不可用或不可访问时操作失败关闭。完成后再次检查原路径的 dev/ino；路径替换不会把权限操作转向替代文件。非 Linux Unix 路径继续使用原来的 `lchown` 和 `fchmodat(..., AT_SYMLINK_NOFOLLOW)` 逻辑。

## 验证

构建和测试在专属 LoongArch 容器内执行，使用已验证的 Rust 1.99.0 ABI1.0 SDK。目标设备信息为 `loongarch64`、Linux `5.4.18-168-generic`、glibc `2.28`；容器使用 Loongnix GNU/Linux 20 用户态。构建容器为 `service-ipc-build-2.7.6`，与 Clash 共用不可变基座镜像，但源码、SDK 输入、Cargo 缓存和 target 目录独立；没有修改 Cargo 缓存中的依赖源码。

目标测试在 root 与非 root 下各运行 4 项，均通过：

- `core_socket_permissions_stay_on_pinned_inode`
- `core_socket_rejects_replacement_before_pinning`
- `core_socket_rejects_symlink`
- `owner_core_socket_is_private`

在目标旧基座上，外置观察 shim 记录最终实现的 libc `fchmodat(fd, "", 0600, AT_EMPTY_PATH)` 返回 `EINVAL`，随后 procfs 调用成功；这一轮没有注入错误。`EINVAL` 来自旧 glibc 包装函数，strace 中没有对应的 AT_EMPTY_PATH chmod 系统调用。strace 可见真实的 `fchownat(fd, "", ..., AT_EMPTY_PATH)` 和 `/proc/self/fd/<fd>` 上的 chmod，确认两者针对同一 pinned fd；socket inode 最终为 `0600`。替换原路径后，victim 普通文件仍为 `0640`，未被 chown/chmod。

另用外置 `LD_PRELOAD` 故障注入验证错误分支：向 AT_EMPTY_PATH 调用注入 `EPERM` 或 `EACCES` 时返回错误且没有 procfs 尝试；向回退注入 `ENOENT` 或 `EACCES` 时也返回错误。它们是故障注入结果，不代表真实 kernel 的 errno，也不表示本次实际卸载了 procfs。

最终 release 包内 daemon 在目标 LoongArch 容器中实际启动。携带正确 `X-IPC-Magic` 的 Unix socket `GET /version` 返回 `build_version=2.7.6`、协议 epoch `2`、revision `5`、最低客户端 revision `5`；独立 `service-integration-driver probe` 成功。daemon 收到 SIGTERM 后退出并移除 IPC socket。此次 smoke 以 root 在隔离容器内运行；没有在真实桌面会话中安装或启用服务。

针对性 rustfmt 检查和提交 `git show --check` 通过。最终实现没有重跑整个 crate 测试套件，不能据这些定点测试声称全套通过。早期完整 lib 对照记录为：原 tag 79 项通过、13 项失败、1 项忽略；早期补丁 83 项通过、12 项失败、1 项忽略。其中原有的 socket 权限测试转为通过，其余 12 个同名失败仍在：10 项缺少经管理员安装的 mihomo core fixture，1 项启动 reconciliation 前置条件不满足，1 项 legacy cleanup 断言失败。早期对照日志为 `logs/baseline-lib.log` 与 `logs/patched-lib.log`，不作为最终提交的全套验证。现代 kernel/glibc 的 `AT_EMPTY_PATH` 直接成功分支未实机验证。

## ABI 与运行依赖

daemon、installer、uninstaller 均为 ELF64 little-endian LoongArch，machine `258`、flags `0x3`，解释器 `/lib64/ld.so.1`，GNU/Linux ABI tag 为 `4.15.0`，最高引用的 GLIBC 符号版本为 `GLIBC_2.28`。在容器中通过本机动态加载器执行了 daemon smoke；动态库包括 `libc.so.6`、`libgcc_s.so.1`，以及 glibc 2.28 中分出的 pthread、m、dl 组件。

## Debian 交付

正式包：[clash-verge-service_v2.7.6_loongarch64.deb](https://github.com/ChinaOSProject/binaries/blob/main/clash-verge-service/loongarch64/clash-verge-service_v2.7.6_loongarch64.deb)，1,475,200 字节。

SHA-256：`aad0c68c71c830af53d784c365a0db92c1766262f193007f0f6805bfda8e7000`

包元数据为 `clash-verge-service` `2.7.6-1`、架构 `loongarch64`，Depends 为 `libc6 (>= 2.28), libgcc1 (>= 1:4.2), systemd`。`libc6` 与 `libgcc1` 下限来自 `dpkg-shlibdeps`；installer/uninstaller 会运行 `systemctl`，因此包依赖 systemd。上游 v2.7.6 没有 Debian packaging/control 定义可复用。

分包参考 Debian Buster 的 [transmission 2.94-2+deb10u2 control](https://sources.debian.org/data/main/t/transmission/2.94-2+deb10u2/debian/control)：将原生 daemon 与 GUI 分开，实际共有资料才独立放入 `-common`，动态依赖由 shlibdeps 生成。这里使用上游实际程序名 `clash-verge-service`；Debian 参考中没有该同名软件包，未冒称其已有官方发行包。本项目的差异是保留依赖相邻路径的上游 installer/uninstaller，并由其生成 unit，因此三个程序共同交付，且不照抄 Transmission 的 `lsb-base`、独立服务账户或 `transmission-common` 依赖。本包没有足以单独交付的公共数据文件；库开发内容按下述 crate 清单另行处理。

包内有 `clash-verge-service`、`clash-verge-service-install`、`clash-verge-service-uninstall` 三个 `/usr/bin` 程序及 GPL-3.0 许可证。三个程序放在同一目录，因为 installer 通过自身可执行文件路径查找旁边的 daemon。安装包本身没有 maintainer script，不会创建 unit、启用或启动服务。管理员或 Clash 的提权安装流程可调用：

```sh
sudo /usr/bin/clash-verge-service-install --install-service
```

上游 installer 会从旁边复制 daemon、写入 `/etc/systemd/system/clash-verge-service.service`，然后执行 `systemctl daemon-reload`、`enable`、`start` 并等待 IPC ready。卸载走配套的 `clash-verge-service-uninstall`。包已在专属容器中解包，并使用解包后的 daemon 完成上述 IPC smoke。主 Agent fresh apt 安装校验通过：版本 `2.7.6-1`、架构 `loongarch64`，三个已测程序 SHA-256 与构建产物相同，dpkg audit 正常，且没有自动启用服务 unit。其独立 ELF 检查确认三个 ELF machine `258`、flags `0x3`，无 setuid 位或危险路径。

本包只承载运行程序。Rust crate 开发包会在同任务的独立交付中继续盘点和生成，按 Debian 实际 crate 包名、版本并存关系、许可及 Cargo 离线消费要求处理；不会把它们混入这个 service runtime 包，也不会仅因 Rust 代码静态嵌入而跳过该项。

最终验证记录位于 `/tmp/service-ipc-20261102/logs/`：`final-targeted-preformat.log`、`final-targeted-nonroot.log`、`final-libc-path.stdout` / `.strace`、`final-real-kernel.strace`、`final-EPERM.*` / `final-EACCES.*`、`final-proc-ENOENT.stdout` / `final-proc-EACCES.stdout`、`release-final-libc.log`、`service-smoke-daemon.log`、`service-smoke-version.json`、`package-abi-final.txt`、`package-main-apt-install.log`、`package-main-review.json`。故障注入辅助程序和构建脚本均留在任务临时目录，未混入源码提交或运行包。
