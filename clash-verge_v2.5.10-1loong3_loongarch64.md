# Clash Verge Rev 2.5.10-1loong3

2026 年 11 月 26 日，在银河麒麟 V10 SP1 LoongArch 上修复 `root:root 0775` 导致的安装服务和 TUN 失败。旧版应用已能调用服务安装器。安装器后的 IPC 验收在检查 `/` 的组写权限时拒绝已安装的 Mihomo 副本，导致安装动作无法完成。源码修复及权限例外见 [服务适配报告](lib_clash-verge-service-ipc_v2.7.7-2loong2_loongarch64.md)。

## Debian 交付

`clash-verge_v2.5.10-1loong3_loongarch64.deb` 复用 `2.5.10-1loong2` 的 GUI 及资源，更新包版本和 `clash-verge-service (= 2.7.7-2loong2)` 的精确依赖。解包比对确认两版的文件内容、权限、属主及链接一致。沿用此前的 GUI、稳定版和 alpha 版 Mihomo、数据及服务分包。分包所用的 Debian Transmission 对照和 GUI 的 LoongArch ABI、glibc 2.28 检查见 [2.5.10-1loong2 适配报告](clash-verge_v2.5.10-1loong2_loongarch64.md)。

| 包 | SHA-256 |
| --- | --- |
| `clash-verge_v2.5.10-1loong3_loongarch64.deb` | `3f9fa5921388786c0146fae622959039052579338ca11d62c4f315239d4c8da6` |
| `clash-verge-service_v2.7.7-2loong2_loongarch64.deb` | `aff59e8774b4e62c8f72ac118d5c8519b87db0f2a0bd5853c58589595ea03719` |

## 验收

- 宿主 APT 模拟成功解析配套包，只计划升级 GUI 和服务两包。
- 在宿主 Linux 5.4.18 内核的私有命名空间中，旧包在 `0775` 根目录复现安装失败。新服务在 `0755` 和 `/`、`/usr` 同为 `root:root 0775` 时，通过同一 `--prepare-install` 入口安装和验证正式的稳定版与 alpha 版 Mihomo。
- 用服务 IPC 启动稳定版 Mihomo 后，查询到真实 `cvtest0` TUN 网卡，路由表中出现该网卡的 IPv4 默认路由和本地路由。停止后网卡消失。重启守护进程再次通过两份 core 的验收及 TUN 启停。
- 复验拒绝了 `0777`、非零 GID、扩展访问 ACL、组可写的 `/var/lib` 和已批准 core。`/usr` 下的受保护源文件通过自动检查，该目录增加扩展访问 ACL 后被拒绝。
- 日志位于 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/tun-20261126.MLfWVn/`，对应 `extended-old-0775.log`、`candidate-root-0755.log`、`candidate-root-0775.log`、其他 `candidate-*.log` 及 `apt-simulation.log`。

TUN 验收使用真实网卡和路由，systemd 调用由测试夹具模拟。宿主目录权限保持 `0775`。宿主的 systemd 注册和桌面提权点击待实测。
