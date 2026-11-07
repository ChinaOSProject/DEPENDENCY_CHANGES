# Herdr 0.9.5 LoongArch64 适配报告

状态：源码和原生测试完成；v0.9.5 release 构建、分包及包级安装/ABI/入口验证完成。

## 来源与分支

- 上游：https://github.com/herdrdev/herdr ，正式版本 [v0.9.5](https://github.com/herdrdev/herdr/releases/tag/v0.9.5)，提交 `2ecf19a92fc3a7fff75b6ce3655ef04f4625ec01`。
- 适配仓库：https://github.com/ChinaOSProject/herdr 。旧 loong64 为 `d4db08e0577dc8aa645c713b64d19e28e056aa13`，基于 0.9.1。
- 正式上游版本经 `debian10` 传播至 `loong64`；同步提交 `eda7c487b645d9637223d2a98b5410d7a1f6877d`。
- 通用启动修复先提交到 `debian10`：`5b99272554457d1d9e29d1dce17fb2abb365f645`，再由合并提交 `e7d90c0fa3` 进入 `loong64`。
- LoongArch 架构识别提交及最终源码：`ef1f4f159923fd9271391d5fc483b6c2bf806220`。相对官方 v0.9.5 仅六个文件有必要修改，工作区干净。

## 必要适配

1. 在迁移后的 `crates/ghostty-vt/build.rs` 保留 LoongArch legacy Linux 5.4.18 / glibc 2.28 target 映射，复用已适配的 Zig 0.16.0 C ABI 工具链；未新增依赖源码补丁。
2. 远程发现和自更新平台识别使用真实 `loongarch64`。远端设置可复用兼容的本地程序或显式指定构建；不存在官方 LoongArch 资产时明确返回错误，不伪装成 x86_64/ARM，也不请求不存在的官方包。
3. 修复启动期间的信号窗口：旧代码先开放 JSON API，再加载配置和恢复会话，直到主循环才安装退出信号。实际 strace 证明 API 已响应时 SIGTERM 仍是默认处理，因此进程直接被终止且未记录正常退出原因。现将监听和原有事件通道提前，在正常启动与 live handoff 中都先注册，再开放 API/恢复状态；保留运行期事件唤醒。原有 SIGHUP 生存和 SIGTERM 日志回归通过，并补充 SIGINT 检查。未改变协议、状态格式或跨版本 endpoint 合约。

## 隔离与缓存

所有编译、打包及容器测试使用项目专用 Podman 镜像，未在宿主全局安装任何构建工具或库。最终构建环境为 `localhost/herdr-build:0.9.5-buster-loongarch64-v0-tests`，image ID `d8821fc6fafc08efdf41c62ab3385e37b43acccfa9e3a24ed2d267fe1a85569d`，容器 `herdr-v095-validation-20261029`。

基座 glibc 2.28；Rust 1.98.1 ABI v0，Zig 0.16.0，LLVM/LLD 21。只在专属容器内补齐 Git 2.45.4、gawk、SSH 客户端、strace 等测试工具。Python init 回收测试子进程，nextest 保持测试进程隔离。

下载、Rust target 和工具缓存使用外盘 `/media/laevatein/VCLIB/ChinaOSProject-cache/herdr/v0.9.5-20261029/`。NTFS 上 RENAME_NOREPLACE 实测返回 EINVAL，因此仅 Zig 临时工作缓存放在 `/tmp/herdr-v095-20261029/zig-local` ext4；不修改 Zig 源码。精确权限的 Debian 暂存亦使用 ext4。未格式化或修改外盘既有数据。

## 验证与边界

