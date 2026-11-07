# Rust 1.99.0 oldworld SDK：Debian 分包交付

本项将 Loongnix 发布的 Rust 1.99.0 ABI 1.0 SDK 拆为八个经过安装和使用验证的 Debian 包，版本均为 1.99.0-2。没有重新编译 Rust，也没有修改 Rust、LLVM 或 Cargo 源码。此前 1.99.0-1 单体候选已由本组替代，不作为最终交付。

上游源码仓库：[rust-lang/rust](https://github.com/rust-lang/rust)。该 SDK 最初用于 Clash Verge Rev v2.5.8 / service 的构建准备，随后用于 Codex CLI v0.166.0 的构建环境。Clash v2.5.10 要求 Rust 1.99.1，本组 1.99.0 不满足该新要求。

## 输入与版本

- 发布归档：[Loongnix Rust 1.99.0 ABI 1.0 SDK](https://ftp.loongnix.cn/toolchain/rust/rust-1.99/2026-10-09/abi1.0/rust-1.99.0-loongarch64-unknown-linux-gnu.tar.xz)。
- tar.xz SHA256：127a4130c0462e524659590b32ba79e705133b23c9dc978ae91276f653eb9dcc；已与发布方同目录 .sha256 核对。
- rustc 1.99.0 (a4d5d3f0e 2026-10-09)，完整 commit a4d5d3f0ef546687c1cf33f518c87086cab05629，host loongarch64-unknown-linux-gnu，LLVM 22.1.8。
- Cargo 1.99.0 (fa274c525 2026-10-09)、rustfmt 1.10.0-stable、Clippy 0.1.99。
- 原 SDK 组件为 rustc、cargo、rust-std-loongarch64-unknown-linux-gnu、rustfmt-preview、clippy-preview，含 rustdoc、LLD 和 objcopy 等随附工具。

分包使用已核验旧单体 deb 作只读输入，其 SHA256 为 748e734015819d761e48d5995d601350449f3b9afd0974757fd9db82dda9f940。此值是旧 deb 的哈希，不是上面的发布归档哈希；旧输入已留作备份。

## Debian 参考、依赖和文件归属

参考 [Debian rustc 源包](https://sources.debian.org/src/rustc/) 1.98.1+dfsg1-1~exp1 的 control、rules、安装清单及 copyright，读取副本位于任务目录 packaging-references/rustc-1.98.1/debian/。采用编译器、Cargo、版本化运行库、开发包、格式化器、Clippy、LLVM 辅助工具和 GDB 脚本的分包方式。原生包使用本机 dpkg 实测的 loongarch64，纯脚本 GDB 包为 all。

| 包 | 实际内容 |
| --- | --- |
| rustc | rustc、rustdoc、proc-macro 服务、编译器 man pages 和 target schema |
| cargo | Cargo、man pages、Bash/Zsh 补全 |
| libstd-rust-1.99 | 动态标准库、librustc_driver 和 SDK 自带的编译器运行库/过程宏 |
| libstd-rust-dev | target 的 rlib/rmeta、self-contained 链接输入及动态标准库链接 |
| rustfmt | rustfmt 和 cargo-fmt |
| rust-clippy | clippy-driver 和 cargo-clippy |
| rust-llvm | SDK 附带的 LLD、objcopy 等工具，不冒充完整通用 LLVM SDK |
| rust-gdb | rust-gdb、rust-gdbgui 和 Python pretty-printer 脚本 |

开发包精确依赖同版本运行库，rustc 精确依赖开发包，Cargo 和 GDB 包精确依赖 rustc。原生运行依赖按 ELF 核对为 libc6 (>= 2.28)、libgcc1 (>= 1:4.2)。rustc 另依赖 GCC、libc 开发文件、binutils 以支持正常编译链接；GDB 包依赖 gdb、python3，未捆绑调试器。迁出文件的包声明 Breaks/Replaces: rustc (<< 1.99.0-2)，处理旧单体的文件所有权。

按 SDK 实际输入分包，不生成不存在的 Rust 源码、HTML 文档、rust-analyzer CLI、其他架构标准库或空元包。动态标准库移至运行库目录并保留 target 链接；补全、target schema 按 Debian 路径放置。上游安装器的 manifest/uninstall 记账文件不发行，由 dpkg 管理安装和卸载。各包独立提供 copyright 和适用的完整许可文本，包级检查通过。构建 registry 镜像配置未写入交付包。

## 最终交付

下列八包均为 1.99.0-2；路径采用 binaries/包名/Architecture/包名_v1.99.0_Architecture.deb。主 Agent 已重新核对正式文件大小与 SHA256。

| 包 | Architecture | 字节 | SHA256 |
| --- | --- | ---: | --- |
| rustc | loongarch64 | 5,646,776 | 95cb982fda84c32ac772e77676ed7ce1918f0c560bf3d5a42cc02cb9db76bc8d |
| cargo | loongarch64 | 12,432,776 | 12fd048cda2462426399d5df82b34be33d5b6275a566d089291eeca4f9e23544 |
| libstd-rust-1.99 | loongarch64 | 85,665,836 | ee9ac06a2642f34c939738298916293ebe79df2b9088401fc44ffd0b27fce210 |
| libstd-rust-dev | loongarch64 | 30,350,456 | 1bdd055a7c5fb4418c8fee4d9ad2effd64bff8270967a76124e247f56a517faf |
| rustfmt | loongarch64 | 3,086,512 | 322fc9e256fd104280ae7915a0ee36f960c6c49c3e0c9a22f9755e827731cef9 |
| rust-clippy | loongarch64 | 3,976,076 | b40c40d0bfb8f4618084d082e977dde9b1b7d764a37749c906f5a3f767e1657e |
| rust-llvm | loongarch64 | 40,873,864 | dad0f195bb0d426cfbd17b9fe4ba2eae621eca431d8bd74c1d2e9efffb82e08d |
| rust-gdb | all | 427,184 | 307f0ec9ec7ffde752fda4e02a5751068228508b15e30d432c5d7737b053843c |

## 隔离、ABI 和验证

初始 SDK 封包、后续分包、安装与编译验证均在专属 Podman VFS 环境完成，基座 Debian Buster / glibc 2.28，分包验证容器 rust199-split-main-build。没有向宿主安装 SDK 或构建依赖。该验证容器和重复 staging 已在缓存维护中回收；冻结输出、脚本、输入备份及日志均保留。

- 34 个独立 ELF 均为 LoongArch machine 258、flags 0x3，动态入口解释器 /lib64/ld.so.1，最高 GLIBC 要求 2.28，无 GLIBCXX 运行库要求。原始扫描的 370 个 rlib ELF 成员 ABI 一致；分包后 27 个 rlib 的 SHA256 未变。
- 整组安装及从旧 rustc 1.99.0-1 单体升级至八包 1.99.0-2 成功。136 个上游 payload 文件逐项核对，重复文件所有权为 0，许可文本完整性通过。
- 安装后的 rustc、Cargo、rustfmt、Clippy、LLD 和 objcopy 版本入口成功；Cargo 构建运行独立项目、Clippy、格式化器和 rustdoc 文档生成通过。动态标准库示例的 loader 解析成功。
- Rust 2024 线程/Arc/Mutex 示例在容器和真实宿主均成功；宿主只运行容器编译的程序，没有在宿主编译。
- rust-gdb 导入 pretty-printer 成功，并实际在 main 断点停止、列出程序线程。

rust-lldb 不在最终八包内：基座 LLDB 13 嵌入 Python 3.7，发布方 helper 使用较新的赋值表达式，真实导入返回 SyntaxError。失败保留在安装 smoke 日志，未忽略错误或以版本检查代替实际使用验证。

未执行 Rust 编译器/标准库完整套件；SDK smoke 不代表 Codex 或 Clash 应用已通过编译测试。

## 复核资料

任务根目录 /tmp/clash-verge-rev-v2.5.8-20261101：

- 初始归档、ELF 和宿主示例：logs/rust199-package-elf.json、rust199-shlibdeps.log、rust199-package-smoke.log、rust199-host-smoke.log。
- 分包映射、脚本：rust-sdk-split-main/manifest.json 及同目录辅助脚本。
- 最终八包清单：rust-sdk-split-main/logs/accepted-family-delivery-20261105.json。
- 安装及 smoke：同目录 logs/family-apt-install.log、installed-smoke-20261105.log、core-installed-smoke-20261105.log。
- 单体升级和逐项核对：同目录 logs/core-family-upgrade-20261105.log、core-family-verification-20261105.json。

正式包和报告由主 Agent 审核发布。SDK、Cargo 缓存、应用源码没有在本项中打补丁。
