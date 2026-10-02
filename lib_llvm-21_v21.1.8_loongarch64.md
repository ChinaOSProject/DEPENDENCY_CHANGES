Repository:
https://github.com/llvm/llvm-project

Dependency:
LLVM、Clang、LLD

Version:
21.1.8，Debian package version 21.1.8-1

Type:
LoongArch64 旧版 ABI 源码适配及编译配置。

Reason:
在 Kylin V10 SP1、Linux 5.4.18-168-generic、glibc 2.28、GCC 8.3.0 的开发机上编译和运行 LLVM 21，并作为 Zig 0.16.0 的 LLVM 依赖。

Root Cause:
目标系统的 GCC 8 和 glibc 使用 LoongArch ELF ABI v0、SOP relocations 以及 /lib64/ld.so.1。上游 LLVM 21 的 LLD 没有提供完整的旧版 SOP relocation 处理。旧版 GCC 生成的 .eh_frame 使用绝对地址编码，PIE 链接需要正确转换 personality、LSDA 和 FDE 指针。旧版 loader 路径也需要通过 sysroot 中实际存在的文件识别。

Changes:

- 在 LLD 的 LoongArch relocation 处理中加入 SOP 操作及操作数检查，支持 ABI v0 和 v1 输入；保留不兼容 float ABI 和无效 ABI version 的错误检查。
- 使用现有 EhReader 和 LLVM DataExtractor 读取 CIE/FDE 信息，在 LoongArch64 旧版输入的 PIC/PIE 链接中将相应绝对地址编码转换为等宽 PC-relative 编码。
- Clang 通过 VFS 检查 sysroot：标准 lp64d loader 路径不存在且 /lib64/ld.so.1 存在时使用旧版 loader。
- 补充 SOP relocation 的正常输入和错误输入测试，更新 ABI v0/v1 interlink 测试。测试中的 ELF header 修改使用 pyelftools 读取和写入。
- 使用 LLVM_BUILD_LLVM_DYLIB=ON 和 LLVM_LINK_LLVM_DYLIB=ON 提供共享 LLVM 依赖。系统 zstd 1.3.8 缺少 LLVM 21 所需 API，设置 LLVM_ENABLE_ZSTD=OFF。
- 包内工具安装于 /usr/lib/llvm-21，提供带 -21 后缀的 /usr/bin 链接，并生成实际共享库的 Debian symbols 依赖索引。

Source:

- LLVM 21.1.8 官方源码： https://github.com/llvm/llvm-project/releases/download/llvmorg-21.1.8/llvm-project-21.1.8.src.tar.xz
- 官方源码 SHA256：4633a23617fa31a3ea51242586ea7fb1da7140e426bd62fc164261fe036aa142
- SOP relocation 参考来源：Loongnix 发布的 LLVM 22.1.8 源码， https://ftp.loongnix.cn/toolchain/llvm/llvm22/
- Loongnix llvm-project_22.1.8-1.src.tar.gz SHA256：9acdcc3531fe7641bc9455a644508b03d3facecbfb4a92ebfde7f01df1e5b278
- Loongnix lld/ELF/Arch/LoongArch.cpp SHA256：5b16f21c50f58fdde58bbbd05f6bd29f42bf944df7886ba781a23e5a3f7a7905
- 适配补丁：llvm-21_v21.1.8_loongarch64.patch
- 适配补丁 SHA256：73ab90191ad3fbea05c626dc98073b41abb1e89f8aaf87b32d4bd72c4571b8bd

Compatibility Impact:
旧版 LoongArch64 ABI 支持限定在对应目标和输入文件。现代 ABI v1 的链接测试通过。共享库安装在 LLVM 21 的独立目录，目标系统的 glibc 和 libstdc++ 保持原有版本。未进行 AArch64 或现代 LoongArch64 操作系统上的运行测试。

Upstream Status:
SOP relocation 处理参考已公开的 Loongnix 源码。本次针对 LLVM 21 的适配、EH 编码转换和测试在本地维护，未提交 upstream commit 或 PR。

Validation:

- rootless Podman 容器 zig-0.16.0-loongarch64-build，基础镜像 cr.loongnix.cn/library/debian:buster-slim，glibc 2.28。
- 使用 GCC 8.3.0 和 CMake 3.27.9 完成 LLVM 21.1.8 Release 编译及安装，启用全部默认 target backends、Clang 和 LLD。
- 33 个 LLD LoongArch ELF 测试全部通过，覆盖现代 ABI v1、旧版 ABI v0 interlink、SOP 操作以及实际错误输入。
- 开发机本体执行 lli，ORC 和 MCJIT 均运行官方 hello.ll 并输出 Hello World。
- 开发机本体使用 Clang 21 和 LLD 21 编译并运行默认 PIE 的 C 程序。
- 开发机本体将 GCC 8 的旧版 ABI C++ 对象与 Clang 21 对象链接为 PIE，验证线程局部数据独立性和 C++ exception unwinding。
- C 程序 ELF interpreter 为 /lib64/ld.so.1，ELF type 为 DYN。
- 扫描全部包内 ELF 的 readelf --version-info：最高 GLIBC_2.27，最高 GLIBCXX_3.4.22。
- Debian Depends 使用实际 dpkg-shlibdeps 结果，并声明 python3 和 perl。
- 使用 dpkg-deb --fsys-tarfile 和 tar --compare 验证整个 Debian 包的文件内容。

Project-side Changes:
Zig 使用 ZIG_SHARED_LLVM=ON 链接这套 LLVM 21。Zig 的 LoongArch64 信号接口及 native loader 检测适配在 Zig 仓库的 loong64 分支单独维护。

Package:
../binaries/llvm-21/loongarch64/llvm-21_v21.1.8_loongarch64.deb

Package SHA256:
c292fe67bee233850b7983144cb1f6d3197c4db4d2f1d882c16ec85c66ee7be8