- 最终源码 `cargo fmt --check`、`git diff --check` 通过。
- 官方 nextest 完整 Rust 测试：4033 通过，14 项按上游默认规则跳过（包括手动性能测试及需要专用环境的测试）；无失败。
- Python 维护合约：150 通过；UI 热路径架构检查：6 通过。
- 独立回归：SIGTERM/SIGINT 两个退出信号测试通过，各自同时验证 SIGHUP 后 server 保持运行和退出原因记录。
- 首轮测试的旧 Git 问题已修复：Git 2.20 不支持测试用 GIT_CONFIG_COUNT，离线 URL 重定向未生效；换用容器专用 Git 2.45.4 后相关 Rust 及 Python 测试通过。原始失败日志保留，没有吞掉失败。
- `just ci-tests all()` 的 Rust 与 Python 阶段通过，随后 JavaScript 维护阶段因容器没有 Bun 返回 127；该完整配方不宣称全通过。集成资产和文档的 Bun 检查亦未完成。此适配没有修改 JavaScript/TypeScript、集成脚本、文档、发布工作流或协议，采用完整原生 Rust、Python 合约、运行和 ABI 验证作为本机交付依据。
- `just lint` 的格式阶段通过，Clippy 阶段因 ABI v0 Rust SDK 未附带 Clippy 返回 101。缓存中另一份 Clippy 为 ELF flags 0x43 的新 ABI 构建，不满足本项目旧 ABI SDK 的要求；Clippy 检查未完成。
- 未执行 Windows SDK 交叉编译。本次平台相关行为通过现有 platform wrapper 调用，未新增 Windows API 或变动目标平台依赖；不宣称已实测 Windows/macOS。

原始日志和项目临时辅助文件位于 `/tmp/herdr-v095-20261029/`，均未放进源码提交。关键日志：`logs/main-ci-tests.log`、`logs/main-ui-architecture.log`、`logs/main-startup-signals.log`、`logs/main-final-release-package.log`、`logs/main-signal-trace.*`。

## 交付

- Debian 包：`/home/laevatein/ChinaOSProject/binaries/herdr/loongarch64/herdr_v0.9.5_loongarch64.deb`，8,202,572 字节，SHA256 `617f40d902d0d27c5642f578f865d58ef500fcdb9a6371b78387186b279eebf0`；同目录提供 `.deb.sha256`。
- Control 字段：`Package: herdr`、`Version: 0.9.5-1`、`Architecture: loongarch64`、`Depends: libc6 (>= 2.28), libgcc1`。
- release 二进制：45,113,224 字节，SHA256 `7cdcca87c27e2d543f00f40472f2946bbcff7d7b2bc80380fca375519d2b1e72`；来自适配提交 `ef1f4f159923fd9271391d5fc483b6c2bf806220`。包内 `usr/share/doc/herdr/build-info.json` 记录提交、工具链、ABI 与镜像来源。
- 本轮专属构建基座来源：image `c2718873d324848fc802156064aae47a42b9074812ad469805e40208e5f41090`，parentless layer `e2886cb93e194795bb4ac674257019bf35e6cb222be5669fbef8761a7c0da5dc`；只读快照归档 87,756,800 字节，SHA256 `05ff53b0e7b0fef3485c380bcbb9bcaed2f37e9859aacd86c983d5d15c175a8b`。容器内 `dpkg` 架构为 `loongarch64`，glibc 为 2.28；`/etc/shadow` 保留 gid 42、0640。
- 在专属 Buster 容器安装后，`dpkg-query` 报告 `herdr 0.9.5-1 loongarch64`，安装文件与构建二进制 SHA 相同。ELF flags 为 `0x3`（LP64），解释器为 `/lib64/ld.so.1`，最高 GLIBC 符号版本为 2.28；`ldd` 列出的动态依赖全部解析。
- 真实 CLI 入口：安装后的 `/usr/bin/herdr --version` 输出 `herdr 0.9.5`；`--help` 与 `--default-config` 均成功返回。
## Debian/Ubuntu CLI 分包与工具复核（补充，2026-11-05 至 2026-11-06）

本节只记录本轮补充的 CLI 分包和工具来源，不改写上文 2026-10-29 全套 suite 的结果及日志，也未重跑 suite、替换当前 server 或修改已交付的 Herdr deb。

### 同类 CLI 包的分包、依赖和文档做法

