# Netavark v2.1.0 loongarch64

Dependency: Rust toolchain
Version: rustc 1.88.0; Cargo 1.88.0; Loongnix ABI 1.0 LoongArch64 distribution
Type: 构建工具链
Reason: Cargo.toml 要求 Rust 1.88，目标设备为 Kylin V10 SP1 loongarch64。
Root Cause: 目标机没有 Rust 工具链；Kylin 软件源提供的 rustc 候选版本为 1.41.0。
Changes: 将 Rust 1.88.0 安装到目标机用户缓存目录，并在 bubblewrap 中以只读方式使用。归档 SHA-256 为 af308e9f68ac622e12be0973d7ab236412d80379d050b8d135ebdb6cdbe770d4。
Compatibility Impact: 原生 LoongArch64 ELF 使用目标机系统运行库；未替换 glibc、libstdc++ 或系统头文件。
Upstream Status: 使用 Loongnix ABI 1.0 工具链发行包；未修改 Rust 上游源码。
Validation: rustc 与 Cargo 均报告 1.88.0；`cargo build --locked --release` 完成；三个 ELF 均识别为 LoongArch64。
Project-side Changes: 未修改 Cargo.toml、Cargo.lock 或项目源码。

Dependency: Protocol Buffers compiler
Version: protoc 3.6.1; libprotobuf17/libprotoc17 3.6.1.3-2kylin5.2+esm2
Type: 构建依赖
Reason: build.rs 通过 tonic-prost-build 编译 src/proto/proxy.proto。
Root Cause: 目标机原先未安装 protoc。
Changes: 从 Kylin 软件源下载 protobuf-compiler 与运行库，并解压到用户缓存目录供 bubblewrap 只读挂载；未安装到系统目录。
Compatibility Impact: protoc 只参与生成 Rust 源码，不增加 Netavark 的运行时共享库依赖。
Upstream Status: 使用 Kylin 提供的 protobuf 软件包；未修改 protobuf 源码。
Validation: 隔离环境中 `protoc --version` 输出 libprotoc 3.6.1；项目 release 构建完成。
Project-side Changes: 未修改项目依赖声明。

Dependency: Cargo crate registry
Version: Cargo.lock 锁定的 crate 版本
Type: crate 下载索引
Reason: 目标机 Cargo 配置的 crates.io 索引无法更新。
Root Cause: `https://crates.loongnix.cn/crates.io-index` 返回 HTTP 404。
Changes: 仅在 Cargo 命令行临时指定 RsProxy sparse 索引；未更改仓库 Cargo 配置或 Cargo.lock。
Compatibility Impact: 使用 `--locked` 保持 crate 版本不变；索引镜像不改变 Netavark 运行行为。
Upstream Status: 仅使用索引镜像分发 Cargo.lock 指定的 crate；未修改 crate 源码。
Validation: 索引更新及 crate 下载完成，release 构建通过。
Project-side Changes: 未修改 Cargo.toml、Cargo.lock 或 .cargo/config.toml。

Dependency: Target runtime libraries
Version: glibc 2.28; libgcc1; nftables 0.9.3-2 (Kylin 软件源候选版本)
Type: 运行时依赖
Reason: ELF 动态链接需要 libc 与 libgcc_s；默认 firewall 支持依赖 nftables 软件包。
Root Cause: 目标机运行基座为 Kylin V10 SP1，系统 glibc 为 2.28。
Changes: Debian control 声明 `libc6 (>= 2.28)`、`libgcc1`、`nftables`，并按项目打包说明将 `aardvark-dns` 列为推荐依赖。
Compatibility Impact: 主程序最高需要 GLIBC_2.28；未替换目标机系统共享库。`aardvark-dns` 为可选 DNS 服务；当前 Kylin 软件源没有该包候选，Netavark 核心网络配置功能可在不安装它时运行。
Upstream Status: 未修改系统运行库源码。
Validation: ldd 可解析主程序共享库；readelf 显示 GLIBC 版本需求最高为 2.28；主程序在目标机 bubblewrap 中通过 `--version` 与 `--help` 启动。
Project-side Changes: 仅新增 Debian 包元数据；项目源码与 Cargo 锁文件未修改。

Package: `../binaries/netavark/loongarch64/netavark_v2.1.0_loongarch64.deb`
Package SHA-256: 30e227efea8fa573526a33039beb0f39572af577fa2f96663b722c26a2b4479a
Validation scope: `netavark --version` 与 `netavark --help` 在目标机 bubblewrap 中启动通过；未执行容器网络配置与防火墙端到端测试。
