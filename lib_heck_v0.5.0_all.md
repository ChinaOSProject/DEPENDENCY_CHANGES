# heck 0.5.0 源码开发包交付记录

日期：2026-11-12。该普通依赖使用未经修改的上游 crate 源码，作为 `Architecture: all` 的 Rust 源码开发包交付。本记录不涉及 Rust 编译器源码贡献。

## 上游输入与原样保留

- 项目 GitHub：`https://github.com/withoutboats/heck`
- crate：`heck 0.5.0`
- 原版归档：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/tools-official-inputs-20261112/archives/heck-0.5.0.crate`
- 原版归档 SHA256：`2304e00983f87ffb38b55b444b5e3b60a884b5d30c0fca7d82fe33449bbe55ea`
- 原版 Cargo.toml SHA256：`db0e63b72de157a2f5194458b73b6542c3f4e17e33ab34a71d0260bfc035467e`
- 既有 Cargo 目录源校验元数据 `.cargo-checksum.json` SHA256：`0aaf45afdf9ff05597bf2ded4af1548bf66b34ff08ae54e4371242c4022d7dd2`
- 许可声明：`MIT OR Apache-2.0`；本包分发条款：`MIT or Apache-2.0`。

原始 archive 的全部 16 个文件、Cargo.toml、Cargo.toml.orig（如存在）、源码、测试/示例文件和完整上游许可均逐文件核对，未裁剪、重写或修改。校验元数据保持既有已验证目录源的原字节，`package` 校验值等于上面的原版 archive SHA256，`files` 与原版 archive 文件内容逐项吻合；它不是伪称 archive 自带的额外文件。

完整许可/通知文件及 SHA256：

- `LICENSE-APACHE`：`a60eea817514531668d7e00765731449fe14d059d3249e0bc93b36de45f759f2`
- `LICENSE-MIT`：`7b63ecd5f1902af1b63729947373683c32745c16a10e8e6292e2e2dcd7e90ae0`

## Debian 依据与分包差异

直接读取的真实 Debian 包装源码为 `rust-heck_0.5.0-1+really0.5.0-2.debian.tar.xz`，缓存位置 `/tmp/clash-verge-rev-v2.5.8-20261101/packaging-references/rust-crates-main/rust-heck_0.5.0-1+really0.5.0-2.debian.tar.xz`，SHA256 `c8d0eaffdd73b3a635a0210dafba91ca05bf5c2e1f7edd28414a6f4f44713ce5`。其中 `debian/control` 与独立保存的 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/references/rust-heck.control` 逐字节一致，SHA256 `0d342698371e917e93bcc17a996aefa74b09c02eb03ccb9cc455e6dd799d8df7`。可核对的发行版归档路径为 `https://deb.debian.org/debian/pool/main/r/rust-heck/rust-heck_0.5.0-1+really0.5.0-2.debian.tar.xz`。本次未重新联网下载或刷新该缓存。

沿用其 `rust-heck` 源包和 `librust-heck-dev` 二进制包命名、`/usr/share/cargo/registry/heck-0.5.0` 布局及对应版本别名。当前真实输入版本仍是 0.5.0，未把参考归档的版本替换成上游输入版本。参考 control 的 any/same 改为 all/foreign，因为本包只有原样可移植源码、许可和文档，不含 ELF、目标对象、`.a`、`.rlib`、`.rmeta`、共享库或已编译 SDK；不存在应另拆的运行库/SONAME/工具包。

实际验证的特性别名：`default`。该 crate 无额外特性表，沿用真实 Debian control 中的空默认特性别名。

该实际特性闭包没有必需的外部 Rust 源码依赖，因此未声明 Depends。rustc/Cargo、MSRV 和链接工具属于源码消费者的构建前提，未机械转成此源码包的运行依赖。上游仅测试/开发用或未经选择的可选/其他平台依赖仍保留在原 manifest 中，本次没有声称其完整闭包已经交付。

## 实际包

- Package：`librust-heck-dev`
- Version：`0.5.0-1`
- Architecture：`all`
- Multi-Arch：`foreign`
- 大小：14016 bytes
- SHA256：`10a4503cafa2c6a8ae4c9114e1f62be2d6d1540ffc79796fc85841f1e48c320c`
- 正式候选路径：`/home/laevatein/ChinaOSProject/binaries/librust-heck-dev/all/librust-heck-dev_v0.5.0-1_all.deb`
- 校验 sidecar：`/home/laevatein/ChinaOSProject/binaries/librust-heck-dev/all/librust-heck-dev_v0.5.0-1_all.deb.sha256`
- 原始 control SHA256：`8c5c7cd762fae43974483745cc8a3f4bb18fb76cf96af838ef8e815df83f226e`

