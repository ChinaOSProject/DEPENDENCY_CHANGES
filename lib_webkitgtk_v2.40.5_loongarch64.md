# WebKitGTK / JavaScriptCore 2.40.5：Clash 私有 GTK3 / Soup3 栈

日期：2026-11-12。上游 GitHub 仓库：`https://github.com/WebKit/WebKit`；GTK port：`https://webkitgtk.org/`。实际 release 为 2.40.5，完整包版本 `2.40.5-1loong1`；不是系统 WebKitGTK 4.0 的替代。`webkitgtk-2.40.5.tar.xz` 的已保存下载记录散列为 `7de051a263668621d91a61a5eb1c3771d1a7cec900043d4afef06c326c16037f`，见 `/tmp/clash-verge-rev-v2.5.8-20261101/logs/download-webkit240.log`。

## Debian 来源与实际拆分

参考准确 Debian 源包 `webkit2gtk 2.40.5-1`：`https://snapshot.debian.org/package/webkit2gtk/2.40.5-1/`。其 `.dsc`、`.debian.tar.xz`、copyright 已保存，真实 `debian/control`、各 `.install` 的成员哈希记录在 `archive-review-20261112/debian-reference-extraction.json`。采用 `libwebkit2gtk-4.1-0/-dev` 和 `libjavascriptcoregtk-4.1-0/-dev` 的 ABI 分包，并加 `clash-` 私有前缀。

Debian 该版 JSC 命令行包名是 `libjavascriptcoregtk-4.0-bin`；本组实际构建的是私有 4.1 JSC，故以 `clash-libjavascriptcoregtk-4.1-bin` 交付，不能声称其名称与 Debian 原包完全相同。另将实际生成的翻译分到 Architecture:all 的 `clash-libwebkit2gtk-4.1-data`，MiniBrowser 分到有实际可执行文件的 `clash-libwebkit2gtk-4.1-examples`。WebKit runtime 拥有 WebKitWebProcess、WebKitNetworkProcess、injected bundle 和主库。未生成 gir、WebDriver、完整文档/测试的内容不创建空包。

私有 prefix 为 `/usr/lib/clash-verge/runtime`；JSC/WebKitGTK SONAME 分别为 `libjavascriptcoregtk-4.1.so.0`、`libwebkit2gtk-4.1.so.0`。WebKit 精确依赖同组 JSC、GLib、Soup3、GStreamer core/base/GL 与 GCC13 私有 runtimes；使用发行版 GTK3、ICU67、图形/图片/XML 等现有库。v4 特别补上 WebKit 沙箱链接所需 `libseccomp2`。全部精确 Depends 如下，未将构建工具机械加入 runtime：

| 包 | 实际 control Depends |
|---|---|
| `clash-libjavascriptcoregtk-4.1-0` | `clash-libglib2.0-0 (= 2.70.5-1loong1), clash-libgcc-s1 (= 13.4.0-1loong1), clash-libstdc++6 (= 13.4.0-1loong1), clash-libatomic1 (= 13.4.0-1loong1), libicu67, libsystemd0, zlib1g, libc6 (>= 2.28)` |
| `clash-libjavascriptcoregtk-4.1-bin` | `clash-libjavascriptcoregtk-4.1-0 (= 2.40.5-1loong1)` |
| `clash-libjavascriptcoregtk-4.1-dev` | `clash-libjavascriptcoregtk-4.1-0 (= 2.40.5-1loong1), clash-libglib2.0-dev (= 2.70.5-1loong1), libicu-dev, pkg-config` |
| `clash-libwebkit2gtk-4.1-0` | `clash-libjavascriptcoregtk-4.1-0 (= 2.40.5-1loong1), clash-libglib2.0-0 (= 2.70.5-1loong1), clash-libsoup-3.0-0 (= 3.6.5-1loong1), clash-libgstreamer1.0-0 (= 1.22.0-1loong1), clash-libgstreamer-plugins-base1.0-0 (= 1.22.0-1loong1), clash-libgstreamer-gl1.0-0 (= 1.22.0-1loong1), clash-libgcc-s1 (= 13.4.0-1loong1), clash-libstdc++6 (= 13.4.0-1loong1), clash-libatomic1 (= 13.4.0-1loong1), libicu67, libgtk-3-0, libepoxy0, libxml2, libxslt1.1, liblcms2-2, libwoff1, libfontconfig1, libfreetype6, libharfbuzz0b, libharfbuzz-icu0, libjpeg62-turbo, libpng16-16, libopenjp2-7, libwebp6, libwebpdemux2, libsecret-1-0, libhyphen0, libx11-6, libxcomposite1, libxdamage1, libxrender1, libxt6, libgbm1, libdrm2, libegl1, libglx0, libc6 (>= 2.28), libseccomp2` |
| `clash-libwebkit2gtk-4.1-data` | `（无 Depends）` |
| `clash-libwebkit2gtk-4.1-dev` | `clash-libwebkit2gtk-4.1-0 (= 2.40.5-1loong1), clash-libjavascriptcoregtk-4.1-dev (= 2.40.5-1loong1), clash-libsoup-3.0-dev (= 3.6.5-1loong1), clash-libgstreamer1.0-dev (= 1.22.0-1loong1), clash-libgstreamer-plugins-base1.0-dev (= 1.22.0-1loong1), clash-libgstreamer-gl1.0-dev (= 1.22.0-1loong1), clash-libglib2.0-dev (= 2.70.5-1loong1), libgtk-3-dev, libicu-dev, libxml2-dev, libxslt1-dev, libsecret-1-dev, libhyphen-dev, libwoff-dev, pkg-config` |
| `clash-libwebkit2gtk-4.1-examples` | `clash-libwebkit2gtk-4.1-0 (= 2.40.5-1loong1)` |

