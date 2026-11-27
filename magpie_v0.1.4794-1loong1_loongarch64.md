# Magpie Daily v0.1.4794 LoongArch 适配

[Magpie Daily 更新接口](https://usemagpie.ai/api/latest?channel=daily)于 2026 年 11 月 27 日返回 `v0.1.4794`。该版本于 2026 年 11 月 25 日 20:07（北京时间）发布，对应[上游源码提交 `f7cb8aa0`](https://github.com/yetone/magpie/tree/f7cb8aa0baace17612097c0d72b58355289bda0f)。本地 `debian10` 从上游标签快进，`loong64` 再从 `debian10` 快进。两条分支都指向该提交。

## 构建与运行依赖

从标签导出源码并原生构建。复用 Go 1.27.1 oldworld SDK、GCC 8.3.0、GTK 3.24.5 和 WebKitGTK 2.40.5。构建启用 `CGO_ENABLED=1` 和 `production,gtk3` 标签，通过 `-X main.version=v0.1.4794` 写入版本。上游 `go.mod` 和 `go.sum` 与上一版一致。临时 modfile 继续引用 [Wails v3.0.0-beta.24 的 GTK3 修复](lib_wails_v3.0.0-beta.24_loongarch64.md)。

沿用[上一版打包方式](magpie_v0.1.4762-1loong1_loongarch64.md)。Debian [Hugo](https://packages.debian.org/trixie/hugo) 的打包方式提供 Go 内嵌资源布局参考，[Liferea](https://packages.debian.org/trixie/liferea) 提供 GTK3 和 WebKitGTK 运行依赖参考。Magpie 的桌面、CLI 和嵌入式 Web 界面共用一个 ELF。现有 `clash-*` 运行库包提供 GLib、libsoup、JavaScriptCoreGTK 和 WebKitGTK。

## 产物与验证

| 字段 | 值 |
| --- | --- |
| 包名 / 版本 / 架构 | `magpie` / `0.1.4794-1loong1` / `loongarch64` |
| 文件 | `binaries/magpie/loongarch64/magpie_v0.1.4794-1loong1_loongarch64.deb` |
| 包大小 | 32,471,916 字节 |
| 包 SHA-256 | `a42047f4efc1cf85399e94ef71d573557ecdf51dad7001be3dc24703ce352472` |
| ELF SHA-256 | `fa07e6d331219daf20a1531ddcab794cf8f0f601ba498cf6de1186e054138015` |
| ELF ABI | LoongArch LP64，flags `0x3`，解释器 `/lib64/ld.so.1`，最高直接符号版本 `GLIBC_2.27` |
| ELF RUNPATH | `/usr/lib/clash-verge/runtime/lib` |

- 清除 `LD_LIBRARY_PATH` 后，所有动态库均可解析。直接依赖和 RUNPATH 与上一版一致。
- Debian 包 SHA-256、八个包内文件的 MD5 和解包后 ELF 字节比对通过。APT 模拟仅将 `magpie` 从 `0.1.4762-1loong1` 升级到 `0.1.4794-1loong1`。
- 使用隔离 XDG 配置启动，`--version` 返回 `magpie v0.1.4794`。GTK3 桌面在 Xvfb 中完整渲染首次启动页，本地网关通过 `healthcheck`，应用正常退出。
- 独立网络命名空间与临时用户目录中，18 项 Daily 渠道、网关账号、CLI、Library 和 TUI 回归测试通过。

构建脚本与验证日志保存在 `/mnt/chinaos-cache/ChinaOSProject-cache/magpie/v0.1.4794/`。

## 发布

`debian10` 和 `loong64` 已推送至 ChinaOSProject Magpie fork，远端提交均为 `f7cb8aa0baace17612097c0d72b58355289bda0f`。Debian 包和校验文件已发布到 [ChinaOSProject/binaries 提交 `fbeab74`](https://github.com/ChinaOSProject/binaries/commit/fbeab74ae5955464326424830cb54592b705c78e)。
