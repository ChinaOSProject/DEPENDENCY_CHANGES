# LightningCSS 1.33.0 LoongArch64 原生绑定及 Rust 开发包组报告

审查日期：2026-11-12（Asia/Shanghai）。原生 Node 绑定与 JS transform 验证成功。现已在既有容器内完成并归档 110 个 Rust source-development 包、114 个 crate 的精确依赖闭包；离线源码 consumer 在主构建退出后持有共享锁编译、运行并通过。原先的 source-dev 交付缺口已关闭。

## 精确上游与原生结果

- 官方仓库：https://github.com/parcel-bundler/lightningcss
- npm 版本：`1.33.0`；官方 npm metadata 的 `gitHead`：`1d680fa14e9a089c92dc0929f869d6757ae91c30`。
- 核心 Cargo crate：`lightningcss 1.0.0-alpha.72`；Node wrapper：`lightningcss_node 0.1.0`。
- 已有原生输出：`target/lightningcss/release/liblightningcss_node.so`，17,080,704 字节，SHA256 `7c1819b6882ac7f0539253fdaece143893e8bbea09d425f4846f180a60e30851`。
- 主 Agent 已将该绑定放入实际 npm fallback loader 路径 `lightningcss/lightningcss.linux-loong64-gnu.node`。真实 JS `transform` API 已输出 `a{color:red;margin:0}`。
- ELF 为 LoongArch、flags `0x3`，最高 GLIBC 要求 `2.28`。移除 `LD_LIBRARY_PATH` 后所有 native dependencies 解析成功，无 LLVM 运行时依赖。

原生绑定仍由实际应用的 JS loader 使用；没有把缺少完整 JS API 的裸 `.node` 文件冒充系统 Node 包，也没有编译任务之外的 LightningCSS CLI。

## Debian 依据和原版 API

Debian sid 的实际 source package 是 **`lightningcss`**，VCS 仓库名才是 `rust-lightningcss`。参考二进制包为 CLI `lightningcss 1.33.0+ds-8` 和 `librust-lightningcss-dev 1.0.0+1.33.0+ds-8`（Architecture: all）。已保存的官方 control、rules、patch series 及 filelist 证明 source-dev 包共同安装五个 crate 到 `/usr/share/cargo/registry/`：

| crate | 精确版本 |
| --- | --- |
| lightningcss | 1.0.0-alpha.72 |
| lightningcss-derive | 1.0.0-alpha.43 |
| parcel_selectors | 0.28.3 |
| static-self | 0.1.2 |
| static-self-derive | 0.1.1 |

参考来源：

- https://sources.debian.org/data/main/l/lightningcss/1.33.0%2Bds-8/debian/control
- https://sources.debian.org/data/main/l/lightningcss/1.33.0%2Bds-8/debian/rules
- https://sources.debian.org/data/main/l/lightningcss/1.33.0%2Bds-8/debian/patches/series
- https://packages.debian.org/sid/all/librust-lightningcss-dev/filelist

本组沿用上述五-crate 主包布局，依赖采用 Debian 常见 `librust-*-dev` 和旧 API 系列的版本后缀包名，声明版本化 crate/feature Provides。没有应用 Debian 的 API 升级补丁：原版要求 clap 3、cssparser 0.37、cssparser-color 0.5、dashmap 5、itertools 0.10、pastey 0.1，与 Debian 的 4/0.38/0.6/6/0.14/0.2 保持区分。

## 精确 source-dev 闭包与输入

实际原生 fingerprints 显示核心编译 features 为 `default, bundler, nodejs, sourcemap, serde, visitor, into_owned`，以及由依赖激活的 `dashmap, parcel_sourcemap, rayon, static-self`。用独立 consumer 对这些功能执行 Cargo tree 与 metadata。实际 LoongArch normal/build 编译树为 **86 个 crate**，与原生 fingerprints 的 86 个精确版本证据一致；Cargo metadata 还包含 28 个不会在该树编译的可选依赖 / 跨目标 / 条件 derive resolver 输入。filtered metadata 有 99 个依赖，另外 15 个仅出现在全部目标解析中；不能把 filtered metadata 的所有条目误称为已编译 crate。完整 resolver 闭包共 **114 个 crate / 110 个 source-dev 包**。五个本地 crate 合为主包，另外 109 个第三方 crate 各有实质源码内容。