- Debian Bookworm 与 Ubuntu 24.04 Noble 将 Git CLI 放在架构相关的 `git` 包，将手册放在 `Architecture: all` 的同版本 `git-man` 包；主包依赖匹配版本的 `git-man`。Debian Bookworm `git`（1:2.39.5-0+deb12u3）的 Depends 包括 `libc6 (>= 2.34)`、`libcurl3-gnutls (>= 7.56.1)`、`liberror-perl`、`libexpat1 (>= 2.0.1)`、`libpcre2-8-0 (>= 10.34)`、`perl`、`zlib1g (>= 1:1.2.2)` 和版本匹配的 `git-man`。Ubuntu Noble 页面同样列出版本匹配的 `git-man` 依赖。参考：[Debian git](https://packages.debian.org/bookworm/git)、[Debian git-man](https://packages.debian.org/bookworm/all/doc/git-man)、[Ubuntu Noble git](https://packages.ubuntu.com/noble/git)、[Ubuntu Noble git-man](https://packages.ubuntu.com/noble/all/git-man)。
- Debian Trixie 的 `just`（1.40.0-1+b1）是单包 CLI，Depends 为 `libc6 (>= 2.39)` 和 `libgcc-s1 (>= 4.2)`（个别架构有专门下限）；Ubuntu Noble 的 `just`（1.21.0-1）也为单包，依赖 `libc6 (>= 2.34)` 与 `libgcc-s1 (>= 4.2)`（armhf 下限为 4.3）。两者随包提供 man page 和 copyright 文件。参考：[Debian Trixie just](https://packages.debian.org/trixie/just)、[Ubuntu Noble just](https://packages.ubuntu.com/noble/just)。本次 `just 1.43.0` 针对 glibc 2.28 构建，按 Loongnix 旧 ABI 的包名使用 `libgcc1`；man page 由缓存二进制的 `just --man` 生成。
- cargo-nextest 是本任务实际使用且可复用的 Cargo 子命令 runner，因此交付独立架构包。上游 v0.9.100 的 `rust-version = 1.85` 是编译 nextest 的 MSRV，不代表 runner 执行测试所需的 Rust 版本；上游稳定性说明记载 Rust 1.41 已知可运行测试，但旧版本不持续测试。因此 Depends 明确列出 `cargo (>= 1.41), rustc (>= 1.41)`，并依赖 libc6 (>= 2.28)、libgcc1。全新容器用 apt 安装 1.99.0-2 split 组后，单独请求 cargo-nextest 时 apt 自动拉入独立 `cargo` 包；`dpkg-query -S /usr/bin/cargo` 确认其软件包归属，最小 runner smoke 在 Rust 1.99.0-2 下通过。上游没有 man page，不创建空文档包；包内保留 README、完整 Apache/MIT 许可和 copyright。参考：[v0.9.100 Cargo.toml](https://github.com/nextest-rs/nextest/blob/cargo-nextest-0.9.100/Cargo.toml)、[上游稳定性说明](https://nexte.st/docs/stability/)。
- Debian Policy §12.5 要求每个二进制包在 `/usr/share/doc/<package>/copyright` 提供许可证信息；Ubuntu 的 `dh_installdocs` 也把 `debian/copyright` 安装到包文档目录。参考：[Debian Policy 12.5](https://www.debian.org/doc/debian-policy/ch-docs.html#copyright-information)、[Ubuntu dh_installdocs](https://manpages.ubuntu.com/manpages/bionic/man1/dh_installdocs.1.html)。
- Herdr 是单一终端 CLI，帮助和默认配置由程序入口提供，没有一套需要按架构拆分的手册库；因此使用一个 `herdr` 架构包，随包带 `build-info.json`、copyright 和上游/供应商许可证文件。其 control 依赖为 `libc6 (>= 2.28), libgcc1`。
- 前一 Git 2.45.4 候选为 1:2.45.4-1，来自缓存安装二进制，启用了 NO_CURL/NO_GETTEXT，没有 HTTP/HTTPS remote helper，也没有 man pages；它仍使用普通 git 包名，安装会替换完整 Git，因此已撤回并仅保存在 /mnt/chinaos-cache/ChinaOSProject-cache/herdr/v0.9.5/supplemental-20261106/out/git_v2.45.4_loongarch64.old-candidate.deb，SHA256 e65aaf6d1d7d2f9c2aa603fd95c63952c536d53b41167aaefe5455ad593a74cc。该候选不可发布，也不构成 HTTPS 证据。
- 替代候选从未修改的 Git v2.45.4 上游源码归档重编，SHA256 93f82ed1cab772acf3167db3be2bd2e54876e0ab53c6a9af744940752b961d88；该构建未启用 NO_CURL，含 HTTP/HTTPS helper。按构建范围关闭了可选 Perl、Python 和 Tcl/Tk 集成。
- 包按 Debian/Ubuntu 与既有 Git 2.43.0 做法拆成架构相关的 git 1:2.45.4-2 和 Architecture: all 的 git-man 1:2.45.4-2。git 依赖 libc6 (>= 2.28)、libcurl4 (>= 7.56.1)、libexpat1 (>= 2.0.1)、libssl1.1 (>= 1.1.0)、zlib1g (>= 1:1.2.2)、ca-certificates 和严格匹配版本的 git-man；git-man 提供 196 个手册页并带 Breaks/Replaces: git (<< 1:2.45.4-2)。整组安装核对只有共同目录项、没有文件重叠。
- 安装后 Git 主程序和 HTTP helper 的 ABI 均为 flags 0x3、loader /lib64/ld.so.1、最高 GLIBC_2.28；动态依赖全部解析。HTTP/HTTPS ls-remote 与 clone 通过；HTTPS 对显式信任的 IP SAN fixture CA 成功，对系统 CA 不信任的自签证书按预期拒绝。

### 文档和许可证来源

- Herdr：仓库 `LICENSE`、`vendor/libghostty-vt/LICENSE`、`vendor/portable-pty/LICENSE.md`；已交付 Herdr deb 中对应 `/usr/share/doc/herdr/copyright`、`libghostty-vt-LICENSE` 和 `portable-pty-LICENSE`。
- Git 2.45.4：未修改的 [上游 v2.45.4 源码](https://github.com/git/git/tree/v2.45.4)，包括 [COPYING](https://github.com/git/git/blob/v2.45.4/COPYING)、[README.md](https://github.com/git/git/blob/v2.45.4/README.md) 和 [Documentation/RelNotes/2.45.4.txt](https://github.com/git/git/blob/v2.45.4/Documentation/RelNotes/2.45.4.txt)；源归档 SHA256 为 93f82ed1cab772acf3167db3be2bd2e54876e0ab53c6a9af744940752b961d88。git 包在 /usr/share/doc/git/ 提供上述文档与 copyright；git-man 包带 COPYING 和 copyright。
- cargo-nextest 0.9.100：上游 [cargo-nextest Cargo.toml](https://github.com/nextest-rs/nextest/blob/cargo-nextest-0.9.100/cargo-nextest/Cargo.toml) 声明 Apache-2.0 OR MIT，[workspace Cargo.toml](https://github.com/nextest-rs/nextest/blob/cargo-nextest-0.9.100/Cargo.toml) 的 `rust-version = 1.85` 是构建 MSRV；[上游稳定性说明](https://nexte.st/docs/stability/) 将运行测试的已知兼容版本记为 Rust 1.41，并注明不持续测试该旧版本。[README](https://github.com/nextest-rs/nextest/blob/cargo-nextest-0.9.100/cargo-nextest/README.md)、[Apache 2.0](https://github.com/nextest-rs/nextest/blob/cargo-nextest-0.9.100/LICENSE-APACHE) 和 [MIT](https://github.com/nextest-rs/nextest/blob/cargo-nextest-0.9.100/LICENSE-MIT) 文本均随包提供。上游 crate SHA256 74e1003eeb47ed3d42cbdecf2bdf722c332736ab85440137050d90760c897e91.
- just 1.43.0：上游 [1.43.0 Cargo.toml](https://github.com/casey/just/blob/1.43.0/Cargo.toml) 声明 `CC0-1.0`，[上游 LICENSE](https://github.com/casey/just/blob/1.43.0/LICENSE) 是 CC0 1.0 Universal；包内 `/usr/share/doc/just/copyright` 提供完整 CC0 文本。

### 工具来源与分包结论

| 工具 | 本项目使用来源 | 处理结论 |
| --- | --- | --- |
| Git 2.45.4 | 2026-10-29 suite 使用的缓存安装二进制 SHA256 为 4981102de79a425aa0d2ca2a88d06433a7d8ce1bb280f32c803527d9477c5ffd；本轮另从未修改上游源码归档 SHA256 93f82ed1cab772acf3167db3be2bd2e54876e0ab53c6a9af744940752b961d88 重编完整 HTTP/HTTPS 支持。 | 已用 Git 1:2.45.4-2 替换本项目先前无 curl 候选：[git 包](/home/laevatein/ChinaOSProject/binaries/git/loongarch64/git_v2.45.4_loongarch64.deb)，Architecture loongarch64，SHA256 71b125b42f6e3ba7b1ca6ae18b3f80352c9bf6a44289d2a9af19c4f576bda2d3；以及 [git-man 包](/home/laevatein/ChinaOSProject/binaries/git-man/all/git-man_v2.45.4_all.deb)，Architecture all，SHA256 e2bb7a9bd7a3af336505892e3e62869fd4282cf0efde5989c32eaa9d2807bf6a。前一 NO_CURL 包仅作备份，不支持 HTTP/HTTPS。 |
| just 1.43.0 | 2026-10-29 Cargo 安装收据 `/media/laevatein/VCLIB/ChinaOSProject-cache/herdr/v0.9.5-20261029/tools/.crates.toml`；缓存二进制 SHA256 `27f2ac9230b4b451074f944e3b200cb8981e546f727dcc68eaeb07c7d0f96d8b`。 | 作为 Debian/Ubuntu 已独立发行的通用命令运行器补交 [just_v1.43.0_loongarch64.deb](/home/laevatein/ChinaOSProject/binaries/just/loongarch64/just_v1.43.0_loongarch64.deb)，Architecture `loongarch64`，Version `1.43.0-1`，Depends `libc6 (>= 2.28), libgcc1`，SHA256 `f92793ca2ab8a6d9683862286c71a39b95c1553ffd0fe44380da53d299e499a4`。 |
| Rust 1.98.1 ABI v0 | 使用现有 Rust 工具链缓存 `/home/laevatein/.cache/herdr-loong64-build/tools/rustc-1.98.1-abi0`。缓存中已有 `rustc_v1.98.1_loongarch64.deb`（Version `1.98.1-1`，SHA256 `5b98197e8bb4ae454038ad319764b08e26c47c3a4f27aa67d7ea70861c3a0050`）和 `rustc_v1.98.1-2_loongarch64.deb`（SHA256 `c875e0565f04b3989d008faf283f8583d200bf3aeb140926e0ec3329f3f0e734`）。 | 本轮只读复用缓存工具链，没有新编译或重打包 Rust。Rust 1.99.0-2 split 候选由主 Agent 在公共交接中核验，本补充未触碰其候选文件。 |
| Zig 0.16.0 | 现有发行 deb `/home/laevatein/src/zig-0.16.0-container/artifacts/zig_v0.16.0_loongarch64.deb`，SHA256 `c4080c8442a0240e4e9f8855ff87a0a7da7352f03d07cc8032395fc1d157ec8e`。 | 已有包只读复用，不重打包。 |
| LLVM/LLD 21.1.8 | 现有发行 deb `/home/laevatein/src/zig-0.16.0-container/artifacts/llvm-21_v21.1.8_loongarch64.deb`，SHA256 `c292fe67bee233850b7983144cb1f6d3197c4db4d2f1d882c16ec85c66ee7be8`；包内包含 `ld.lld-21`/LLD。 | 已有 LLVM 包只读复用，LLD 不另行拆包。 |
| cargo-nextest 0.9.100 | 从已验证的只读缓存原样打包，runner 二进制 SHA256 0a90f766adc234afb99116bdfa52711cfdb057e4977ad6606d57be139e116484；对应上游 crate SHA256 74e1003eeb47ed3d42cbdecf2bdf722c332736ab85440137050d90760c897e91。 | [cargo-nextest_v0.9.100_loongarch64.deb](/home/laevatein/ChinaOSProject/binaries/cargo-nextest/loongarch64/cargo-nextest_v0.9.100_loongarch64.deb)，Architecture loongarch64，Version 0.9.100-2，Depends `libc6 (>= 2.28), libgcc1, cargo (>= 1.41), rustc (>= 1.41)`，SHA256 a1c88a5d067bbc65cad42be3b402bb450e0ff29b811a2d96f5ba26d4903a1159。包带 README、copyright 和 Apache/MIT 全文；fresh split-SDK 容器的 apt 单包依赖解析拉入 Cargo，ABI 与入口核验通过，最小 runner smoke 1 项通过。旧 0.9.100-1 / SHA256 98cb21ace691de7ffcfab457bb4655b0c6cbf65ad7345d95931c46e9b1809830 已撤回并保存在 Herdr 专属缓存，不作为发布候选。没有重跑 Herdr suite。
| Git 2.43.0 / git-man | 已有项目包仍为 1:2.43.0-2；包位于 binaries/git/loongarch64/ 和 binaries/git-man/all/，SHA256 分别为 90cc1e4de09df21c062c9b7fdcddda7a01be95a33955cfc1cd13177ae9d1a8bd、e31490565fe3fe6d211b54c4977e264227bd2a2825024646fe181ba9effb46e0。 | 只读核对并保持原样，没有替换或重打包，也没有把其手册混入 Git 2.45.4 包。 |
| gawk、OpenSSH client、strace、build-essential/GCC 8.3、binutils、libc6-dev、make | 通过 Loongnix DaoXiangHu-stable 仓库作为发行包安装；不是本项目新编或重打包工具。 | 继续使用发行包，不额外分包。`scalar` 随 Git 2.45.4 包提供；`zig-cc`、`zig-ar`、`linker-lld21` 是已有工具的本地调用 shim。 |

### 补充包验证日志

- Herdr 0.9.5 实际容器安装、ABI 和入口日志：`/tmp/herdr-v0.9.5-20261105/logs/package-install-verification.log`。
- Git 2.45.4 受共同 flock 保护的重编日志及状态：/mnt/chinaos-cache/ChinaOSProject-cache/herdr/v0.9.5/supplemental-20261106/logs/git-2.45.4-build.log、git-build-status.json。
- Git 1:2.45.4-2 与 git-man 整组安装、dpkg metadata、ABI、动态依赖、文档分包检查：/mnt/chinaos-cache/ChinaOSProject-cache/herdr/v0.9.5/supplemental-20261106/logs/git-package-install-verification.log、git-package-file-split-audit.log。
- Git HTTP/HTTPS ls-remote、clone、受信 IP SAN CA 成功及不受信系统 CA 拒绝：/mnt/chinaos-cache/ChinaOSProject-cache/herdr/v0.9.5/supplemental-20261106/logs/git-http-https-smoke.log，机器可读结果位于 verify/git-host-transports/results.json。
- cargo-nextest 0.9.100-2 验证容器为 `herdr-nextest-split-verify-20261107`。四个 Rust 1.99.0-2 split 核心包和 nextest 候选的 apt 安装日志：/mnt/chinaos-cache/ChinaOSProject-cache/herdr/v0.9.5/supplemental-20261106/logs/cargo-nextest-split-apt-install.log；移除 Cargo/nextest 后只请求 `cargo-nextest`、APT 自动安装 `cargo 1.99.0-2` 的日志：`cargo-nextest-split-dependency-only-install.log`；软件包归属、版本、许可证路径、ABI flags 0x3、loader `/lib64/ld.so.1`、最高 GLIBC_2.28 和动态依赖日志：`cargo-nextest-split-install-verification.log`；共同锁下 one-test runner smoke：`cargo-nextest-split-runner-smoke.log`，结果 JSON 为 `verify/cargo-nextest-split-smoke.json`。旧 1.98.1 单体环境日志 `cargo-nextest-final-install-check.log` 和 `cargo-nextest-runner-smoke.log` 保留作历史记录，不作为新 Depends 或 split SDK 验证证据。未重跑 Herdr suite.
- just 1.43.0 补充包解包根目录中的版本、man page、recipe、ELF 和动态库检查：`/tmp/herdr-v0.9.5-20261105/logs/just-package-verification.log`。
- Git 2.45.4、git-man 2.45.4 和 cargo-nextest 0.9.100 三个正式候选均有 SHA256 sidecar。原 Herdr deb、Git 2.43.0 包、Rust/Zig/LLVM deb、共享缓存和历史 suite 结果均未修改。
