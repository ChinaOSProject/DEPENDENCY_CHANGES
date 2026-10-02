# Herdr 0.9.1 LoongArch64 适配报告

Project: Herdr 0.9.1
Source: `065ef9d6a531c49fb8bee7e818ef837065b21ee9`，`loong64` 分支
Environment: `laevatein@192.168.4.55`，Kylin V10 SP1，Linux 5.4.18，glibc 2.28，GCC 8.3.0，`loongarch64`
Image: `localhost/kylin-v10-sp1-apt:10.1`
Container: `herdr-loong64-compile-20260925`，使用用户指定的 rootless overlay 存储参数

Dependency: Zig compiler toolchain、LLVM/LLD、Rust compiler toolchain
Version: Zig 0.16.0、LLVM/LLD 21.1.8、Rust 1.98.1（软件包版本 1.98.1-2，使用 LLVM 22.1.8）
Type: Toolchain

Reason:
Herdr 的 libghostty-vt 通过 C ABI 与 Rust 互调，目标设备使用 LoongArch ELF ABI v0 及 glibc 2.28。

Root Cause:
Zig 0.16.0 的 LLVM backend 缺少 LoongArch C ABI 的结构体参数及返回值处理。`GhosttyTerminalScrollViewport` 的大小为 24 bytes，LoongArch LP64 要求通过地址传递；Rust 按此规则传递地址，Zig 生成的函数将参数寄存器直接作为结构体字段读取。真实测试中的 `ghostty_terminal_scroll_viewport` 因此出现 SIGSEGV，TUI server 也会在处理终端视图时退出。系统 GNU ld 2.31.1 对该构建的 `.eh_frame_hdr` 报告 overlapping FDEs，链接需要使用已适配旧版 LoongArch ABI 的 LLD。

Changes:
- `src/codegen/loongarch64/abi.zig`：增加 LoongArch64 C ABI 类型分类、结构体及 union 的整数传递、浮点成员处理和参数寄存器数量计算。
- `src/codegen/llvm/FuncGen.zig`：为 LoongArch64 参数、返回值及 sret 提供对应处理，并使用字段自身的 alignment。
- `src/codegen/llvm.zig`：使用对应字段的 alignment 保存参数。
- 在独立 Zig 源码目录应用修改，复用现有 LLVM 21.1.8 及 Zig C++ 集成库，构建修复后的 Zig 0.16.0。
- Cargo 使用系统 GCC 8 驱动及 `-fuse-ld=lld`，链接器为 LLD 21.1.8。
- Rust 1.98.1-2、Zig 的旧版 Linux 信号接口及 native loader 适配来自已提供的工具包。

Compatibility Impact:
编译器修改限定在 LoongArch64 C ABI。最终 Herdr 为 ELF ABI v0，flags 为 `0x3`，loader 为 `/lib64/ld.so.1`，最高 glibc 符号需求为 `GLIBC_2.28`。运行依赖为 `libc6 (>= 2.28)` 和 `libgcc1`。实际运行验证范围为上述 Kylin 开发机；现代 LoongArch64 target 完成编译检查。

Upstream Status:
本次 Zig C ABI 修改自行实现，补丁单独归档。LoongArch 调用规则依据 [Procedure Call Standard](https://raw.githubusercontent.com/loongson/la-abi-specs/release/lapcs.adoc)。

Validation:
- GCC 8 与修复版 Zig 双向互调通过，覆盖 3、8、12、16、24 bytes 的结构体、union、嵌套浮点成员及参数寄存器数量不足的情况。
- `loongarch64-linux.5.19-gnu.2.36` target 编译检查通过。
- `cargo build --release --locked --offline` 成功，使用修复版 Zig 编译完整 libghostty-vt。
- 41 个 `ghostty::tests` 全部通过，0 个失败、0 个忽略。
- 开发机本体通过 SSH PTY 启动 TUI，验证两个真实窗格的命令输入、中文输出、窗格创建及关闭。
- Debian 包在专用 OCI 容器中安装成功；安装后的 `/usr/bin/herdr` 与已验证二进制逐字节一致。
- 安装后的 server、workspace 创建及真实 PTY 中文输出检查通过。
- `rustfmt --check build.rs` 通过；ABI、动态库及 glibc 版本需求通过 `readelf` 和 `ldd` 检查。

Project-side Changes:
`build.rs` 增加 `loongarch64-unknown-linux-gnu` 到 `loongarch64-linux.5.4.18-gnu.2.28` 的映射。

Patch: `zig_v0.16.0_loongarch64-herdr-cabi.patch`
Patch SHA-256: `2d396a326c57a34b7c5f24cd7f82611e238b99113e1d7e2bef0b9db4c180c583`
Patch Check: 在现有 Zig 工作目录中通过 `git apply --check`。

Package: `../binaries/herdr/loongarch64/herdr_v0.9.1_loongarch64.deb`
Package Version: `0.9.1-1`
Package Size: `7116000 bytes`
Package SHA-256: `cdd8c9a2a30709ab5fbea0242e10edcf2c5fc34444ed8ee79802f2b71ff3ac8d`
Binary SHA-256: `6889fc4797db1e31e0e08c355175d5a9988552f01bf7aae8f217a069ab1a00f4`

Build Evidence: 开发机 `$HOME/.cache/herdr-loong64-build/main-20261001/`，包含构建脚本、互调程序、测试日志及打包日志。
