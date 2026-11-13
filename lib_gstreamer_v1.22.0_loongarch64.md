# GStreamer 1.22.0：Clash 私有 core 与 plugins-base 分包

日期：2026-11-12。上游 GitHub 仓库：`https://github.com/GStreamer/gstreamer`；上游 monorepo：`https://gitlab.freedesktop.org/gstreamer/gstreamer`。本组 core 和 plugins-base 均为 1.22.0，完整包版本 `1.22.0-1loong1`，统一安装至 `/usr/lib/clash-verge/runtime`，不接管系统 GStreamer 注册表或公共库路径。

## Debian 参考与分包差异

参考源包为 Debian `gstreamer1.0 1.22.0-2+deb12u1` 与 `gst-plugins-base1.0 1.22.0-3+deb12u6`。准确版权文本已保存；本轮额外从 Debian Sources 保存两者真实 control，URL、SHA256 见 `archive-review-20261112/primary-reference-controls/fetch-manifest.json`：

- `https://sources.debian.org/src/gstreamer1.0/1.22.0-2+deb12u1/debian/control/`
- `https://sources.debian.org/src/gst-plugins-base1.0/1.22.0-3+deb12u6/debian/control/`

保留其 core runtime/dev/tools、base runtime/dev/plugins/apps、GL runtime、X11/GL 插件分工，加 `clash-` 前缀。与参考的具体差异是：`clash-gstreamer1.0-common` 独立承载 core/base 两套翻译与 licence-translations.dict；`clash-libgstreamer-gl1.0-dev` 从 Debian 通常随 base dev 交付的 GL 开发内容中细分出真实 GL 头文件/链接/pkg-config；`clash-gstreamer1.0-dev` 承载已生成的 `gst-hotdoc-plugins-scanner`（原生 ELF）与 Python `gst-plugins-doc-cache-generator`，所以它是有实际工具的 loongarch64 包。没有人为添加空的测试/gir/ALSA 插件包。

v4 将 `libgsttypefindfunctions.so` 从 core runtime 移到 `clash-gstreamer1.0-plugins-base`，因为它实际依赖 base 的 libgstpbutils；这样避免 core/base 互相依赖。记录在 `logs/native-family-v4-package-manifest.json` 的 ownership_moves，模块字节未改。v4 还为 GL/X11 插件补齐 JPEG、PNG、Cairo、PangoCairo、Xext、Xi 真实依赖，并补齐 base 软件许可，未经重编。

| 包 | 实际 control Depends |
|---|---|
| `clash-gstreamer1.0-common` | `（无 Depends）` |
| `clash-gstreamer1.0-dev` | `clash-libgstreamer1.0-dev (= 1.22.0-1loong1), clash-libgstreamer-plugins-base1.0-dev (= 1.22.0-1loong1), python3` |
| `clash-gstreamer1.0-gl` | `clash-libgstreamer-gl1.0-0 (= 1.22.0-1loong1), clash-gstreamer1.0-plugins-base (= 1.22.0-1loong1), libegl1, libglx0, libgbm1, libdrm2, libjpeg62-turbo, libpng16-16 (>= 1.6.2-1)` |
| `clash-gstreamer1.0-plugins-base` | `clash-libgstreamer1.0-0 (= 1.22.0-1loong1), clash-libgstreamer-plugins-base1.0-0 (= 1.22.0-1loong1)` |
| `clash-gstreamer1.0-plugins-base-apps` | `clash-gstreamer1.0-plugins-base (= 1.22.0-1loong1), clash-libgstreamer-plugins-base1.0-0 (= 1.22.0-1loong1)` |
| `clash-gstreamer1.0-tools` | `clash-libgstreamer1.0-0 (= 1.22.0-1loong1)` |
| `clash-gstreamer1.0-x` | `clash-libgstreamer-plugins-base1.0-0 (= 1.22.0-1loong1), clash-gstreamer1.0-plugins-base (= 1.22.0-1loong1), libpango-1.0-0, libx11-6, libcairo2 (>= 1.14.0), libpangocairo-1.0-0 (>= 1.42.4), libxext6 (>= 2:1.3.0), libxi6 (>= 2:1.6.99.1)` |
| `clash-libgstreamer-gl1.0-0` | `clash-libgstreamer-plugins-base1.0-0 (= 1.22.0-1loong1), libc6 (>= 2.28), libegl1, libglx0, libgbm1, libdrm2, libx11-6` |
| `clash-libgstreamer-gl1.0-dev` | `clash-libgstreamer-gl1.0-0 (= 1.22.0-1loong1), clash-libgstreamer-plugins-base1.0-dev (= 1.22.0-1loong1), libgl1-mesa-dev, libegl1-mesa-dev, libgles2-mesa-dev, libx11-xcb-dev, libdrm-dev, libgbm-dev, pkg-config` |
| `clash-libgstreamer-plugins-base1.0-0` | `clash-libgstreamer1.0-0 (= 1.22.0-1loong1), clash-libglib2.0-0 (= 2.70.5-1loong1), libc6 (>= 2.28), liborc-0.4-0` |
| `clash-libgstreamer-plugins-base1.0-dev` | `clash-libgstreamer-plugins-base1.0-0 (= 1.22.0-1loong1), clash-libgstreamer1.0-dev (= 1.22.0-1loong1), clash-libglib2.0-dev (= 2.70.5-1loong1), liborc-0.4-dev, pkg-config` |
| `clash-libgstreamer1.0-0` | `clash-libglib2.0-0 (= 2.70.5-1loong1), libc6 (>= 2.28), libcap2, libcap2-bin` |
| `clash-libgstreamer1.0-dev` | `clash-libgstreamer1.0-0 (= 1.22.0-1loong1), clash-libglib2.0-dev (= 2.70.5-1loong1), pkg-config` |

