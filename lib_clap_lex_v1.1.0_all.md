# clap_lex 1.1.0 普通源码依赖包记录

日期：2026-11-13（实际消费者验收：2026-11-12）。交付原样的普通第三方 Rust crate 源码开发包；不含新编译对象或 SDK。主 Agent 独立审核和发布。

## 上游、来源及完整字节

- GitHub 项目：`https://github.com/clap-rs/clap`
- crate/version：`clap_lex 1.1.0`
- 官方 crate 归档：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/tools-official-inputs-20261112/archives/clap_lex-1.1.0.crate`
- 官方归档 SHA256：`c8d4a3bb8b1e0c1050499d1815f5ab16d04f0959b233085fb31653fbfc9d98f9`
- 原版 Cargo.toml SHA256：`01d043b9d8911420d51b1b21313278cbcdac10cd852e277b25260a458f5f8fee`
- 既有目录源 `.cargo-checksum.json` SHA256：`4a7847f3c2200c27f324afbff9a730d27799169ae208b6280f6c0a6515f0d889`
- 原 manifest 许可表达式：`MIT OR Apache-2.0`

官方 archive 全部 9 个文件及匹配 vendor 的校验元数据均完整保留。源文件、原 manifest、原 lock（如有）、测试、示例、源码通知和完整许可证逐文件 SHA256 一致；未修补或裁剪。目录源校验元数据为既有已验证 vendor 的附加文件，不声称它原本包含在 .crate 中；其 package 值等于官方归档 SHA256，files 表与原版 archive 文件逐项匹配。原 registry 布局为 `/usr/share/cargo/registry/clap_lex-1.1.0`。

完整许可证及通知同时留在源码树及包文档目录：

- `LICENSE-APACHE` SHA256 `c6596eb7be8581c18be736c846fb9173b69eccf6ef94c5135893ec56bd92ba08`
- `LICENSE-MIT` SHA256 `6efb0476a1cc085077ed49357026d8c173bf33017278ef440f222fb9cbcb66e6`

## 真实 Debian 对照与明确的本地命名偏差

真实常用对应包为 `librust-clap-lex-dev`，Debian 源包 `rust-clap-lex`。直接读取缓存的实际 Debian 包装归档 `/tmp/clash-verge-rev-v2.5.8-20261101/packaging-references/rust-crates-main/rust-clap-lex_1.1.1-1.debian.tar.xz`，SHA256 `910951b09bb8017b6fe13011d40de268b91b9929559370afcfd0d041a96305e0`。其中 debian/control 与 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/references/rust-clap-lex.control` 原字节一致，control SHA256 `e2dbf812bd5271b3c8be0f28cddc4b4ea3a53c10cb2e532e93f2c2e8e4a9bc90`。发行版原归档可按 `https://deb.debian.org/debian/pool/main/r/rust-clap-lex/rust-clap-lex_1.1.1-1.debian.tar.xz` 核对；本次没有重新联网。

本地真实二进制名 `librust-clap-lex-1.1.0-dev` 使用完整 crate 版本，以同时保留多个冻结 lock 所需的精确版本。这是根审查明确批准的本地 coinstall 命名偏差，**不声称 Debian/Ubuntu 发布这个精确名字**。沿用原 crate-version registry 路径，每个路径仅有一个所有者。参考包 Architecture `any` / Multi-Arch `same`，本包改为 all/foreign，因为负载仅为可移植原源码和文档；没有 ELF、.o、.a、.rlib、.rmeta、SONAME 或可执行工具。

未声明 Provides、Conflicts、Breaks、Replaces 或 Depends；也未声明 default/可选功能的 Debian 虚拟别名。已验收的实际特性闭包不需要额外普通源码依赖；rustc/Cargo、MSRV 和链接器是消费者构建前提，不机械写成该源码包的运行 Depends。未选择的上游开发、可选及其他平台依赖仍原样保留，未声称其闭包已经交付。实际原观察特性 `[]`；消费参数 default-features=false，features=`[]`。

## 正式候选包

- Package：`librust-clap-lex-1.1.0-dev`
- Version：`1.1.0-0chinaos1`
- Architecture：`all`；Multi-Arch：`foreign`
- 路径：`/home/laevatein/ChinaOSProject/binaries/librust-clap-lex-1.1.0-dev/all/librust-clap-lex-1.1.0-dev_v1.1.0-0chinaos1_all.deb`
- 大小：15528 bytes；SHA256：`899c9767aae8d5f891b3d2926275616e1d482d85d395854af914bec17cab7126`
- SHA256 sidecar：`/home/laevatein/ChinaOSProject/binaries/librust-clap-lex-1.1.0-dev/all/librust-clap-lex-1.1.0-dev_v1.1.0-0chinaos1_all.deb.sha256`
- control SHA256：`7c372896e0e820eff642f0c5668b68db669b80f4fdb7027eee8d65dfb9bf7698`

```debcontrol
Package: librust-clap-lex-1.1.0-dev
Source: rust-clap-lex
Version: 1.1.0-0chinaos1
Architecture: all
Multi-Arch: foreign
Section: rust
Priority: optional
Maintainer: ChinaOSProject <packages@chinaosproject.local>
Installed-Size: 60
Homepage: https://github.com/clap-rs/clap
X-Cargo-Crate: clap_lex
X-Cargo-Crate-Version: 1.1.0
Description: Rust source for clap_lex 1.1.0
 Unchanged official source and complete notices for offline Cargo
 directory-source consumers. The exact binary name is a local
 coinstallation choice; optional feature closure is not advertised.
```

## 实际共装和离线消费

