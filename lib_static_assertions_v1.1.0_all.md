# static_assertions 1.1.0 源码开发包交付记录

日期：2026-11-12。该普通依赖使用未经修改的上游 crate 源码，作为 `Architecture: all` 的 Rust 源码开发包交付。本记录不涉及 Rust 编译器源码贡献。

## 上游输入与原样保留

- 项目 GitHub：`https://github.com/nvzqz/static-assertions-rs`
- crate：`static_assertions 1.1.0`
- 原版归档：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/tools-official-inputs-20261112/archives/static_assertions-1.1.0.crate`
- 原版归档 SHA256：`a2eb9349b6444b326872e140eb1cf5e7c522154d69e7a0ffb0fb81c06b37543f`
- 原版 Cargo.toml SHA256：`6531dbe3d557e427f9e3510e50cdf3de751a319eece11c9a937b35cfab8744c4`
- 既有 Cargo 目录源校验元数据 `.cargo-checksum.json` SHA256：`0bb6cd75842131047c3f45489c4d9a170d2619e954ab24e4535f25c71109ad4a`
- 许可声明：`MIT OR Apache-2.0`；本包分发条款：`MIT or Apache-2.0`。

原始 archive 的全部 17 个文件、Cargo.toml、Cargo.toml.orig（如存在）、源码、测试/示例文件和完整上游许可均逐文件核对，未裁剪、重写或修改。校验元数据保持既有已验证目录源的原字节，`package` 校验值等于上面的原版 archive SHA256，`files` 与原版 archive 文件内容逐项吻合；它不是伪称 archive 自带的额外文件。

完整许可/通知文件及 SHA256：

- `LICENSE-APACHE`：`cfc7749b96f63bd31c3c42b5c471bf756814053e847c10f3eb003417bc523d30`
- `LICENSE-MIT`：`ea084a2373ebc1f0902c09266e7bf25a05ab3814c1805bb017ffa7308f90c061`

## Debian 依据与分包差异

直接读取的真实 Debian 包装源码为 `rust-static-assertions_1.1.0-1.debian.tar.xz`，缓存位置 `/tmp/clash-verge-rev-v2.5.8-20261101/packaging-references/rust-crates-main/rust-static-assertions_1.1.0-1.debian.tar.xz`，SHA256 `d93b27f5901c7f2b2b80725a14b7918e6f97028d966d5e7a72e0590fed4df912`。其中 `debian/control` 与独立保存的 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/references/rust-static-assertions.control` 逐字节一致，SHA256 `50a3878dd039363dff446b82bcaf9b18141dee4bea2b2f54c0adfe056dcd10f7`。可核对的发行版归档路径为 `https://deb.debian.org/debian/pool/main/r/rust-static-assertions/rust-static-assertions_1.1.0-1.debian.tar.xz`。本次未重新联网下载或刷新该缓存。

沿用其 `rust-static-assertions` 源包和 `librust-static-assertions-dev` 二进制包命名、`/usr/share/cargo/registry/static_assertions-1.1.0` 布局及对应版本别名。当前真实输入版本仍是 1.1.0，未把参考归档的版本替换成上游输入版本。参考 control 的 any/same 改为 all/foreign，因为本包只有原样可移植源码、许可和文档，不含 ELF、目标对象、`.a`、`.rlib`、`.rmeta`、共享库或已编译 SDK；不存在应另拆的运行库/SONAME/工具包。

实际验证的特性别名：`default`。验证默认稳定语法闭包；原 manifest 中 nightly 特性保留，但未声明 +nightly Provides，也未以启用 nightly 代替正常的消费者验证。

该实际特性闭包没有必需的外部 Rust 源码依赖，因此未声明 Depends。rustc/Cargo、MSRV 和链接工具属于源码消费者的构建前提，未机械转成此源码包的运行依赖。上游仅测试/开发用或未经选择的可选/其他平台依赖仍保留在原 manifest 中，本次没有声称其完整闭包已经交付。

## 实际包

- Package：`librust-static-assertions-dev`
- Version：`1.1.0-1`
- Architecture：`all`
- Multi-Arch：`foreign`
- 大小：20204 bytes
- SHA256：`65aeca0720db028951d82d529103e65990bf4157bdfda782c46688641ee102c5`
- 正式候选路径：`/home/laevatein/ChinaOSProject/binaries/librust-static-assertions-dev/all/librust-static-assertions-dev_v1.1.0-1_all.deb`
- 校验 sidecar：`/home/laevatein/ChinaOSProject/binaries/librust-static-assertions-dev/all/librust-static-assertions-dev_v1.1.0-1_all.deb.sha256`
- 原始 control SHA256：`59695230169fa0afc464a82f05d0e3cd07bc9cddfdd1fb80f761052a419f4a92`

实际 control（不是发行版变量模板）：

```debcontrol
Package: librust-static-assertions-dev
Source: rust-static-assertions
Version: 1.1.0-1
Section: rust
Priority: optional
Architecture: all
Multi-Arch: foreign
Maintainer: ChinaOSProject <packages@chinaosproject.local>
Installed-Size: 86
Provides: librust-static-assertions+default-dev (= 1.1.0-1), librust-static-assertions-1+default-dev (= 1.1.0-1), librust-static-assertions-1-dev (= 1.1.0-1), librust-static-assertions-1.1+default-dev (= 1.1.0-1), librust-static-assertions-1.1-dev (= 1.1.0-1), librust-static-assertions-1.1.0+default-dev (= 1.1.0-1), librust-static-assertions-1.1.0-dev (= 1.1.0-1)
Homepage: https://github.com/nvzqz/static-assertions-rs
X-Cargo-Crate: static_assertions
X-Cargo-Crate-Version: 1.1.0
Description: Rust source for static_assertions
 Unchanged official crate source and full upstream notices for offline
 Cargo directory-source consumers. Only the documented feature scope
 is advertised; this portable source package has no runtime SDK dependency.
```

## 已执行的隔离验证

组装、xz 压缩、dpkg 独立 root 安装及消费编译均在已有专用容器 `codex-v0.166.0-v8-directory-probe-20261107` 内完成，持共同 host flock，最多 2 jobs，离线。宿主仅复制已验证包和生成本报告；未宿主安装，未改容器共享 `/usr`。

10 包实际整组安装成功，dpkg 查询得到精确版本及 all/installed。每个包的 md5sums、dpkg 文件归属、唯一文件所有权、原始源码、完整通知与校验元数据通过校验。本包有 24 条已校验文件记录，组内共 201 条，无重复所有权。随后只把已安装的这 10 个源码树迁到无关路径，使用全新空 Cargo home 和显式 directory-source replacement，离线生成精确 lock；locked/offline metadata 与依赖树证实仅解析这 10 个精确版本，没有借用原 vendor 或宏/开发依赖。

消费者对本包的实际检查：编译期验证 usize 为 64 位、正向 Clone/Send/Sync trait 约束和 Cell 的负向 Sync 约束。

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