全 workspace Cargo.lock 的 241 项不是本次 consumer 闭包。若把所有可选功能和目标要求一并展开，初步图有 164 个 crate / 160 个包，超出 150 包上限；本组没有扩大到该范围，也没有宣称交付全部可选功能或其开发测试依赖。

输入审查从五个既有 Cargo home 中找到 107 个精确缓存 `.crate`，全部 SHA256 与原 Cargo.lock 一致。五个本地 crate 来自精确上游 workspace。仅两个 resolver 所需源输入缺失；经主 Agent follow-up 5 明确授权，从官方 static.crates.io 下载到任务专属目录，未改动共享 Cargo cache：

| 输入 | 字节数 | Cargo.lock 核验 SHA256 |
| --- | ---: | --- |
| redox_syscall 0.5.8 | 26,319 | 03a862b389f93e68874fbf580b9de08dd02facb9a788ebadaf4a3fd33cf58834 |
| wasi 0.11.0+wasi-snapshot-preview1 | 28,131 | 9c8d87e72b64a3b4db28d11ce29237c246188f4f51057d65a7eab63b7987e423 |

这两项是 Redox/WASI 的 resolver 输入，不参与 LoongArch 编译。已核验的 109 个 archive 共 11,070,761 字节；包括五个本地 crate 的源输入共 3,761 文件、86,466,962 字节。当前无缺失源输入。

## 包内容、版本和 feature 范围

主包：`librust-lightningcss-dev_v1.0.0~alpha.72+1.33.0-0chinaos1_all.deb`，439,288 字节，SHA256 `b018910e8f1ec3891b725553ad72a9d4c23e71e56a02f55cd3ed913a14bed500`。Debian 版本用 `~alpha.72` 保留实际预发布语义，未采用 Debian 去掉 alpha 的版本。包 control 为 `Architecture: all`；交付文件名和归档目录采用用户已确认的 `all` 约定，包内是可移植源码。原生 LoongArch 验证范围不改变源码包的架构。

110 包合计 9,458,476 字节；提取后的 114 个 crate 目录共 86,794,482 字节。每个包有真实源码、原许可证/notice、copyright、changelog、md5sums，所有 Depends 精确约束到组内实际包版本。主包连同五个本地 crate 的依赖合并生成，而非复制已经 API 升级的 Debian Depends。完整包名、版本、SHA256、Depends、Provides 及正式路径在相邻的 `lightningcss-1.33.0-source-dev-packages.json`。任务内 `package-manifest.json` 保留原始物理 staging 文件名，`artifact-manifest.json` 的 `staging_*` 字段明确记录这些副本。

为支持独立 registry 布局，只在打包副本内规范化 Cargo.toml：移除顶层 workspace 关系和 dependency path，保留源版本、全部依赖/目标/feature 声明与 `Cargo.toml.orig`。Rust 源码和 API 不变；3,647 个非规范化清单文件、86,272,932 字节已与精确 archive/workspace 输入逐一比对一致。共享上游源码和缓存均未修改。

feature Provides 来自实际声明、隐式空 default 和原生编译 fingerprints。核心主包广告上述实际功能；不广告 `browserslist`、`cli`、`jsonschema`、`substitute_variables`。最后一项的依赖可能已存在，但它本身未在原构建中编译，因此不宣称支持。`parcel_selectors` 不广告 `bench`、`jsonschema`、`smallvec`；其他所有 crate 的已广告和未广告 feature 逐项见 `feature-scope.json`。纯 resolver crate 没有凭空获得编译 feature Provides。

版本并存采用 Debian 旧系列名称，例如 `librust-syn-1-dev`、`librust-phf-0.11-dev`。本组 cfg-if 1.0.0 使用 `librust-cfg-if-1-dev`，与已交付的 `librust-cfg-if-dev` 1.0.4 包名、doc 路径和 crate 路径均不同。与 float16 的五个已交付包共同提取后，LightningCSS 全部文件仍与 md5sums 一致。

