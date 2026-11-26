# Clash Verge Service IPC 2.7.7-2loong2

日期为 2026 年 11 月 26 日。上游是 [clash-verge-rev/clash-verge-service-ipc](https://github.com/clash-verge-rev/clash-verge-service-ipc)，适配仓库是 [ChinaOSProject/clash-verge-service-ipc](https://github.com/ChinaOSProject/clash-verge-service-ipc)。本轮源码基于 `loong64` 和 `debian10` 共同的提交 `eaaae99bb24c77155dbea95b3964f6d5be42b0f1`。

## Linux 路径策略

麒麟设备的 `/`、`/usr` 和 `/usr/bin` 都是 `root:root 0775`。本地观察显示 `/var`、`/var/lib` 和 `/run` 是 `root:root 0755`。旧服务写入两个经摘要核验的 core 后，运行时全链校验在 `/` 拒绝这两个 core，安装器因而无法通过 IPC 安装验收。

ChinaOS 构建显式启用 `chinaos-kylin-root-group`。校验继续逐层检查整条路径。仅允许 `/` 和 `/usr` 这两个目录采用 UID 0、GID 0、精确 `0775`。只有 `lgetxattr` 查询 `system.posix_acl_access` 返回 `ENODATA` 时才接受例外。存在访问 ACL 或读取结果不明确时沿用拒绝分支。本机 `/usr/bin` 已将系统程序的目录写入权限授予 GID 0，因此此定制包以 GID 0 属于受信任的系统主体为前提。默认 Cargo feature 集继续使用上游权限策略。补丁归档于 [patches/clash-verge-service-ipc_v2.7.7-2loong2_kylin-root-group.patch](patches/clash-verge-service-ipc_v2.7.7-2loong2_kylin-root-group.patch)。

自动 staging 仍检查源文件及其祖先。该设备的 core 链接指向 `/usr/bin`，因此通过现有 `--prepare-install` 摘要流程安装两个 core。服务执行和 IPC 验收仍使用 `/var/lib/clash-verge-service/cores/` 中的批准副本，并要求其文件和其他目录组件通过原有权限检查。持久状态、systemd 登记和 IPC 路径沿用当前应用集成。

## 构建和交付

复用缓存的 Debian 10 LoongArch 工具链、锁定的 Git 对象和 148 份 crate 归档，使用 `cargo build --locked --release --features standalone,client,chinaos-kylin-root-group` 构建三个服务程序。缺少的编译输入从 crates.io 按原锁文件取得。

Debian 包沿用 `clash-verge-service 2.7.7-1` 的拆分和运行依赖方式。上轮报告引用的 Debian Buster `transmission 2.94-2+deb10u2` 将独立 daemon 与 GUI 分开。本包将相邻部署所需的 daemon、installer、uninstaller 和许可证放入一个运行包，运行库依赖由 `dpkg-shlibdeps` 提取，`systemd` 由安装器与卸载器的运行需求声明。

候选包为 `clash-verge-service_v2.7.7-2loong2_loongarch64.deb`，大小 1,562,668 字节，SHA-256 为 `aff59e8774b4e62c8f72ac118d5c8519b87db0f2a0bd5853c58589595ea03719`。运行依赖为 `libc6 (>= 2.28), libgcc1 (>= 1:4.2), systemd`。三个 ELF 均为 LoongArch ELF64 LP64，解释器为 `/lib64/ld.so.1`，最高 glibc 符号版本均为 `GLIBC_2.28`。旧基座的动态加载器解析了全部依赖。兼容检查使用的 `lgetxattr` 在 daemon 和 installer 中要求 `GLIBC_2.27`。

## 回归结果

- 旧包 `2.7.7-1` 在根目录为 `0775` 时仍拒绝两个批准副本，记录在 `logs/regression-baseline.log`。
- 候选包通过 `0755` 与麒麟模式的安装和 IPC 验收，两个正式 Mihomo 包的批准副本均与输入逐字节一致。受保护的 `/usr/auto` 自动 staging 通过，`0775` 的 `/usr/bin` 自动 staging 和错误摘要均被拒绝。
- `/` 和 `/usr` 各自为 `0777` 或带扩展访问 ACL 时均被相关调用拒绝。`0775`、非零 GID 的 `/usr`、组可写的 `/var/lib` 和组可写的已批准 core 均被拒绝。
- 主验收在麒麟 5.4.18 内核的私有挂载和网络命名空间中使用 `.deb` 内的服务程序。`0755` 和 `0775` 两档均成功安装，实际启动的 Mihomo 建立 `cvtest0` 和 IPv4 路由，停止后清除网卡。终止并重新启动守护进程后，两档均重新通过 core 验收及 TUN 启停。该复验同时确认根目录的非零 GID 和命名用户访问 ACL 均遭拒绝。
- 完整回归在 `logs/regression-summary.log`，构建在 `logs/build-release.log`，ABI 证据在 `logs/abi.log`。

服务编译缓存位于 `/mnt/chinaos-cache/ChinaOSProject-cache/service-ipc/v2.7.7-2loong2/`。TUN 日志见 [GUI 适配报告](clash-verge_v2.5.10-1loong3_loongarch64.md)。隔离测试用夹具执行安装器生成的 unit 中的 `ExecStart`，宿主机 systemd 与桌面授权待实测。