实际 control（不是发行版变量模板）：

```debcontrol
Package: librust-heck-dev
Source: rust-heck
Version: 0.5.0-1
Section: rust
Priority: optional
Architecture: all
Multi-Arch: foreign
Maintainer: ChinaOSProject <packages@chinaosproject.local>
Installed-Size: 60
Provides: librust-heck+default-dev (= 0.5.0-1), librust-heck-0+default-dev (= 0.5.0-1), librust-heck-0-dev (= 0.5.0-1), librust-heck-0.5+default-dev (= 0.5.0-1), librust-heck-0.5-dev (= 0.5.0-1), librust-heck-0.5.0+default-dev (= 0.5.0-1), librust-heck-0.5.0-dev (= 0.5.0-1)
Homepage: https://github.com/withoutboats/heck
X-Cargo-Crate: heck
X-Cargo-Crate-Version: 0.5.0
Description: Rust source for heck
 Unchanged official crate source and full upstream notices for offline
 Cargo directory-source consumers. Only the documented feature scope
 is advertised; this portable source package has no runtime SDK dependency.
```

## 已执行的隔离验证

组装、xz 压缩、dpkg 独立 root 安装及消费编译均在已有专用容器 `codex-v0.166.0-v8-directory-probe-20261107` 内完成，持共同 host flock，最多 2 jobs，离线。宿主仅复制已验证包和生成本报告；未宿主安装，未改容器共享 `/usr`。

10 包实际整组安装成功，dpkg 查询得到精确版本及 all/installed。每个包的 md5sums、dpkg 文件归属、唯一文件所有权、原始源码、完整通知与校验元数据通过校验。本包有 23 条已校验文件记录，组内共 201 条，无重复所有权。随后只把已安装的这 10 个源码树迁到无关路径，使用全新空 Cargo home 和显式 directory-source replacement，离线生成精确 lock；locked/offline metadata 与依赖树证实仅解析这 10 个精确版本，没有借用原 vendor 或宏/开发依赖。

消费者对本包的实际检查：检查 HTTPServer 的缩略词边界 snake_case 转换和 Hello World 的 kebab-case 转换。

普通 locked/offline release build 时 RUSTC_BOOTSTRAP 未设置，程序实际执行成功；另用独立 target、RUSTC_BOOTSTRAP=-1 执行 cargo check 成功，故意启用不稳定特性的负对照以 E0554 正确失败。未重复上游全量测试，也未重建 Rust 编译器、标准库或工具。

消费工具如实标识为 `rustc 1.99.1-dev` / LLVM 23.1.1 和 `cargo 1.99.1-dev (5f94df478 2026-08-27)`，不是官方 stable 发布身份。最终测试程序 SHA256 `25ca9b2628755ed5c778cd933dcaa89d64178668f025b75f3e3eb08206fdfe7a`，machine 258、flags 0x3、解释器 /lib64/ld.so.1，直接 GLIBC 符号上限 2.28，容器内动态依赖全部解析。该临时测试程序包含容器内 GCC 运行库 RUNPATH，并非本源码包交付内容。

必须保留的证据界限：本包消费者生成的 rlib ELF 成员标志实际为 `{"0x43": 3}`；10 个 rlib 共 34 个 ELF 成员均为 0x43，未把其改写/宣称成 0x3。最终可执行文件运行成功不代表编译器、标准库、对象文件 ABI 或编译器源码发布审核已经通过。工具及其受保护输入的前后哈希一致。

## 证据位置

统一任务目录：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112`。

- `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/packages-index.json`：包、control、原始文件和归档 SHA256。
- `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/package-file-verification.json`：实际安装文件的 201 条检查。
- `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/consumer-result.json`：全部 24 个步骤、日志 SHA256、消费输出和输入保护结果。
- `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/consumer-rlib-object-flags.json`：实际新编译的 rlib 成员标志。
- `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/installed-relocated-fixture`：隔离 dpkg root、迁移源码、consumer、lock、日志与目标输出。
- `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/deliverables-manifest.json`：正式候选包/报告及校验值汇总。

源码无补丁；先前 PSM、native C SDK、LightningCSS 和 float16 产物未变。未运行或向 Magpie 发送信号，未 commit 或 push；候选提交主 Agent 独立审核发布。
