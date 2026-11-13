# GLib 2.70.5：最终 Clash 私有运行库、SDK 与工具分包

日期：2026-11-12。上游 GitHub 仓库：`https://github.com/GNOME/glib`；主要上游：`https://gitlab.gnome.org/GNOME/glib`。实际 GLib 版本为 2.70.5，本组完整包版本均为 `2.70.5-1loong1`，用于 libsoup 3.6.5/WebKitGTK 4.1 的旧世界 LoongArch 私有依赖栈。

## 撤回原单包候选

旧报告描述的 `libglib2.0-bin` 单包（Version `2.70.5-1`，SHA256 `1f2d581c5155a3ca9e93d07c37abc6d7a87f0dfc4d6faa4fbe2b010a81da6b37`）混合了 runtime、headers、SDK 与工具，且使用公共包名。该候选已撤回、不可发布，不能当作系统 `libglib2.0-bin` 的升级。原报告逐字保存在 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/native-family/archive-review-20261112/lib_glib_v2.70.5_loongarch64.withdrawn-monolith.md`；保全散列见 `glib-obsolete-report-preservation.json`。历史单包及原测试记录不被删除，也不作为最终交付。宿主公共 GLib 未被替换。

## Debian 依据及实际文件归属

参考 Debian 源包 `glib2.0 2.70.5-1`：`https://snapshot.debian.org/package/glib2.0/2.70.5-1/`。保存的 `.dsc`、`.debian.tar.xz` 位于 `references/source-package-inputs/`；本轮抽取真实 `debian/control` 及 `.install` 到 `archive-review-20261112/primary-reference-controls/`，含归档与成员 SHA256 的记录为 `debian-reference-extraction.json`。采用其 `libglib2.0-0 / -bin / -dev / -dev-bin / -data` 分工，并加 `clash-` 前缀实现并装：

- `clash-libglib2.0-0`：GLib/GObject/GIO/GModule/GThread SONAME 库，及 Debian 运行库惯例中的 `gio-querymodules`、`glib-compile-schemas`。
- `clash-libglib2.0-bin`：gio、gdbus、gsettings、gresource、gapplication 等运行工具。
- `clash-libglib2.0-dev-bin`：resource 编译器、gdbus-codegen、GLib 代码生成/开发工具。
- `clash-libglib2.0-dev`：头文件、glibconfig.h、pkg-config、开发链接文件及实际已安装的 SDK 内容。
- `clash-libglib2.0-data`：实际生成的翻译/公共数据，Architecture:all。

统一私有前缀 `/usr/lib/clash-verge/runtime`，不是旧报告的 `/usr/lib/glib-2.70.5`，不会提供公共 `/usr/bin/gio` 替换器。没有生成的完整上游测试、文档或 introspection 内容不伪造空包。版本锁定的私有依赖和真实系统 runtime/dev 依赖如下。

| 包 | 实际 control Depends |
|---|---|
| `clash-libglib2.0-0` | `libc6 (>= 2.28), libpcre3, libffi7, libmount1, libselinux1, zlib1g` |
| `clash-libglib2.0-bin` | `clash-libglib2.0-0 (= 2.70.5-1loong1)` |
| `clash-libglib2.0-data` | `（无 Depends）` |
| `clash-libglib2.0-dev` | `clash-libglib2.0-0 (= 2.70.5-1loong1), clash-libglib2.0-bin (= 2.70.5-1loong1), clash-libglib2.0-dev-bin (= 2.70.5-1loong1), libffi-dev, libpcre3-dev, libmount-dev, libselinux1-dev, pkg-config, zlib1g-dev` |
| `clash-libglib2.0-dev-bin` | `clash-libglib2.0-0 (= 2.70.5-1loong1), python3` |

## 构建与验证范围

已有构建日志 `/tmp/clash-verge-rev-v2.5.8-20261101/logs/build-glib270.log` 记录 Meson 0.63.3、GCC 8.3.0、实际 native GLib 2.70.5；此轮不修改/重编 GLib 源码。最终包的版权来自保存的准确 Debian 2.70.5-1 文本和上游 COPYING，完整 licence 文本已随包归档。

已有 `logs/family-v2-pkgconfig-versions.log` 记录 GLib 2.70.5。`logs/gio-tls-rpath-v2510-02.log` 记录私有 GLib 的受信任本地 TLS 请求返回 HTTP 200；系统 glib-networking/dconf 模块的复用依赖实际加载环境。libsoup TLS 记录仍包含旧模块对 `advertised-protocols` 的 GLib-GIO 警告，不能据此宣称 ALPN/HTTP2 或所有旧系统模块兼容性已完整验证。完整依赖边检查使用共享部分注明的加载环境。

## 最终产物

