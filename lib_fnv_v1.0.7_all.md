# fnv 1.0.7 源码开发包交付记录

日期：2026-11-12。该普通依赖使用未经修改的上游 crate 源码，作为 `Architecture: all` 的 Rust 源码开发包交付。本记录不涉及 Rust 编译器源码贡献。

## 上游输入与原样保留

- 项目 GitHub：`https://github.com/servo/rust-fnv`
- crate：`fnv 1.0.7`
- 原版归档：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/tools-official-inputs-20261112/archives/fnv-1.0.7.crate`
- 原版归档 SHA256：`3f9eec918d3f24069decb9af1554cad7c880e2da24a9afd88aca000531ab82c1`
- 原版 Cargo.toml SHA256：`0854ced32611c6389356c0b43ae663da9d4480376c5f0825b82eb442dd84e8d8`
- 既有 Cargo 目录源校验元数据 `.cargo-checksum.json` SHA256：`7caf0dd34528f64c63ca85b77ddd82c1dbb7633dab1f928b140a0ce96a64b1d5`
- 许可声明：`Apache-2.0 / MIT`；本包分发条款：`MIT or Apache-2.0`。

原始 archive 的全部 9 个文件、Cargo.toml、Cargo.toml.orig（如存在）、源码、测试/示例文件和完整上游许可均逐文件核对，未裁剪、重写或修改。校验元数据保持既有已验证目录源的原字节，`package` 校验值等于上面的原版 archive SHA256，`files` 与原版 archive 文件内容逐项吻合；它不是伪称 archive 自带的额外文件。

完整许可/通知文件及 SHA256：

- `LICENSE-APACHE`：`a60eea817514531668d7e00765731449fe14d059d3249e0bc93b36de45f759f2`
- `LICENSE-MIT`：`65fdb6c76cd61612070c066eec9ecdb30ee74fb27859d0d9af58b9f499fd0c3e`

## Debian 依据与分包差异

直接读取的真实 Debian 包装源码为 `rust-fnv_1.0.7-1.debian.tar.xz`，缓存位置 `/tmp/clash-verge-rev-v2.5.8-20261101/packaging-references/rust-crates-main/rust-fnv_1.0.7-1.debian.tar.xz`，SHA256 `c4d3dbec893ae5e379d3a9bb168014538591835a295f8e94cd6872489a354870`。其中 `debian/control` 与独立保存的 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-leaf-tranche-20261112/references/rust-fnv.control` 逐字节一致，SHA256 `c10dfbfc143a065251b7de402aed7fad18ba026363db901cd57d7312b71426a8`。可核对的发行版归档路径为 `https://deb.debian.org/debian/pool/main/r/rust-fnv/rust-fnv_1.0.7-1.debian.tar.xz`。本次未重新联网下载或刷新该缓存。

沿用其 `rust-fnv` 源包和 `librust-fnv-dev` 二进制包命名、`/usr/share/cargo/registry/fnv-1.0.7` 布局及对应版本别名。当前真实输入版本仍是 1.0.7，未把参考归档的版本替换成上游输入版本。参考 control 的 any/same 改为 all/foreign，因为本包只有原样可移植源码、许可和文档，不含 ELF、目标对象、`.a`、`.rlib`、`.rmeta`、共享库或已编译 SDK；不存在应另拆的运行库/SONAME/工具包。

实际验证的特性别名：`default`, `std`。验证并声明 default 和 std；原默认特性只启用无额外依赖的 std。

该实际特性闭包没有必需的外部 Rust 源码依赖，因此未声明 Depends。rustc/Cargo、MSRV 和链接工具属于源码消费者的构建前提，未机械转成此源码包的运行依赖。上游仅测试/开发用或未经选择的可选/其他平台依赖仍保留在原 manifest 中，本次没有声称其完整闭包已经交付。

## 实际包

- Package：`librust-fnv-dev`
- Version：`1.0.7-1`
- Architecture：`all`
- Multi-Arch：`foreign`
- 大小：13276 bytes
- SHA256：`df7132f1e496c8241d673bfc9f0e80f0c6999b2bf6b84642e2d961fed0b4171b`
- 正式候选路径：`/home/laevatein/ChinaOSProject/binaries/librust-fnv-dev/all/librust-fnv-dev_v1.0.7-1_all.deb`
- 校验 sidecar：`/home/laevatein/ChinaOSProject/binaries/librust-fnv-dev/all/librust-fnv-dev_v1.0.7-1_all.deb.sha256`
- 原始 control SHA256：`6968e1a96b8764012d060e8e885dd20d851d6393153ff41b3f9e96e182a582cd`

实际 control（不是发行版变量模板）：

```debcontrol
Package: librust-fnv-dev
Source: rust-fnv
Version: 1.0.7-1
Section: rust
Priority: optional
Architecture: all
Multi-Arch: foreign
Maintainer: ChinaOSProject <packages@chinaosproject.local>
Installed-Size: 51
Provides: librust-fnv+default-dev (= 1.0.7-1), librust-fnv+std-dev (= 1.0.7-1), librust-fnv-1+default-dev (= 1.0.7-1), librust-fnv-1+std-dev (= 1.0.7-1), librust-fnv-1-dev (= 1.0.7-1), librust-fnv-1.0+default-dev (= 1.0.7-1), librust-fnv-1.0+std-dev (= 1.0.7-1), librust-fnv-1.0-dev (= 1.0.7-1), librust-fnv-1.0.7+default-dev (= 1.0.7-1), librust-fnv-1.0.7+std-dev (= 1.0.7-1), librust-fnv-1.0.7-dev (= 1.0.7-1)
Homepage: https://github.com/servo/rust-fnv
X-Cargo-Crate: fnv
X-Cargo-Crate-Version: 1.0.7
Description: Rust source for fnv
 Unchanged official crate source and full upstream notices for offline
 Cargo directory-source consumers. Only the documented feature scope
 is advertised; this portable source package has no runtime SDK dependency.
```

## 已执行的隔离验证

组装、xz 压缩、dpkg 独立 root 安装及消费编译均在已有专用容器 `codex-v0.166.0-v8-directory-probe-20261107` 内完成，持共同 host flock，最多 2 jobs，离线。宿主仅复制已验证包和生成本报告；未宿主安装，未改容器共享 `/usr`。

10 包实际整组安装成功，dpkg 查询得到精确版本及 all/installed。每个包的 md5sums、dpkg 文件归属、唯一文件所有权、原始源码、完整通知与校验元数据通过校验。本包有 16 条已校验文件记录，组内共 201 条，无重复所有权。随后只把已安装的这 10 个源码树迁到无关路径，使用全新空 Cargo home 和显式 directory-source replacement，离线生成精确 lock；locked/offline metadata 与依赖树证实仅解析这 10 个精确版本，没有借用原 vendor 或宏/开发依赖。

消费者对本包的实际检查：FNV-1a 的 hello 已知向量等于 0xa430d84680aabd0b，并完成 FnvHashMap 插入及查询。

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