## 安装后 Cargo 用法

整组 source-dev 包安装后，可在独立 consumer 的 `.cargo/config.toml` 中指定系统源码 registry：

```toml
[source.crates-io]
replace-with = "debian-packaged-sources"
[source.debian-packaged-sources]
directory = "/usr/share/cargo/registry"
[net]
offline = true
```

consumer 的 Cargo.toml 可声明 `lightningcss = { version = "=1.0.0-alpha.72", features = ["serde", "visitor"] }`；默认 bundler/nodejs/sourcemap 也已包含。此例针对本组的受支持 feature 范围。项目 lockfile 应保留，以选择已经核验的精确依赖版本。

## 验证、正式归档和证据

已通过：两个新 archive 和全部缓存 archive 的原 lock checksum、114-crate 离线 Cargo metadata、110 包的实际 archive control 中 171 条精确 Depends 闭包、全部 4,282 包文件的提取/MD5 检查、上述上游字节比对、float16 五包共同提取检查。

独立 CSS minify consumer 已从共同提取的包源码 registry，使用新建空 Cargo home 和 `cargo run --locked --offline --jobs 2` 编译、运行成功，输出严格等于 `a{color:red;margin:0}`，exit 0。Cargo home 没有 registry index/cache，未依赖外部 source cache 或网络；共同 registry 也包含 float16 五包，但本 consumer 的 lock 保留本组精确版本。host wrapper 在主 Clash release 构建退出后才获取共享 `/tmp/chinaos-heavy-build-20261105.lock`，完整持锁执行 consumer；测试用时 145.981 秒。验证不包含全部 CSS 特性、独立 CLI、所有 optional features 或跨目标编译。

110 个包及相邻 `.deb.sha256` 已复读校验并正式归档到 `/home/laevatein/ChinaOSProject/binaries/{包名}/all/{包名}_v{完整版本}_all.deb`。完整交付表为本报告相邻的 `lightningcss-1.33.0-source-dev-packages.json`；feature 清单为 `lightningcss-1.33.0-source-dev-features.json`。

包组装和 consumer 验证均位于既有容器；已验证产物的正式归档和本次路径修正仅进行宿主文件操作。证据根目录为：

```text
/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/tool-package-review-20261111/
```

2026-11-12 路径修正只将本组 110 个已验证产物重新归档到 `all`，相邻 checksum sidecar 引用新的 `_all.deb` basename。全部包 SHA256 和字节数保持不变；新副本复读验证后才删除本组原错误位置的未跟踪正式副本。任务 cache 的原 staging 副本保留。没有重建 payload/control 或重跑已通过的 consumer。逐包记录见 `lightningcss-source-dev/archive-path-correction.json`。

主要证据：`lightningcss-cache-input-inventory.json`、`lightningcss-source-inputs/downloads.json`、`lightningcss-source-dev/{input-verification.json,metadata-all.json,metadata-loongarch.json,normalized-manifests.json,compiled-feature-evidence.json,feature-scope.json,package-manifest.json,group-verification.json,source-integrity-verification.json,archive-control-closure.json,consumer-verification.json,artifact-manifest.json}`。复现脚本为相邻 `lightningcss-{assess-inputs.mjs,fetch-missing.py,normalize-manifests.mjs,source-dev.py,source-dev-smoke.py}`。首轮缺口审查完整保留在 `lightningcss-source-dev-previous-gap-report.md` 和 `lightningcss-source-dev-gap.json`，其中未安装 Debian Rust 源包的事实已由本次缓存闭包组装解决，不再作为外部 blocker。

没有宿主构建或安装、共享容器根安装、npm 重装、上游代码/cache 修改、无关 CLI 构建、commit/push、Magpie/gtk-launch 执行、Herdr 操作或容器安全策略更改。pnpm、esbuild、TSRS、Rolldown 和 Wails 既有交付保持完整。
