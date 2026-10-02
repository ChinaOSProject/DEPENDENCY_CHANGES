# Rust 1.98.1 LoongArch64 工具链构建报告

Dependency: Rust compiler toolchain
Version: rustc 1.98.1 (48a229ceaefd4985c50990b14116b6d856af0985), LLVM 22.1.8
Type: Toolchain
Reason: Kylin V10 SP1 LoongArch64 环境使用 glibc 2.28，需要 Rust 1.98.1 工具链生成 ELF ABI v0 对象。
Changes: Rust ELF 元数据采用 ABI v0 标志；LLVM LoongArch 功能名由目标机功能列表确定；LoongArch 目标禁用 f16、f128 可靠性配置；目标数据布局与 CPU 名称更新；stdarch 的 dbar、ibar intrinsic 参数改为与 LLVM 声明一致的 i64。
Compatibility Impact: 软件包声明 Architecture: loongarch64，并依赖 libc6 (>= 2.28)、libgcc1、libstdc++6、zlib1g。打包后的 rustc 与 libstd ELF 标志为 0x3。依赖符号检查显示 glibc 最高版本为 GLIBC_2.28。
Upstream Status: 本次源码适配尚未提交上游。
Validation: Rust 1.98.1 stage2 完成构建；rustc、Cargo、rustdoc、rustfmt、cargo-fmt 均报告对应版本。由解包后的 Rust 软件包分别使用系统 GCC 与软件包内 LLD 编译 LoongArch64 测试程序，生成文件的 ELF 标志为 0x3，程序输出 `loongarch v0 smoke`。解包后的 Cargo 成功构建仓库中的 wasm-component-ld 项目。
Package: `../binaries/rustc/loongarch64/rustc_v1.98.1_loongarch64.deb`
Package Version: 1.98.1-2
Package SHA-256: c875e0565f04b3989d008faf283f8583d200bf3aeb140926e0ec3329f3f0e734

Repository: https://github.com/ChinaOSProject/rust
