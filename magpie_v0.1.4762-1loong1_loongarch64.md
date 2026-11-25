# Magpie 0.1.4762-1loong1 LoongArch 适配

上游仓库为 <https://github.com/yetone/magpie>。源码来自 `v0.1.4762`，提交 `161baac4cdb769792b247feaa92974f5a5a2594a`。上游于 2026 年 11 月 25 日 01:18（北京时间）发布此版本。`debian10` 从目标标签快进，`loong64` 随后从 `debian10` 快进。

## 构建与依赖

通过 `git archive` 导出精确源码，在 LoongArch 宿主机原生构建。复用 Go 1.27.1 oldworld SDK、GCC 8.3.0、GTK 3.24.5、私有 WebKitGTK 2.40.5 SDK 和上一版 Go 缓存。构建使用 `CGO_ENABLED=1`、`-tags production,gtk3`、`-trimpath`、`-buildvcs=false`，通过 `-X main.version=v0.1.4762` 写入版本。应用源码、`go.mod` 与 `go.sum` 均为上游版本，后两项与 `v0.1.4697` 一致。临时 modfile 继续引用独立归档的 Wails `v3.0.0-beta.24` [GTK3 空窗口修复](lib_wails_v3.0.0-beta.24_loongarch64.md)，提交 `f190670b6a3a5fd5ac8a77d881dcdce22f051218`。

包名、运行依赖和分包依据沿用[上一版适配](magpie_v0.1.4697-1loong1_loongarch64.md)。参考 Debian [Hugo](https://packages.debian.org/trixie/hugo) 的 Go 内嵌资源布局，以及 [Liferea](https://packages.debian.org/trixie/liferea) 的 GTK3/WebKitGTK 运行依赖。Magpie 的桌面、CLI 和 Web 资源共用同一 ELF，目标系统由现有 `clash-*` 私有包提供 GLib、libsoup、JavaScriptCoreGTK 和 WebKitGTK。复用已分包的工具链和运行库，本轮交付的软件包为 Magpie。

## 产物与验证

| 字段 | 值 |
| --- | --- |
| 包名 / 版本 / 架构 | `magpie` / `0.1.4762-1loong1` / `loongarch64` |
| 文件 | `binaries/magpie/loongarch64/magpie_v0.1.4762-1loong1_loongarch64.deb` |
| 包大小 | 32,398,344 字节 |
| 包 SHA-256 | `40a4eed7e359d9af24d8e31d6531c3dcd04898326b11b993822897d627003f25` |
| ELF SHA-256 | `567c66b29c96e9ad506d3cb82d6baa39736b19225f87e9ee74fbd9123de8b001` |
| ELF ABI | LoongArch LP64，flags `0x3`，解释器 `/lib64/ld.so.1`，最高直接符号版本 `GLIBC_2.27` |
| ELF RUNPATH | `/usr/lib/clash-verge/runtime/lib` |

- 外盘及 NVMe 临时目录的两个链接结果逐字节一致。清除 `LD_LIBRARY_PATH` 后所有动态库正常解析，DT_NEEDED 和 RUNPATH 与上一版相同。
- 解包后的程序在 NVMe 上使用隔离 XDG 配置，版本命令返回 `magpie v0.1.4762`，Xvfb 中完整渲染首次启动页，独立回环网关通过 `healthcheck`，测试进程正常退出。此前外盘配置下的一次重复测试发生目录请求和回环探测超时，日志留在 `smoke-full/`。
- 独立网络命名空间中，五项 CLI 回归、三项 MCP 空闲设置回归及一项 TUI Library 回归通过。新增 Playwright 浏览器测试因宿主缺少 Playwright 环境未运行。
- Debian 包 SHA-256、八个包内文件的 MD5 及解包后 ELF 字节比对通过。APT 模拟仅升级 Magpie。验证时本机安装版本为 `0.1.4697-1loong1`。

构建脚本、发行元数据、验证日志和原生窗口截图位于 `/mnt/chinaos-cache/ChinaOSProject-cache/magpie/v0.1.4762/`。

## 发布

ChinaOSProject Magpie fork 的 `debian10` 与 `loong64` 均已推送并核对目标提交。Debian 包和 SHA-256 校验文件发布于 [ChinaOSProject/binaries 提交 `6e9a0be`](https://github.com/ChinaOSProject/binaries/commit/6e9a0be593d1b29248d4509660460d62b5a7b662)，重新下载的 Git LFS 对象与本地包逐字节一致。
