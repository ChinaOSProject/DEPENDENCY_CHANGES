# autocfg 1.5.1 普通源码依赖包记录

日期：2026-11-13（实际消费者验收：2026-11-12）。交付原样的普通第三方 Rust crate 源码开发包；不含新编译对象或 SDK。主 Agent 独立审核和发布。

## 上游、来源及完整字节

- GitHub 项目：`https://github.com/cuviper/autocfg`
- crate/version：`autocfg 1.5.1`
- 官方 crate 归档：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/tools-official-inputs-20261112/archives/autocfg-1.5.1.crate`
- 官方归档 SHA256：`f2032f911046de80f0a198e0901378627c33f59ea0ac00e363d481118bd70a53`
- 原版 Cargo.toml SHA256：`ed7c832b4250c3be736e040d68688c2b83a5b164a93ef533e61532cbe0d44e99`
- 既有目录源 `.cargo-checksum.json` SHA256：`557c31564e646c5991db9e09fdac15502ca5e9577d822d1ff7483b5442c7131a`
- 原 manifest 许可表达式：`Apache-2.0 OR MIT`

官方 archive 全部 23 个文件及匹配 vendor 的校验元数据均完整保留。源文件、原 manifest、原 lock（如有）、测试、示例、源码通知和完整许可证逐文件 SHA256 一致；未修补或裁剪。目录源校验元数据为既有已验证 vendor 的附加文件，不声称它原本包含在 .crate 中；其 package 值等于官方归档 SHA256，files 表与原版 archive 文件逐项匹配。原 registry 布局为 `/usr/share/cargo/registry/autocfg-1.5.1`。

完整许可证及通知同时留在源码树及包文档目录：

- `LICENSE-APACHE` SHA256 `a60eea817514531668d7e00765731449fe14d059d3249e0bc93b36de45f759f2`
- `LICENSE-MIT` SHA256 `27995d58ad5c1145c1a8cd86244ce844886958a35eb2b78c6b772748669999ac`

## 真实 Debian 对照与明确的本地命名偏差

真实常用对应包为 `librust-autocfg-dev`，Debian 源包 `rust-autocfg`。直接读取缓存的实际 Debian 包装归档 `/tmp/clash-verge-rev-v2.5.8-20261101/packaging-references/rust-crates-main/rust-autocfg_1.5.1-1.debian.tar.xz`，SHA256 `9ed275adc6f384eac7d9190474721fb7824e4d108c6da4cf575a35017d45a3be`。其中 debian/control 与 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/references/rust-autocfg.control` 原字节一致，control SHA256 `5856193de1059d751f17a52dbfbd82ca91fd3fadda6364ee0df47e2bb30c1f75`。发行版原归档可按 `https://deb.debian.org/debian/pool/main/r/rust-autocfg/rust-autocfg_1.5.1-1.debian.tar.xz` 核对；本次没有重新联网。

本地真实二进制名 `librust-autocfg-1.5.1-dev` 使用完整 crate 版本，以同时保留多个冻结 lock 所需的精确版本。这是根审查明确批准的本地 coinstall 命名偏差，**不声称 Debian/Ubuntu 发布这个精确名字**。沿用原 crate-version registry 路径，每个路径仅有一个所有者。参考包 Architecture `any` / Multi-Arch `same`，本包改为 all/foreign，因为负载仅为可移植原源码和文档；没有 ELF、.o、.a、.rlib、.rmeta、SONAME 或可执行工具。

未声明 Provides、Conflicts、Breaks、Replaces 或 Depends；也未声明 default/可选功能的 Debian 虚拟别名。已验收的实际特性闭包不需要额外普通源码依赖；rustc/Cargo、MSRV 和链接器是消费者构建前提，不机械写成该源码包的运行 Depends。未选择的上游开发、可选及其他平台依赖仍原样保留，未声称其闭包已经交付。实际原观察特性 `[]`；消费参数 default-features=false，features=`[]`。

## 正式候选包

- Package：`librust-autocfg-1.5.1-dev`
- Version：`1.5.1-0chinaos1`
- Architecture：`all`；Multi-Arch：`foreign`
- 路径：`/home/laevatein/ChinaOSProject/binaries/librust-autocfg-1.5.1-dev/all/librust-autocfg-1.5.1-dev_v1.5.1-0chinaos1_all.deb`
- 大小：21432 bytes；SHA256：`21fca8ef8e3466e2d3af8d55ba59e6b72b29f8ae39cbe9d1aa0bb97afb0b965b`
- SHA256 sidecar：`/home/laevatein/ChinaOSProject/binaries/librust-autocfg-1.5.1-dev/all/librust-autocfg-1.5.1-dev_v1.5.1-0chinaos1_all.deb.sha256`
- control SHA256：`fc288acd911911883fe5e59651df31ad1c7eaa86839031d09af1db26bcc80df5`

```debcontrol
Package: librust-autocfg-1.5.1-dev
Source: rust-autocfg
Version: 1.5.1-0chinaos1
Architecture: all
Multi-Arch: foreign
Section: rust
Priority: optional
Maintainer: ChinaOSProject <packages@chinaosproject.local>
Installed-Size: 76
Homepage: https://github.com/cuviper/autocfg
X-Cargo-Crate: autocfg
X-Cargo-Crate-Version: 1.5.1
Description: Rust source for autocfg 1.5.1
 Unchanged official source and complete notices for offline Cargo
 directory-source consumers. The exact binary name is a local
 coinstallation choice; optional feature closure is not advertised.
```

## 实际共装和离线消费

