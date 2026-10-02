# LLVM 22.1.8 LoongArch64 构建报告

Dependency: LLVM compiler infrastructure
Version: 22.1.8, Debian package version 22.1.8+rust1.98.1-2
Type: Toolchain dependency
Reason: Rust 1.98.1 LoongArch64 编译需要支持目标系统 ELF ABI v0 的 LLVM 与 LLD。
Source: https://ftp.loongnix.cn/toolchain/llvm/llvm22/
Changes: 使用 Loongnix LLVM 22.1.8 源码构建完整 LLVM 工具与开发文件，生成 Architecture: loongarch64 软件包。本次没有修改 LLVM 源码。
Compatibility Impact: 软件包声明依赖 libc6 (>= 2.28)、libgcc1、libstdc++6、zlib1g。软件包内的 LLD 成功链接 ABI v0 Rust 程序，输出 ELF 标志为 0x3。
Validation: 从软件包解包后运行 llvm-config、llc、ld.lld、FileCheck，均报告 LLVM 22.1.8；llvm-config 的 prefix 指向解包目录中的安装路径。软件包内 LLD 完成 Rust 测试程序链接，生成程序运行并输出 `loongarch v0 smoke`。两个 Debian 归档均完成完整内容枚举。
Package: `../binaries/llvm-22/loongarch64/llvm-22_v22.1.8_loongarch64.deb`
Package SHA-256: c844ce421e4265ab154f414849fce25b6d1d73f45d21a52030866bcfe7911f44

Repository: https://github.com/llvm/llvm-project