## 构建与已完成验证

已有 `/tmp/clash-verge-rev-v2.5.8-20261101/build-gst122.sh` 以 Meson/ninja、私有 prefix、release/nofallback 构建 core/base；关闭 tests/examples/docs/introspection，开启工具及 X11/EGL/GLX 的 GL 路径。旧版本 1.18.6 和尝试过的其他 source 不属于本次最终交付。

已有 `logs/gst-version-v2510-02.log` 记录 1.22.0；`logs/gst-audio-pipeline-final-v2510.status`、`logs/gst-video-pipeline-final-v2510.status` 均为 `exit=0 PASS`，对应有限 buffers 的 audiotestsrc/videotestsrc 到转换/缩放/重采样及 fakesink。它们验证插件扫描与音视频基础管线，不代表真实硬件播放、所有编解码器或完整 upstream suite。GL 构建存在不等于每种 GPU/显示后端均已验收。共享记录给出每个插件实际包归属与闭包；本轮不重复这些编译或 GUI 测试。

## 最终产物

本来源的既有 ABI 审计覆盖 53 个 ELF，最高导入版本为 GLIBC 2.28；这是文件实际导入版本，control 可能为统一基座要求保留更保守的 libc6 >=2.28。按来源归类结果由原始逐文件审计派生，记录在 `archive-review-20261112/family-abi-summary-from-accepted-evidence.json`。