## 实际构建配置与验证边界

既有配置脚本 `/tmp/clash-verge-rev-v2.5.8-20261101/configure-webkit-gst122-01.sh` 选择 GTK3/Soup3、GStreamer 1.22.0、GCC13 C/C++、Release/O2、私有安装 RPATH 与 oldworld loader。LoongArch 旧基座使用 C loop（JIT/DFG/FTL 关闭）、system malloc；视频与 Web Audio 开启，WebRTC/MediaRecorder/Wayland/gamepad/spellcheck/AVIF/JPEGXL、文档/introspection、API/layout tests 和 WebDriver 关闭，MiniBrowser 开启。这个功能边界不能称作上游所有功能均已启用。此轮只审查既有二进制与元数据，不编辑/重编 WebKit 源码或改变沙箱设置。

已有 `logs/family-v2-webkit-process-maps-final.log` 记录 MiniBrowser/WebKitWebProcess/WebKitNetworkProcess 实际加载私有 GLib、Soup3、GStreamer、JSC、WebKit 与 GCC runtimes；同份进程映射保留 ICU63/ICU67 共存事实，不能外推成任意宿主 ICU 混用场景均已认证。`logs/webkit-minibrowser-final-v2510.log` 仍有旧 GTK 的 `gtk-overlay-scrolling` 和 accessibility bus 警告。既有页面/子进程 smoke 不是完整 API/layout/security suite；Clash 最终 GUI 验收由主 Agent 单独掌握，本报告不重跑或扩张其结论。包的安装、ELF flags、GLIBC floor 与闭包依据见共享部分。

## 最终产物

本来源的既有 ABI 审计覆盖 7 个 ELF，最高导入版本为 GLIBC 2.27、CXXABI 1.3.9、GLIBCXX 3.4.32；这是文件实际导入版本，control 可能为统一基座要求保留更保守的 libc6 >=2.28。按来源归类结果由原始逐文件审计派生，记录在 `archive-review-20261112/family-abi-summary-from-accepted-evidence.json`。

