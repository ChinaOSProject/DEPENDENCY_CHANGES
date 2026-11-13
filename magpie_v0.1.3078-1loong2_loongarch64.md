# Magpie 0.1.3078-1loong2：本机安装修订记录

上游仓库：https://github.com/yetone/magpie

本报告对应完整 Debian 版本 `0.1.3078-1loong2`。上游版本为 stable `v0.1.3078`，精确提交 `ef176e21f6c40e5a623709bb0e369647173964de`。2026-11-12 已保存的官方 stable API 响应为 HTTP 200、`channel=stable`、`version=0.1.3078`；原始正文 SHA256 为 `1517acfd0976b330feb58a78f144099266a7269c554fbdd040f1250acd5cfdbc`。

2026-11-13 源码归档：已将对应精确构建基线的 `debian10`、`loong64` 分支推送至 https://github.com/ChinaOSProject/magpie ，并核对两分支远端提交均为 `ef176e21f6c40e5a623709bb0e369647173964de`。应用源码未修改；Wails 依赖的窗口状态修复仍以独立补丁归档，其源码提交审查要求未满足，不包含在这两个应用分支中。fork 的默认分支未改动。

完整的构建、初版历史验证、Debian 分包依据及修复经过保留在 [上游版本汇总报告](magpie_v0.1.3078_loongarch64.md)。本记录只确认当前修订的交付身份和验证边界，不将初版 GUI 测试视为修复版 GUI 测试。

| 项目 | 已核验内容 |
| --- | --- |
| Debian 包 | `binaries/magpie/loongarch64/magpie_v0.1.3078-1loong2_loongarch64.deb` |
| Package / Version / Architecture | `magpie` / `0.1.3078-1loong2` / `loongarch64` |
| 大小 | 22,418,572 字节 |
| 包 SHA256 | `a4ce6cd81c6f7734b8af515da2d4581683fd307b7401808716e88cda49eb7691` |
| ELF SHA256 | `06f6ab247f140fc99f4ef1a831330048cc5ea0d1dafad5780c7a47e2f7042e79` |
| ELF ABI | LoongArch flags `0x3`，解释器 `/lib64/ld.so.1`，最高直接 GLIBC 符号需求 `2.27` |
| ELF RUNPATH | `/usr/lib/clash-verge/runtime/lib` |
| 包校验 | 8 项包内 MD5 通过，包内 ELF 与构建候选一致，无 maintainer scripts |
| 本机安装 | `magpie 0.1.3078-1loong2 loongarch64 install ok installed` |

快捷方式修复将用户应用入口及桌面入口的 `Exec` 改为 `/usr/bin/magpie %u`，保留原权限并通过桌面文件检查。修订 ELF 的 RUNPATH 也使上游自注册产生的原始 ELF 路径能够找到私有运行库；移除 `LD_LIBRARY_PATH` 后，静态依赖检查全部解析成功。没有修改系统全局加载配置或公共 GLib。

GDK 修复位于独立 Wails `v3.0.0-beta.24` 依赖：三个 GTK3 状态查询分别保护空 widget 和空 GDK window。原版回归在 GTK critical 处失败，补丁后新回归及既有状态事件测试通过。生产修改、补丁及测试限制见 [Wails 依赖报告](lib_wails_v3.0.0-beta.24_loongarch64.md)。Magpie 应用源码及锁未修改；构建使用仓库外依赖快照与独立 modfile，未修改 Go 模块缓存源码。

所有编译、打包及回归均在既有 Podman 项目容器中完成。复用已分包 Go SDK 和私有 GTK/WebKit 运行库家族，未新建容器、宿主编译或安装宿主构建依赖。GUI、CLI 与嵌入 Web 同属一个应用二进制，没有独立 SONAME、SDK 或可拆出的 Web 数据目录，不制造空的开发包或数据包。

用户明确禁止修复期间启动、停止、重启或向 Magpie 发信号。修复版安装及后续核验均遵守这一约束，包括未运行 `--version`、应用入口、菜单、桌面快捷方式或 `gtk-launch`。安装前后的两个既有进程启动时间和可执行映像身份保持一致，它们继续使用旧映像。**修复版 GUI 尚未启动验证，不能据此宣称当前活动会话中的警告已经消失。** 未实现的真实 GTK 窗口、销毁过程及窗口管理器状态转换也未完成集成测试。

证据根目录：

- 构建、回归、包及安装：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/magpie-stable-v0.1.3078/repair-20261111/`。
- 快捷方式与 stable 响应：`/mnt/chinaos-cache/ChinaOSProject-cache/local-install-20261110/` 下的 `magpie-launch-repair-20261111/` 和 `logs/magpie-stable-feed-20261112.{headers,body}`。

本记录补齐完整修订号的报告路径，不改变现有包、源码、进程或历史测试结果。源码补丁及本轮交付的远端发布仍待统一审核。
