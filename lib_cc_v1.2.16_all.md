# cc 1.2.16 源码开发包交付记录

日期：2026-11-12。被处理的上游仓库：https://github.com/rust-lang/cc-rs。本项没有上游源码补丁、commit 或 push。

本包是可复用的 Rust crate 源码开发包，供 Cargo 目录源码使用，不是独立 C SDK。正式候选 `librust-cc-dev` 的 Debian 版本为 `1.2.16-1`，`Architecture: all`、`Multi-Arch: foreign`。包内源代码位于 `/usr/share/cargo/registry/cc-1.2.16`，包含原版源码、原版 MIT/Apache-2.0 完整许可证和 Cargo 校验元数据。没有交付本次构建的 `.a`、`.rlib` 或 LoongArch 可执行程序。

产物：`/home/laevatein/ChinaOSProject/binaries/librust-cc-dev/all/librust-cc-dev_v1.2.16-1_all.deb`

字节数：`90984`

SHA256：`6df66f92595e4ef6e0bc231954bc785afa541005ffce027f93885437da424d2e`。同目录 `.sha256` sidecar 已回读核验。

原始官方 crate SHA256：`be714c154be609ec7f5dad223a33bf1482fff90472de28f7362806e6d4832b8c`。独立重新验证原始 `.crate`、官方锁/index 校验值，以及 vendor 中 35 个原始文件逐字节及逐文件 SHA256 一致。原版 Cargo.toml、build.rs、源码和许可证没有修改。`.cargo-checksum.json` 保留实际官方包哈希及完整文件哈希表；相较 Debian 经补丁源码常见的空 `files` 表，本包采用更严格的原始逐文件验证。

Debian 打包依据为真实 `rust-cc_1.2.60-1` 的 `debian/control`、`debian/rules`、`debian/debcargo.toml`、`debian/cargo-checksum.json`、版权与测试控制文件。其规则委托 `dh --buildsystem cargo`，没有独立 `.install` override。包名、源码开发包归属、feature/版本虚拟包名称及依赖命名按这些资料处理。参考源码中的 `Architecture: any`/`Multi-Arch: same` 改为本任务明确要求的可移植源码 `all`/`foreign`；本次不编译源包内的目标代码以充作架构相关载荷。

实际交付版本为官方锁中已经验证的 cc 1.2.16，与参考的 1.2.60 有版本差异。1.2.16 默认依赖只有 shlex `>=1.3.0,<2`，不含后续版本的 find-msvc-tools，因此不人为新增它。默认 feature 对应 source `Depends` 已声明。上游 `parallel` feature 仍完整保留在未改动 Cargo.toml；它需要额外的 jobserver `>=0.1.30` 和 Unix libc `>=0.2.62` 源码，包中将其列为 `Suggests`。与参考的 collapse_features 方式不同，本次仅声明默认 feature 对应的 Provides，不声称提供 `+parallel` 或 `+jobserver` 虚拟包，也不宣称这些可选功能已验证。使用者可另外提供所需源码后启用可选 feature。此差异经主 Agent 确认，用于保持 PSM 的最小真实依赖闭包。

三个包 `librust-psm-dev 0.1.26-1`、`librust-cc-dev 1.2.16-1`、`librust-shlex-dev 1.3.0-1` 在既有专用容器内、共同 host flock 下独立安装到任务目录的 dpkg root，整组配置成功且依赖由版本化 Provides 实际满足。全量包内 md5、原始文件哈希、唯一文件归属和源码 checksum 都通过检查。未向宿主或容器共享 `/usr` 安装这些包。

验证从这些已安装包重新复制到另一个目录前缀的 Cargo directory source，仅配置这三个 crate，不借用完整 vendor 目录，离线生成独立消费工程锁，然后 `cargo build --locked --offline --release --target loongarch64-unknown-linux-gnu --jobs 2`。实际图为 PSM(default)→cc(default)→shlex(default,std)。消费程序使用真实 PSM API 检查非零/16 字节对齐栈指针、下降方向、8192 字节局部栈帧以及返回后的栈指针恢复，输出 `PSM direction=Descending; aligned pointers; nested stack delta=8256; restored=true`。

使用已接受且不可变的 Rust 1.99.1-dev compiler/std 快照，先以未设置 RUSTC_BOOTSTRAP/无不稳定 flag 的通常配置通过，再以 `RUSTC_BOOTSTRAP=-1` 的限制模式重新编译并执行同一消费程序；该值禁止不稳定 feature，不开放 bootstrap 权限。独立负对照 `#![feature(rustc_private)]` 确实以 E0554 被拒绝。compiler 仍标识 dev，并未据此宣称它是官方 stable 渠道。没有重新构建编译器或标准库，相关输入和消费工程锁的前后哈希不变。

最终消费程序 ELF machine=258、flags=0x3、解释器 `/lib64/ld.so.1`、直接 GLIBC 最高 2.28、动态依赖全部解析。其生成的 libpsm_s.a 的真实成员也为 machine258/flags0x3，仅作为验证证据保留，未装入源码包。构建、压包、安装与消费验证均在已有 `codex-v0.166.0-v8-directory-probe-20261107` 容器，持共同锁、jobs2、离线执行；有限的 Debian 参考资料下载位于锁外。

保留的初次环境/检查问题：第一次独立 dpkg root 缺 updates 目录，dpkg 在解包前拒绝；新独立 root 补齐标准数据库目录后通过，包字节未变。限制模式的负对照最初过度匹配 "stable release channel" 文本，实际 compiler 保持 dev 名称但正确返回 E0554；检查改为验证拒绝语义，首次日志保留。未掩盖编译/链接错误，也没有重跑任何不相关全套测试。

证据目录：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/psm-source-dev-packaging-20261112`。关键证据为 `deliverables-manifest.json`、`packages-index.json`（完整 control）、`payload-index.json`、`package-file-verification.json`、`official-input-verified.json`、`references/index.json`、`consumer-result.json`、`stable-policy-result.json` 和各 runner/log；所有旧失败输入、日志及先前六个 native C SDK 交付都保留。构建输入四个官方锁的 SHA256 与先前接受值一致。未操作 Magpie 可执行文件、进程、GUI、服务或会话；未新建容器、未执行 Herdr 操作、未提交或推送。主 Agent 负责最终审核发布。
