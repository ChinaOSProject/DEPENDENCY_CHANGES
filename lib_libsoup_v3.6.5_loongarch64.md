# libsoup 3.6.5：Clash 私有 Soup 3 运行库与 SDK

日期：2026-11-12。上游 GitHub 仓库：`https://github.com/GNOME/libsoup`；主要上游：`https://gitlab.gnome.org/GNOME/libsoup`。实际 libsoup 3.6.5，完整包版本 `3.6.5-1loong1`。私有 SONAME `libsoup-3.0.so.0` 位于 `/usr/lib/clash-verge/runtime/lib`，供 WebKitGTK 4.1 使用，不替换系统 Soup 2。

## Debian 依据与必要差异

实际参考 Debian `libsoup3 3.6.5-3`：`https://snapshot.debian.org/package/libsoup3/3.6.5-3/`；准确 `.dsc`、debian tar、copyright 已保存在 `references/`。control/install 成员和散列见 `archive-review-20261112/debian-reference-extraction.json`。上游运行库与开发包对应 Debian `libsoup-3.0-0`、`libsoup-3.0-dev`，本次加 `clash-` 前缀并装。Debian 的 `libsoup-3.0-common` 对应本组实际翻译内容的 `clash-libsoup-3.0-data`；名称差异明确保留，不声称逐名复刻 Debian。该数据包为 Architecture:all。没有构建 introspection、vapi、文档或安装测试，故不交付虚假的 gir/doc/tests 包。

运行库精确依赖 GLib 2.70.5 私有版本，TLS 后端使用发行版 `glib-networking`；其余真实链接库和头文件依赖按 control 分开。数据包没有强制 Depends，显式安装整组可获得翻译。

| 包 | 实际 control Depends |
|---|---|
| `clash-libsoup-3.0-0` | `clash-libglib2.0-0 (= 2.70.5-1loong1), libc6 (>= 2.28), libsqlite3-0, libpsl5, libbrotli1, zlib1g, libnghttp2-14, glib-networking` |
| `clash-libsoup-3.0-data` | `（无 Depends）` |
| `clash-libsoup-3.0-dev` | `clash-libsoup-3.0-0 (= 3.6.5-1loong1), clash-libglib2.0-dev (= 2.70.5-1loong1), libsqlite3-dev, libpsl-dev, libbrotli-dev, zlib1g-dev, libnghttp2-dev, pkg-config` |

## 已有构建、ABI 与验证

`/tmp/clash-verge-rev-v2.5.8-20261101/build-libsoup365.sh` 和 `logs/build-libsoup365.log` 记录 GCC 8.3.0、Meson 0.63.3、release/nofallback、私有 prefix，关闭 tests/installed_tests/introspection/vapi/docs/sysprof。`tls_check=false` 仅跳过 Meson 配置期的 TLS 模块可用性探测；运行库仍依赖 glib-networking，不代表关闭请求的证书验证。此轮不修改或重新构建 libsoup 源码。

已有 `logs/family-v2-pkgconfig-versions.log` 记录 3.6.5，`logs/soup-tls-rpath-v2510-02.log` 记录本地受信任证书请求 `HTTP status=200 body=16`。该日志同时保留旧 glib-networking 模块的 `advertised-protocols` 警告；未把这个有限 TLS smoke 扩大为 HTTP2/ALPN、完整网络协议或完整上游测试通过。SONAME 文件与开发内容、完整版权文本、安装与 DT_NEEDED 闭包由共享审计记录覆盖。

## 最终产物

本来源的既有 ABI 审计覆盖 1 个 ELF，最高导入版本为 GLIBC 2.27；这是文件实际导入版本，control 可能为统一基座要求保留更保守的 libc6 >=2.28。按来源归类结果由原始逐文件审计派生，记录在 `archive-review-20261112/family-abi-summary-from-accepted-evidence.json`。