本来源的既有 ABI 审计覆盖 15 个 ELF，最高导入版本为 GLIBC 2.28；这是文件实际导入版本，control 可能为统一基座要求保留更保守的 libc6 >=2.28。按来源归类结果由原始逐文件审计派生，记录在 `archive-review-20261112/family-abi-summary-from-accepted-evidence.json`。

| 包 | 完整版本 | 架构 | 字节 | SHA256 |
|---|---|---|---:|---|
| [clash-libglib2.0-0](../binaries/clash-libglib2.0-0/loongarch64/clash-libglib2.0-0_v2.70.5-1loong1_loongarch64.deb) | 2.70.5-1loong1 | loongarch64 | 1229900 | `87cacfc859a341c2eae2f5e4696286a2478d874a8707f05b69ba2241249be097` |
| [clash-libglib2.0-bin](../binaries/clash-libglib2.0-bin/loongarch64/clash-libglib2.0-bin_v2.70.5-1loong1_loongarch64.deb) | 2.70.5-1loong1 | loongarch64 | 86020 | `2fd7cb088ca6970658e4e7b3a00da5e7a073df9f4287891a5f4b6e13a282ffca` |
| [clash-libglib2.0-data](../binaries/clash-libglib2.0-data/all/clash-libglib2.0-data_v2.70.5-1loong1_all.deb) | 2.70.5-1loong1 | all | 1152938 | `9ae2ff3e7bfd399574492c274a969735b79303d0f979f6a51e2029edee46fbc4` |
| [clash-libglib2.0-dev](../binaries/clash-libglib2.0-dev/loongarch64/clash-libglib2.0-dev_v2.70.5-1loong1_loongarch64.deb) | 2.70.5-1loong1 | loongarch64 | 301454 | `9e46613c6ee3f55eb1e231cd366d62f751304205d7dd005db432f707ae58c2da` |
| [clash-libglib2.0-dev-bin](../binaries/clash-libglib2.0-dev-bin/loongarch64/clash-libglib2.0-dev-bin_v2.70.5-1loong1_loongarch64.deb) | 2.70.5-1loong1 | loongarch64 | 99690 | `bff72577325a404eb2b373fb0a3648d72a9bd271ec1859cb6b8879da6a3d611e` |

## 共享归档与验证依据

完整交付索引为 [clash-native-family_v2.5.10_artifacts.json](clash-native-family_v2.5.10_artifacts.json)，可读索引为 [同名 Markdown](clash-native-family_v2.5.10_artifacts.md)。每个 `.deb` 使用完整 Debian Version 命名，旁附 `.deb.sha256`；Architecture:all 的数据/文档包放在 `all/`。目标旧世界发行版的实际 dpkg 架构名为 `loongarch64`，不同于当前 Debian sid 的 `loong64`。

主证据目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/native-family`。本轮沿用已通过检查并由主 Agent 归档的 36 个 v4 包，没有重建、重拷或重跑整组/GUI 测试；仅 GCC 三个工具由独立 v7 候选替换。既有 `native-family-final-manifest-v2510.json` 对应历史 v2，不应作为最终哈希表。最终选择以本共享索引、`archive-review-20261112/archived-artifacts.json` 和 GCC v7 归档记录为准。

`logs/native-family-v4-archive-and-payload-audit.json` 检查了全部 39 个历史 v4 归档、文件 md5 与版权；`logs/native-family-v4-postinstall-integrity.json` 记录专用 Buster 容器实际安装、39 包版本匹配、2742 个普通文件 md5、`dpkg --verify` 和 `apt-get check` 通过。`logs/dt-needed-package-closure-v2510-v4-01.json` 对 133 个 ELF、723 条 DT_NEEDED 依赖边及实际包归属检查为 PASS：使用记录明确指定的私有运行库环境，不能解释成任意裸启动环境均自动解析私有库。

既有 `logs/family-v2-abi-audit.json` 的 133 个原生 ELF 均为 LoongArch、flags `0x3`；50 个程序使用 `/lib64/ld.so.1`，其余 83 个没有 PT_INTERP；整组最高导入 GLIBC 为 2.28、GLIBCXX 为 3.4.32。v4 的内容变化/不变范围另见其 manifest。所有包使用私有 `clash-*` 名称，无用于取代系统包的 `Provides/Conflicts/Breaks/Replaces`，没有覆盖宿主公共库、修改全局 ld.so 配置或全局环境。私有 SDK 的消费者应显式设置 `PKG_CONFIG_PATH=/usr/lib/clash-verge/runtime/lib/pkgconfig` 并选择相应加载路径。

本轮只在现有容器组装 GCC 包和进行有针对性的安装验证，所有实际编译/链接/运行均受宿主共享 flock 保护。未运行各上游完整测试集，未新增 GUI 验收，也未启动/停止/执行 Magpie。应用最终 GUI 验收与推送发布由主 Agent 负责；本执行 Agent 未 commit 或 push。