| 包 | 完整版本 | 架构 | 字节 | SHA256 |
|---|---|---|---:|---|
| [clash-libjavascriptcoregtk-4.1-0](../binaries/clash-libjavascriptcoregtk-4.1-0/loongarch64/clash-libjavascriptcoregtk-4.1-0_v2.40.5-1loong1_loongarch64.deb) | 2.40.5-1loong1 | loongarch64 | 3237706 | `985c9cbfc1d9095d1c49fb0490f488004414d8aaaffc6bd4180d3eb0077dd180` |
| [clash-libjavascriptcoregtk-4.1-bin](../binaries/clash-libjavascriptcoregtk-4.1-bin/loongarch64/clash-libjavascriptcoregtk-4.1-bin_v2.40.5-1loong1_loongarch64.deb) | 2.40.5-1loong1 | loongarch64 | 151916 | `ad71bc3a71e0d56e15378f844670a9a5218c4d1db758ff64df3ee21bfda67cde` |
| [clash-libjavascriptcoregtk-4.1-dev](../binaries/clash-libjavascriptcoregtk-4.1-dev/loongarch64/clash-libjavascriptcoregtk-4.1-dev_v2.40.5-1loong1_loongarch64.deb) | 2.40.5-1loong1 | loongarch64 | 73788 | `4ba362c3312178a3981da1ad6956c67b44c9d066e20ff184f77f572687d0189c` |
| [clash-libwebkit2gtk-4.1-0](../binaries/clash-libwebkit2gtk-4.1-0/loongarch64/clash-libwebkit2gtk-4.1-0_v2.40.5-1loong1_loongarch64.deb) | 2.40.5-1loong1 | loongarch64 | 20724840 | `36468cd95ef46ac6e44f81356bc656d6cf0c89bf8e8907c5c2a599a62527329d` |
| [clash-libwebkit2gtk-4.1-data](../binaries/clash-libwebkit2gtk-4.1-data/all/clash-libwebkit2gtk-4.1-data_v2.40.5-1loong1_all.deb) | 2.40.5-1loong1 | all | 359890 | `d3740b5c16c5dab06736aa4cdd6a510e763a1fab1a0ef867281113b1886060ca` |
| [clash-libwebkit2gtk-4.1-dev](../binaries/clash-libwebkit2gtk-4.1-dev/loongarch64/clash-libwebkit2gtk-4.1-dev_v2.40.5-1loong1_loongarch64.deb) | 2.40.5-1loong1 | loongarch64 | 129552 | `9c905466c9dce252722fcd894806c2bf2ad172d8030f255bd1471cf33ab2d59f` |
| [clash-libwebkit2gtk-4.1-examples](../binaries/clash-libwebkit2gtk-4.1-examples/loongarch64/clash-libwebkit2gtk-4.1-examples_v2.40.5-1loong1_loongarch64.deb) | 2.40.5-1loong1 | loongarch64 | 103414 | `ef253d4c45feef9b6c3f5ae3c7717e726d6c2a5c65027ebd1a483f31c6b2415c` |

## 共享归档与验证依据

完整交付索引为 [clash-native-family_v2.5.10_artifacts.json](clash-native-family_v2.5.10_artifacts.json)，可读索引为 [同名 Markdown](clash-native-family_v2.5.10_artifacts.md)。每个 `.deb` 使用完整 Debian Version 命名，旁附 `.deb.sha256`；Architecture:all 的数据/文档包放在 `all/`。目标旧世界发行版的实际 dpkg 架构名为 `loongarch64`，不同于当前 Debian sid 的 `loong64`。

主证据目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/native-family`。本轮沿用已通过检查并由主 Agent 归档的 36 个 v4 包，没有重建、重拷或重跑整组/GUI 测试；仅 GCC 三个工具由独立 v7 候选替换。既有 `native-family-final-manifest-v2510.json` 对应历史 v2，不应作为最终哈希表。最终选择以本共享索引、`archive-review-20261112/archived-artifacts.json` 和 GCC v7 归档记录为准。

`logs/native-family-v4-archive-and-payload-audit.json` 检查了全部 39 个历史 v4 归档、文件 md5 与版权；`logs/native-family-v4-postinstall-integrity.json` 记录专用 Buster 容器实际安装、39 包版本匹配、2742 个普通文件 md5、`dpkg --verify` 和 `apt-get check` 通过。`logs/dt-needed-package-closure-v2510-v4-01.json` 对 133 个 ELF、723 条 DT_NEEDED 依赖边及实际包归属检查为 PASS：使用记录明确指定的私有运行库环境，不能解释成任意裸启动环境均自动解析私有库。

既有 `logs/family-v2-abi-audit.json` 的 133 个原生 ELF 均为 LoongArch、flags `0x3`；50 个程序使用 `/lib64/ld.so.1`，其余 83 个没有 PT_INTERP；整组最高导入 GLIBC 为 2.28、GLIBCXX 为 3.4.32。v4 的内容变化/不变范围另见其 manifest。所有包使用私有 `clash-*` 名称，无用于取代系统包的 `Provides/Conflicts/Breaks/Replaces`，没有覆盖宿主公共库、修改全局 ld.so 配置或全局环境。私有 SDK 的消费者应显式设置 `PKG_CONFIG_PATH=/usr/lib/clash-verge/runtime/lib/pkgconfig` 并选择相应加载路径。

本轮只在现有容器组装 GCC 包和进行有针对性的安装验证，所有实际编译/链接/运行均受宿主共享 flock 保护。未运行各上游完整测试集，未新增 GUI 验收，也未启动/停止/执行 Magpie。应用最终 GUI 验收与推送发布由主 Agent 负责；本执行 Agent 未 commit 或 push。
