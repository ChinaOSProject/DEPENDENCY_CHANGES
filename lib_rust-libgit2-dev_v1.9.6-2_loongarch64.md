# libgit2 1.9.6 LoongArch ABI1.0 私有静态 SDK

上游仓库：https://github.com/libgit2/libgit2

交付 `rust-libgit2-dev` `1.9.6-2`，Architecture=`loongarch64`，安装前缀 `/opt/rust-native-1.99.1-dev`。正式候选：`/home/laevatein/ChinaOSProject/binaries/rust-libgit2-dev/loongarch64/rust-libgit2-dev_v1.9.6-2_loongarch64.deb`；大小 1,512,092 字节，SHA256 `734f8200cfc175b630d4310d8280bb749d92bbcecec33a6b33e4d02591bd6ed9`，同目录 `.deb.sha256` sidecar 已生成。尚未由本任务 commit、push 或在宿主安装，交由主代理统一审核发布。

## 来源与分包依据

实际完整上游版本为 `1.9.6`。源码来自官方 registry crate `libgit2-sys 0.18.7+1.9.6`，crate SHA256 `23c7391e4b9f4ffab1a624223cc1d7385ff9a678f490768add717de7ea2f4d89`，与权威 lock、官方 index checksum 及归档内容一致。详细源文件/头文件/生成配置来源和哈希见本任务 manifest。除下述明确的 libssh2 独立原版构建外，静态归档直接复用已成功的官方 Rust tools 输入构建，不重编、不修改源码或归档字节。

参考已缓存 Debian Sid Sources 的 `libgit2 1.9.7+ds-1`，标准开发包名 `libgit2-dev`，并核对现有 Debian Buster 衍生容器 apt 元数据的分包/依赖。Sources 快照 SHA256 `20710fbabe55a1373672046a56a8fd7decda2f9d4c7b13290220ca7addc51c76`；apt 元数据证据 SHA256 `b3ff0d826f62d49ceb7efc189f0651b016412dffa616183e1323ae2e82b76c0b`。两组材料均为本地只读复用，没有为了命名进行推测性网络请求。

按真实内容采用 `-dev`：静态库、准确公开/生成头文件、许可、可用 pkg-config 元数据。名称增加 `rust-`，前缀改为私有 opt 目录，以允许与基座 OpenSSL1.1 等系统 SDK 并存；未声明虚假的 Provides/Replaces/Conflicts，未复制没有实际生成的 SONAME 运行库或工具。没有共享运行库，故不机械沿用发行版同版本 shared-runtime Depends。包内 105 个普通文件；不存在与整组六包重复的普通文件归属。

此构建以 OpenSSL/libssh2 为 HTTPS/SSH 后端，PCRE2、llhttp、xdiff、SHA1 collision-detection 实现已编入同一 libgit2.a，而非另产可独立使用的 C SDK 归档；没有虚构对应共享运行包，也没有机械添加未使用的外部 PCRE/HTTP-parser 开发依赖。相关上游许可/作者及第三方 notice 均随包保留。修订2仅更新 README、control/md5 等打包元数据，精确依赖完整 libssh2 修订2；libgit2.a 和所有头文件与成功 Cargo 构建保持相同 SHA256。

## 依赖与链接闭包

实际 control Depends：`rust-libssl-dev (= 3.6.3-1), rust-libssh2-1-dev (= 1.11.1~dev+crate0.3.2-2), zlib1g-dev, libc6-dev (>= 2.28) | libc-dev`。`Recommends: pkg-config`，`Suggests: c-compiler`。依赖来自实际静态未定义符号、真实 pkg-config/CMake 元数据和全归档链接检查；已有系统 zlib/libc 开发接口按真实复用声明，没有把整个构建工具清单写入 Depends。

使用 `PKG_CONFIG_PATH=/opt/rust-native-1.99.1-dev/lib/pkgconfig pkg-config --static --cflags --libs libgit2`。消费 OpenSSL 的 curl/libgit2/libssh2 pc 明确约束私有3.6.3，并核对整组实际解析的 libcrypto 版本为3.6.3；curl 另含 nghttp2/zlib，libgit2 另含完整 libssh2/zlib，SQLite 含 m/dl/pthread。元数据不存在失效 `/cargo-home` 安装前缀或未展开模板 token，解析的 pcfiledir 为私有目录；未悄然链接系统 OpenSSL1.1。

| 归档 | ELF 成员数 | SHA256 |
| --- | ---: | --- |
| `libgit2.a` | 205 | `37a761f42fcfb93d2cc76546d31506f7253344b1377bed3b35f3c0860a3146eb` |

全部归档成员 machine=258、e_flags=0x3。静态归档无动态解释器或 SONAME；最终消费者的 ABI 证据见下节。

## 验证与边界

全部实际构建、打包压缩、安装及 C consumer 编译均在既有 `codex-v0.166.0-v8-directory-probe-20261107` 容器中，持共同 `/tmp/chinaos-heavy-build-20261105.lock`、最多2 jobs；没有新建容器、绕过镜像策略、修改官方锁/源码缓存或宿主全局依赖。正式六包整组 dpkg 安装成功，各包 dpkg 校验和 payload 与 staging 一致；系统 OpenSSL1.1 版本、公开头文件、静态/共享库 SHA256 在安装前后相同。

初始化专属本地 bare repository，写入并读取 blob，核对内容；git_libgit2_version 为1.9.6，SSH/HTTPS feature 已启用。修订2使用最终完整 libssh2 重新链接并运行，通过。 每组 SDK 还对全部归档成员执行 `--whole-archive` 链接并运行空入口，验证实际声明的静态链接闭包，并未只以一个版本字符串代替库功能。六个最终 consumer 均为 flags0x3、`/lib64/ld.so.1`，本包 consumer 最高 GLIBC `2.27`；整组最高2.28，ldd 全解析，未动态依赖私有 SDK 的 SSL/curl/git2/ssh2/nghttp2/sqlite 共享库。未宣称外网集成或全套上游测试通过。

本任务从未执行、启动、停止、重启或向 Magpie 发送信号，包含没有运行 Magpie --version。

完整证据根：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/rust-1.99.1-native/native-c-sdk-packaging-20261112`。正式索引 `deliverables-manifest.json`、`final/packages-index.json`（完整 control）、`final/payload-index.json`、`final/native-archive-index.json`、`final/static-symbol-closure.json`、`final/consumer-abi.json`、`final/system-coexistence.json`。初次失败日志和撤回候选保留在原目录及 `historical-incomplete-libssh2-candidates/NOT-FOR-RELEASE.json`。
