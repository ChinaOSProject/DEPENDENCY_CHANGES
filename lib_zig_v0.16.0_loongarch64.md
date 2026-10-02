Repository:
https://github.com/ziglang/zig

Upstream:
https://codeberg.org/ziglang/zig
tag 0.16.0，commit 24fdd5b7a4c1c8b5deb5b56756b9dbc8e08c86a8

Dependency:
Zig 标准库、LLVM 21.1.8

Version:
Zig 0.16.0，Debian package version 0.16.0-1

Type:
LoongArch64 旧系统源码适配及 compiler 打包。

Reason:
在 laevatein@192.168.4.55 的 Kylin V10 SP1、Linux 5.4.18-168-generic、glibc 2.28、GCC 8.3.0 环境上编译并实际运行 Zig 0.16.0。输出架构名称采用开发机 uname 和 dpkg 报告的 loongarch64。

Root Cause:
旧版 LoongArch64 Linux 内核提供 128 个信号，rt_sigaction 要求 16 bytes 的 kernel signal mask。上游标准库的现代 LoongArch64 定义使用 64 个信号和 8 bytes 的 mask。

旧版 glibc 使用 /lib64/ld.so.1，native ABI 检测需要识别这个 loader，并保留实际检测出的 Linux 5.4 和 glibc 2.28 版本。LLVM 21 对目标系统 ELF ABI v0 和 SOP relocations 的支持由独立 LLVM 依赖适配提供。

Changes:

- lib/std/os/linux.zig：LoongArch64 Linux 目标的 kernel minimum version 小于 5.19 时，NSIG 使用 129，kernel sigset_t 使用 128 bits。
- lib/std/zig/system.zig：在 LoongArch64 Linux 的 loader 检测列表中加入 /lib64/ld.so.1，识别为 GNU ABI；该 loader 对应的版本使用实际检测结果。
- lib/std/os/linux/test.zig：增加 rt_sigaction 回归测试，通过真实 kernel 查询 USR1 的 sigaction，并检查返回值。
- 兼容修改归属 loong64 分支，debian10 保持为 0.16.0 的通用基准。
- 编译参数使用 ZIG_SHARED_LLVM=ON，ZIG_VERSION=0.16.0，ReleaseFast 和 strip；native 自动检测系统 loader 与库目录。
- 包内 Zig 使用 $ORIGIN/../lib/llvm-21/lib 查找 LLVM 共享库，Debian Depends 由实际 ELF 的 dpkg-shlibdeps 结果生成。

Compatibility Impact:
旧版接口选择限定在 LoongArch64 Linux 的对应目标。现代 LoongArch64 target 的编译检查通过，采用 ABI v1 和标准 lp64d loader。未在现代 LoongArch64 操作系统或 AArch64 设备上执行程序。

Upstream Status:
本次 Zig 兼容修改在本地 loong64 分支维护，未提交 upstream commit 或 PR。

Validation:

- 在 rootless Podman 容器 zig-0.16.0-loongarch64-build 中，从源码完成 zig1、zig2、stage3 和安装。
- 开发机本体执行最终 Zig，version 返回 0.16.0，env 返回 loongarch64-linux.5.4.18...5.4.18-gnu.2.28。
- 开发机本体编译并运行仓库的 hello.zig 和 hello_libc.zig，验证独立 Zig 程序及 glibc 链接。
- 开发机本体通过 zig cc 编译并运行 C 程序，通过 zig c++ 编译并运行仓库 test/standalone/c_compiler/test.cpp，验证全局初始化、thread_local、线程和 exception。
- 使用完整源码的标准库测试入口执行 rt_sigaction 和 sigset_t，具体用例均通过；每组 64 tests passed。
- 执行 test/behavior.zig：1996 passed、40 skipped、0 failed。skipped 来自上游测试的现有条件。
- 使用 loongarch64-linux.5.19-gnu.2.36 编译信号测试，未执行；ELF flags 为 0x43，interpreter 为 /lib64/ld-linux-loongarch-lp64d.so.1。
- readelf --version-info：最终 Zig 的最高 GLIBC_2.28，最高 GLIBCXX_3.4.22。ELF interpreter 为 /lib64/ld.so.1。
- 将两个 Debian 包提取到开发机的同一独立目录，清除验证进程的 LD_LIBRARY_PATH 后启动 Zig、llvm-config、Clang 和 LLD，并编译运行程序。
- ldd 确认 LLVM 和 Clang 共享库来自包的提取目录，glibc、libstdc++ 等来自开发机现有系统目录。
- dpkg-deb --fsys-tarfile 与 tar --compare 验证 Debian 包文件内容；本地与开发机上的包 SHA256 一致。
- 三个修改文件通过 zig fmt --check。

Project-side Changes:
lib/std/os/linux.zig、lib/std/os/linux/test.zig、lib/std/zig/system.zig。

Dependency Changes:
LLVM、Clang、LLD 21.1.8 的适配单独归档于 lib_llvm-21_v21.1.8_loongarch64.md 和 llvm-21_v21.1.8_loongarch64.patch。

Patch:
zig_v0.16.0_loongarch64.patch

Patch SHA256:
efcc681c5d2cce8abefe1935da5ba819046154323eecf2e9834d4c666a719294

Package:
../binaries/zig/loongarch64/zig_v0.16.0_loongarch64.deb

Package SHA256:
c4080c8442a0240e4e9f8855ff87a0a7da7352f03d07cc8032395fc1d157ec8e

Package Depends:
libc6 (>= 2.28), libstdc++6, llvm-21 (>= 21.1.8-1), zlib1g (>= 1:1.2.2)

Build Container:
zig-0.16.0-loongarch64-build

Build Image:
localhost/zig-build:0.16.0-llvm21.1.8-loongarch64
SHA256: 137d46f98f23966104d8ba858e08a6f9f392dd0fa98be10106a81d26fbe01e1f

Build Directory:
开发机 /home/laevatein/src/zig-0.16.0-container，容器挂载路径 /work。

Build Command:
podman exec zig-0.16.0-loongarch64-build bash /work/run-zig-build.sh

Trust Policy:
开发机用户的持久 Podman policy.json 允许 cr.loongnix.cn/library/debian 镜像仓库。