| 包 | 完整版本 | 架构 | 字节 | SHA256 |
|---|---|---|---:|---|
| [clash-libsoup-3.0-0](../binaries/clash-libsoup-3.0-0/loongarch64/clash-libsoup-3.0-0_v3.6.5-1loong1_loongarch64.deb) | 3.6.5-1loong1 | loongarch64 | 238392 | `b5593bbe595fa8e6a25a82dd8eae73a5cf76ff6291cc5f478cb54d078aa1d9aa` |
| [clash-libsoup-3.0-data](../binaries/clash-libsoup-3.0-data/all/clash-libsoup-3.0-data_v3.6.5-1loong1_all.deb) | 3.6.5-1loong1 | all | 71274 | `40886e709d2f19a14428b51a9ee5ba74b27090bdaf30fa0a18e269796e1ea044` |
| [clash-libsoup-3.0-dev](../binaries/clash-libsoup-3.0-dev/loongarch64/clash-libsoup-3.0-dev_v3.6.5-1loong1_loongarch64.deb) | 3.6.5-1loong1 | loongarch64 | 34944 | `96e27996c40826d9017ab0350cc6353e6b0d97ea36ffbcc4d0102946f59156be` |

## 共享归档与验证依据

完整交付索引为 [clash-native-family_v2.5.10_artifacts.json](clash-native-family_v2.5.10_artifacts.json)，可读索引为 [同名 Markdown](clash-native-family_v2.5.10_artifacts.md)。每个 `.deb` 使用完整 Debian Version 命名，旁附 `.deb.sha256`；Architecture:all 的数据/文档包放在 `all/`。目标旧世界发行版的实际 dpkg 架构名为 `loongarch64`，不同于当前 Debian sid 的 `loong64`。

主证据目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/native-family`。本轮沿用已通过检查并由主 Agent 归档的 36 个 v4 包，没有重建、重拷或重跑整组/GUI 测试；仅 GCC 三个工具由独立 v7 候选替换。既有 `native-family-final-manifest-v2510.json` 对应历史 v2，不应作为最终哈希表。最终选择以本共享索引、`archive-review-20261112/archived-artifacts.json` 和 GCC v7 归档记录为准。

`logs/native-family-v4-archive-and-payload-audit.json` 检查了全部 39 个历史 v4 归档、文件 md5 与版权；`logs/native-family-v4-postinstall-integrity.json` 记录专用 Buster 容器实际安装、39 包版本匹配、2742 个普通文件 md5、`dpkg --verify` 和 `apt-get check` 通过。`logs/dt-needed-package-closure-v2510-v4-01.json` 对 133 个 ELF、723 条 DT_NEEDED 依赖边及实际包归属检查为 PASS：使用记录明确指定的私有运行库环境，不能解释成任意裸启动环境均自动解析私有库。

既有 `logs/family-v2-abi-audit.json` 的 133 个原生 ELF 均为 LoongArch、flags `0x3`；50 个程序使用 `/lib64/ld.so.1`，其余 83 个没有 PT_INTERP；整组最高导入 GLIBC 为 2.28、GLIBCXX 为 3.4.32。v4 的内容变化/不变范围另见其 manifest。所有包使用私有 `clash-*` 名称，无用于取代系统包的 `Provides/Conflicts/Breaks/Replaces`，没有覆盖宿主公共库、修改全局 ld.so 配置或全局环境。私有 SDK 的消费者应显式设置 `PKG_CONFIG_PATH=/usr/lib/clash-verge/runtime/lib/pkgconfig` 并选择相应加载路径。

本轮只在现有容器组装 GCC 包和进行有针对性的安装验证，所有实际编译/链接/运行均受宿主共享 flock 保护。未运行各上游完整测试集，未新增 GUI 验收，也未启动/停止/执行 Magpie。应用最终 GUI 验收与推送发布由主 Agent 负责；本执行 Agent 未 commit 或 push。