| 包 | 完整版本 | 架构 | 字节 | SHA256 |
|---|---|---|---:|---|
| [clash-gstreamer1.0-common](../binaries/clash-gstreamer1.0-common/all/clash-gstreamer1.0-common_v1.22.0-1loong1_all.deb) | 1.22.0-1loong1 | all | 351626 | `5f9d7983d20197452526e6555f1fe32f80d91cde114633a7d5f5ed15bcb33e23` |
| [clash-gstreamer1.0-dev](../binaries/clash-gstreamer1.0-dev/loongarch64/clash-gstreamer1.0-dev_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 44162 | `112ea4ddcc2f216c736a1a575e2a7adafe4abe17db868bedd49dce38f3309568` |
| [clash-gstreamer1.0-gl](../binaries/clash-gstreamer1.0-gl/loongarch64/clash-gstreamer1.0-gl_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 133722 | `e177435f3ae5537c4a6e356a8afcdb3c077e92e924c737dc5dad3e98d463da4c` |
| [clash-gstreamer1.0-plugins-base](../binaries/clash-gstreamer1.0-plugins-base/loongarch64/clash-gstreamer1.0-plugins-base_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 551902 | `a6d0ddb78907a70ef259d2bef33343bf774f7c1149c7bbcf2d2dc5b4f8387ce4` |
| [clash-gstreamer1.0-plugins-base-apps](../binaries/clash-gstreamer1.0-plugins-base-apps/loongarch64/clash-gstreamer1.0-plugins-base-apps_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 60704 | `a6c9c901bad9124a4a33271b1cb5f692b466c59f9353b0b6432b2129f274080d` |
| [clash-gstreamer1.0-tools](../binaries/clash-gstreamer1.0-tools/loongarch64/clash-gstreamer1.0-tools_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 80696 | `99f4783ed1b4be74fc0c63f550fac07fb43d0533fe9d4c9125c97b83483bcaf6` |
| [clash-gstreamer1.0-x](../binaries/clash-gstreamer1.0-x/loongarch64/clash-gstreamer1.0-x_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 81514 | `4c8a0e3bfa1a6511a3d6c8e27a42dfe902926dbe261cbb34907e0aff2c297d61` |
| [clash-libgstreamer-gl1.0-0](../binaries/clash-libgstreamer-gl1.0-0/loongarch64/clash-libgstreamer-gl1.0-0_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 197128 | `0f35ca3103d3c68026c68420629fbdf392802126bb60e39a7da4f14783bb3ab7` |
| [clash-libgstreamer-gl1.0-dev](../binaries/clash-libgstreamer-gl1.0-dev/loongarch64/clash-libgstreamer-gl1.0-dev_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 61120 | `8fbccd5c77cd12d9e3bee1b8bee1e5d6eca13a9f6bf98800d082a78a05bb0bb6` |
| [clash-libgstreamer-plugins-base1.0-0](../binaries/clash-libgstreamer-plugins-base1.0-0/loongarch64/clash-libgstreamer-plugins-base1.0-0_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 817096 | `bf363bcda3f464123602482d67d3b47e5f05a92bb4c96cfc06e2109ede183b66` |
| [clash-libgstreamer-plugins-base1.0-dev](../binaries/clash-libgstreamer-plugins-base1.0-dev/loongarch64/clash-libgstreamer-plugins-base1.0-dev_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 152874 | `8918b1239a0d26d84570a7ee078e00f557bc828291822358bf01f716d6796fb3` |
| [clash-libgstreamer1.0-0](../binaries/clash-libgstreamer1.0-0/loongarch64/clash-libgstreamer1.0-0_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 931910 | `442ce8ac8f03936fe8facbe758ca1657b842d12ec6b9e60b2fbf8a34a4c35c1c` |
| [clash-libgstreamer1.0-dev](../binaries/clash-libgstreamer1.0-dev/loongarch64/clash-libgstreamer1.0-dev_v1.22.0-1loong1_loongarch64.deb) | 1.22.0-1loong1 | loongarch64 | 197370 | `005aedc1623ceccf7b519186cfc65fa613c794cad6a8b2f30d2337b8e7a0a188` |

## 共享归档与验证依据

完整交付索引为 [clash-native-family_v2.5.10_artifacts.json](clash-native-family_v2.5.10_artifacts.json)，可读索引为 [同名 Markdown](clash-native-family_v2.5.10_artifacts.md)。每个 `.deb` 使用完整 Debian Version 命名，旁附 `.deb.sha256`；Architecture:all 的数据/文档包放在 `all/`。目标旧世界发行版的实际 dpkg 架构名为 `loongarch64`，不同于当前 Debian sid 的 `loong64`。

主证据目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/native-family`。本轮沿用已通过检查并由主 Agent 归档的 36 个 v4 包，没有重建、重拷或重跑整组/GUI 测试；仅 GCC 三个工具由独立 v7 候选替换。既有 `native-family-final-manifest-v2510.json` 对应历史 v2，不应作为最终哈希表。最终选择以本共享索引、`archive-review-20261112/archived-artifacts.json` 和 GCC v7 归档记录为准。

`logs/native-family-v4-archive-and-payload-audit.json` 检查了全部 39 个历史 v4 归档、文件 md5 与版权；`logs/native-family-v4-postinstall-integrity.json` 记录专用 Buster 容器实际安装、39 包版本匹配、2742 个普通文件 md5、`dpkg --verify` 和 `apt-get check` 通过。`logs/dt-needed-package-closure-v2510-v4-01.json` 对 133 个 ELF、723 条 DT_NEEDED 依赖边及实际包归属检查为 PASS：使用记录明确指定的私有运行库环境，不能解释成任意裸启动环境均自动解析私有库。

既有 `logs/family-v2-abi-audit.json` 的 133 个原生 ELF 均为 LoongArch、flags `0x3`；50 个程序使用 `/lib64/ld.so.1`，其余 83 个没有 PT_INTERP；整组最高导入 GLIBC 为 2.28、GLIBCXX 为 3.4.32。v4 的内容变化/不变范围另见其 manifest。所有包使用私有 `clash-*` 名称，无用于取代系统包的 `Provides/Conflicts/Breaks/Replaces`，没有覆盖宿主公共库、修改全局 ld.so 配置或全局环境。私有 SDK 的消费者应显式设置 `PKG_CONFIG_PATH=/usr/lib/clash-verge/runtime/lib/pkgconfig` 并选择相应加载路径。

本轮只在现有容器组装 GCC 包和进行有针对性的安装验证，所有实际编译/链接/运行均受宿主共享 flock 保护。未运行各上游完整测试集，未新增 GUI 验收，也未启动/停止/执行 Magpie。应用最终 GUI 验收与推送发布由主 Agent 负责；本执行 Agent 未 commit 或 push。
