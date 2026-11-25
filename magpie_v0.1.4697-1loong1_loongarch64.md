# Magpie 0.1.4697-1loong1 LoongArch 适配

上游仓库为 <https://github.com/yetone/magpie>。本包使用 2026 年 11 月 23 日发布的 `v0.1.4697`，提交 `5707c143b965fc2ae56e18980cd776fca574519b`。GitHub 发行记录与 Git 标签一致，构建目录的 10,847 个文件和符号链接对照 Git 归档验证通过。

## 构建环境

在 LoongArch 宿主机上使用已有的 Go 1.27.1 oldworld SDK、GCC 8.3.0、GTK 3.24.5 和私有 WebKitGTK 2.40.5 SDK。所有构建缓存放在 `/mnt/chinaos-cache/ChinaOSProject-cache/magpie/v0.1.4697/`。构建参数为 `CGO_ENABLED=1`、`-tags production,gtk3`、`-trimpath`、`-buildvcs=false`，通过 `-X main.version=v0.1.4697` 写入版本。Go 镜像下载的模块按上游 `go.sum` 校验。

`debian10` 从上游目标标签快进，`loong64` 随后从 `debian10` 快进。Magpie 源码和锁文件均与上游标签一致。临时 Go modfile 引用已归档的 Wails `v3.0.0-beta.24` GTK3 空窗口修复，依赖快照中的 4,051 个文件和符号链接与独立适配提交 `f190670b6a3a5fd5ac8a77d881dcdce22f051218` 一致。Wails 源码仓库为 <https://github.com/wailsapp/wails>，修复按[独立适配报告](lib_wails_v3.0.0-beta.24_loongarch64.md)归档。

## Debian 包

| 字段 | 值 |
| --- | --- |
| 包名 / 版本 / 架构 | `magpie` / `0.1.4697-1loong1` / `loongarch64` |
| 文件 | `binaries/magpie/loongarch64/magpie_v0.1.4697-1loong1_loongarch64.deb` |
| 文件大小 | 32,084,040 字节 |
| 包 SHA-256 | `988673f3678760575c08a706d9b6efdae08566437879965ee296e2e5301b7988` |
| ELF SHA-256 | `9bc984095fd4ad6be1b90aab418fda7583f2efa8a1d51e920afc9968a9ee7530` |
| ELF ABI | LoongArch LP64，flags `0x3`，解释器 `/lib64/ld.so.1`，最高直接符号版本 `GLIBC_2.27` |
| ELF RUNPATH | `/usr/lib/clash-verge/runtime/lib` |

分包沿用上一版的 Debian 参考，包括 [Hugo](https://packages.debian.org/trixie/hugo) 的 Go 内嵌资源布局和 [Liferea](https://packages.debian.org/trixie/liferea) 的 GTK3/WebKitGTK 运行依赖。Magpie 的 GUI、CLI 和 Web 资源都在同一 ELF 中，包内另含桌面入口、图标及许可证。目标基座使用既有 `clash-*` 私有 GLib、libsoup、JavaScriptCoreGTK 和 WebKitGTK 分包，以及系统 GTK3 和相关 X11/Wayland 库。新旧 ELF 的直接依赖列表和 RUNPATH 一致。Go SDK 和桌面 SDK 均复用已分包产物，本轮生成的独立交付软件为 Magpie。

## 验证

- 清除 `LD_LIBRARY_PATH` 后所有动态库均能解析，直接 GLIBC 要求低于目标机的 2.28。
- `--version` 返回 `magpie v0.1.4697`。隔离 XDG 配置和 Xvfb 下，GTK3 窗口显示了 WebKit 渲染的首次启动页面，回环网关通过 `healthcheck`，测试进程正常退出。
- 五项 CLI 回归测试通过，涵盖显式和隐式网关地址、网关启动等待及配置一致性。宿主的旧 Magpie 占用默认端口，测试在启用回环设备的独立网络命名空间中复跑。
- 包 SHA-256、八项包内 MD5 和解包后 ELF 字节比对通过。APT 模拟仅升级 Magpie，新增和移除的软件包均为零。
- 宿主已安装版本为 `0.1.3078-1loong2`。自动审批拒绝以管理员权限替换系统包。现有 `serve` 和 `tui` 进程继续运行。

构建与验证日志、截图及可复现脚本保存在上述外盘缓存目录。

## 发布

ChinaOSProject Magpie fork 的 `debian10` 和 `loong64` 远端均指向 `5707c143b965fc2ae56e18980cd776fca574519b`。Debian 包和 SHA-256 校验文件发布于 [ChinaOSProject/binaries 提交 `6d7128d`](https://github.com/ChinaOSProject/binaries/commit/6d7128deff6211819d64c1b2d499fb8ad287a892)。