组装、xz 压缩、独立 dpkg root 安装及编译均仅在已有 `codex-v0.166.0-v8-directory-probe-20261107` 容器完成，持共同 host flock，最多2 jobs，离线。原有128包加新33包共161个精确包实际共装成功；所有 5766 条文件的 md5、SHA256、唯一所有权和 dpkg 状态均通过。本包 29 条文件记录通过。原有五个 float16 组包保持 loongarch64，其余包括全部新包为 all；未改旧包字节或元数据。隔离安装根为 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/verification-attempt3/root`，未改共享容器 /usr 或宿主 /usr。

只把新33个已安装源码树迁到 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/verification-attempt3/unrelated-prefix/only-exact33-directory-source`。三个 cohort 各使用最初为空的 Cargo home、显式 directory-source replacement、精确原版本锁和重命名依赖别名；metadata 确认无借用原 vendor 或额外源码，21+9+3=33版本全部运行实际 API。本包位于 cohort0，别名 `dep_autocfg_1_5_1`；实际检查：Use the installed-source helper to probe an available integer type, an arithmetic expression and an absent type.

默认环境 locked/offline release build 和程序执行全部成功，RUSTC_BOOTSTRAP 未设置；没有用全局 bootstrap 放开特性。编译工具如实为 `rustc 1.99.1-dev` / LLVM23.1.1，以及 `cargo 1.99.1-dev (5f94df478 2026-08-27)`，不是官方 stable 发行。临时消费者最终 ELF machine258、flags0x3、解释器 /lib64/ld.so.1、直接 GLIBC 符号上限 2.28，容器动态依赖全部解析。cohort0 程序 SHA256 `f82171b1d6ff07dada8e400f556b53bf24e41d7c6b36e1e0cd4f83c2903cae23`。测试用 linker/LD_LIBRARY_PATH/RUNPATH 仅属该容器消费环境，未作为可移植 SDK 或本源码包负载交付。

精确版本与 rlib 的绑定见 consumer-exact-rlib-binding.json：先核对 Cargo metadata 中的重命名依赖和原版本 package ID，再核对已成功消费者 build 日志的真实 rustc --extern 路径及该 rlib 哈希。这证明编译器的精确输入归属，不声称优化后最终程序保留每一个归档成员。

本包实际生成 rlib `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/verification-attempt3/cohort-0-target/loongarch64-unknown-linux-gnu/release/build/autocfg/201e2280d2d1aaee/out/libautocfg-201e2280d2d1aaee.rlib`，SHA256 `0e3183ae1755a2195dccdb2f8ab8662acad7808a2d9e05eb824cfa2fa5c4092b`，ELF 成员标志 `{"0x43": 6}`；全部33个 rlib 的实测汇总 `{"0x43": 104}`。未把对象0x43改称0x3。最终可执行程序通过不等于 Rust 编译器、标准库、源码或对象 ABI 验收通过；既有 SDK review candidates 和未解决源码门禁保持原状态，未重建编译器/标准库/工具。

## 保留的失败和验证边界

保留三次失败记录：attempt1 在实际安装成功后错误假设全部旧包 Architecture=all，已改为按索引核对五个原有 loongarch64 包；attempt2 对这五个 Multi-Arch:same 包错误假设未带架构后缀的 md5sums 文件名，已改为精确选择原有架构限定文件。两次均只修正任务验证脚本，包字节未变。attempt3 的 cohort0 默认 build/run/ABI 全部成功，额外 RUSTC_BOOTSTRAP=-1 check 在原版 anyhow 1.0.102 失败 E0554：build.rs 的 compile_probe(false) 显式移除该变量，dev 编译器 probe 接受不稳定功能，随后受限真实编译拒绝它。根审查明确将此人工限制配置与默认验收分列；未改源代码、manifest、预期诊断或编译器策略。attempt4 沿用 cohort0 已成功日志而不重复其 build/run，补齐 cohort1/2；所有额外 -1 结果均如实单列。规范交付首轮仅在预检阶段失败：脚本按 crate 名匹配 rlib，无法区分同一 cohort 的多个 base64 版本；尚未复制任何正式文件。原脚本和失败日志完整保留。独立精确绑定将 Cargo metadata 的 alias/原版本 package ID 与已通过 build 日志的实际 rustc --extern 路径逐项核对，33 条均唯一；修正只采用该已验证路径，不任意选取、不修改消费者结果、不重编译或重跑。

额外限制模式汇总：cohort0: exit101; cohort1: exit101; cohort2: exit0。它没有全部通过；不从 dev 编译器结果推断 stable channel 行为。该组边界适用于本报告，无论本包是否自身包含 anyhow。未运行上游全量测试或未选择平台/可选特性。attempt1-3 未显式指定 TMPDIR；包根、targets 均已在任务 ext4 缓存。attempt4 明确设置 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/temporary` 对应容器 /cargo-home 路径为 TMPDIR。此前失败原日志均保留，不掩盖为成功。

证据统一位于 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112`：packages-index.json 保存全部源文件及归档/control/许可证 SHA256；package-file-verification-attempt4.json 保存实际161包文件验证；consumer-result.json 保存默认 API/锁/日志和独立 -1 结果；consumer-rlib-object-flags-attempt4.json 保存实测对象标志；deliverables-manifest.json 和 reports-index.json 保存正式候选及报告路径与哈希。validation-harness-correction*.json 和 restrictive-policy-classification-and-resume.json 记录脚本修正。所有受保护 compiler/tool/source/lock 及原128 canonical archives 的最终哈希重新核对一致。

源码无补丁。未修改原 source/cache 输入、旧包、共享索引；无宿主构建/安装、新容器、网络、GUI、Magpie 执行/信号、Herdr、commit/push。Rust LLM 政策对 Rust 项目贡献文本/诊断/源码注释仍适用；本记录仅是按独立授权在仓库外生成的普通依赖交付证据。