组装、xz 压缩、独立 dpkg root 安装及编译均仅在已有 `codex-v0.166.0-v8-directory-probe-20261107` 容器完成，持共同 host flock，最多2 jobs，离线。原有128包加新33包共161个精确包实际共装成功；所有 5766 条文件的 md5、SHA256、唯一所有权和 dpkg 状态均通过。本包 15 条文件记录通过。原有五个 float16 组包保持 loongarch64，其余包括全部新包为 all；未改旧包字节或元数据。隔离安装根为 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/verification-attempt3/root`，未改共享容器 /usr 或宿主 /usr。

只把新33个已安装源码树迁到 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/verification-attempt3/unrelated-prefix/only-exact33-directory-source`。三个 cohort 各使用最初为空的 Cargo home、显式 directory-source replacement、精确原版本锁和重命名依赖别名；metadata 确认无借用原 vendor 或额外源码，21+9+3=33版本全部运行实际 API。本包位于 cohort0，别名 `dep_clap_lex_1_1_0`；实际检查：Parse a real long option with its value and a positional argument.

默认环境 locked/offline release build 和程序执行全部成功，RUSTC_BOOTSTRAP 未设置；没有用全局 bootstrap 放开特性。编译工具如实为 `rustc 1.99.1-dev` / LLVM23.1.1，以及 `cargo 1.99.1-dev (5f94df478 2026-08-27)`，不是官方 stable 发行。临时消费者最终 ELF machine258、flags0x3、解释器 /lib64/ld.so.1、直接 GLIBC 符号上限 2.28，容器动态依赖全部解析。cohort0 程序 SHA256 `f82171b1d6ff07dada8e400f556b53bf24e41d7c6b36e1e0cd4f83c2903cae23`。测试用 linker/LD_LIBRARY_PATH/RUNPATH 仅属该容器消费环境，未作为可移植 SDK 或本源码包负载交付。

精确版本与 rlib 的绑定见 consumer-exact-rlib-binding.json：先核对 Cargo metadata 中的重命名依赖和原版本 package ID，再核对已成功消费者 build 日志的真实 rustc --extern 路径及该 rlib 哈希。这证明编译器的精确输入归属，不声称优化后最终程序保留每一个归档成员。

本包实际生成 rlib `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/verification-attempt3/cohort-0-target/loongarch64-unknown-linux-gnu/release/build/clap_lex/899a3b8e6f791d76/out/libclap_lex-899a3b8e6f791d76.rlib`，SHA256 `7a76ffafeeafc8f12e49ed05c570a912cb89e1b81daa6dca234504f48224c38a`，ELF 成员标志 `{"0x43": 3}`；全部33个 rlib 的实测汇总 `{"0x43": 104}`。未把对象0x43改称0x3。最终可执行程序通过不等于 Rust 编译器、标准库、源码或对象 ABI 验收通过；既有 SDK review candidates 和未解决源码门禁保持原状态，未重建编译器/标准库/工具。

## 保留的失败和验证边界

保留三次失败记录：attempt1 在实际安装成功后错误假设全部旧包 Architecture=all，已改为按索引核对五个原有 loongarch64 包；attempt2 对这五个 Multi-Arch:same 包错误假设未带架构后缀的 md5sums 文件名，已改为精确选择原有架构限定文件。两次均只修正任务验证脚本，包字节未变。attempt3 的 cohort0 默认 build/run/ABI 全部成功，额外 RUSTC_BOOTSTRAP=-1 check 在原版 anyhow 1.0.102 失败 E0554：build.rs 的 compile_probe(false) 显式移除该变量，dev 编译器 probe 接受不稳定功能，随后受限真实编译拒绝它。根审查明确将此人工限制配置与默认验收分列；未改源代码、manifest、预期诊断或编译器策略。attempt4 沿用 cohort0 已成功日志而不重复其 build/run，补齐 cohort1/2；所有额外 -1 结果均如实单列。规范交付首轮仅在预检阶段失败：脚本按 crate 名匹配 rlib，无法区分同一 cohort 的多个 base64 版本；尚未复制任何正式文件。原脚本和失败日志完整保留。独立精确绑定将 Cargo metadata 的 alias/原版本 package ID 与已通过 build 日志的实际 rustc --extern 路径逐项核对，33 条均唯一；修正只采用该已验证路径，不任意选取、不修改消费者结果、不重编译或重跑。

额外限制模式汇总：cohort0: exit101; cohort1: exit101; cohort2: exit0。它没有全部通过；不从 dev 编译器结果推断 stable channel 行为。该组边界适用于本报告，无论本包是否自身包含 anyhow。未运行上游全量测试或未选择平台/可选特性。attempt1-3 未显式指定 TMPDIR；包根、targets 均已在任务 ext4 缓存。attempt4 明确设置 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112/temporary` 对应容器 /cargo-home 路径为 TMPDIR。此前失败原日志均保留，不掩盖为成功。

证据统一位于 `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/source-dev-exact-leaf-tranche-20261112`：packages-index.json 保存全部源文件及归档/control/许可证 SHA256；package-file-verification-attempt4.json 保存实际161包文件验证；consumer-result.json 保存默认 API/锁/日志和独立 -1 结果；consumer-rlib-object-flags-attempt4.json 保存实测对象标志；deliverables-manifest.json 和 reports-index.json 保存正式候选及报告路径与哈希。validation-harness-correction*.json 和 restrictive-policy-classification-and-resume.json 记录脚本修正。所有受保护 compiler/tool/source/lock 及原128 canonical archives 的最终哈希重新核对一致。

源码无补丁。未修改原 source/cache 输入、旧包、共享索引；无宿主构建/安装、新容器、网络、GUI、Magpie 执行/信号、Herdr、commit/push。Rust LLM 政策对 Rust 项目贡献文本/诊断/源码注释仍适用；本记录仅是按独立授权在仓库外生成的普通依赖交付证据。
