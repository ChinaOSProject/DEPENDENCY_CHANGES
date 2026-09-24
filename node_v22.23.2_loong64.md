# Node.js v22.23.2 LoongArch64

Dependency: GCC
Version: 13.4.0
Type: 构建工具链
Reason: Node.js v22 源码需要 GCC 13 完成 C++20 构建。
Root Cause: 编译设备系统 GCC 8 无法满足本次源码的构建要求。
Changes: 使用编译设备用户目录中的 GCC 13.4.0；使用 bubblewrap 隔离构建过程；使用 `--openssl-no-asm` 选择 LoongArch64 无汇编 OpenSSL 配置；使用 `-static-libstdc++ -static-libgcc` 处理 GCC 运行库；链接器使用 `/lib64/ld.so.1`。
Compatibility Impact: 产物最高 GLIBC 为 2.28，未引入动态 GLIBCXX 与 CXXABI 依赖。
Upstream Status: 未修改上游源码。
Validation: 远程编译完成；目标设备直接运行输出 `v22.23.2`、`loong64 linux`；基础 JavaScript 执行通过；Debian 包解包后运行通过；ELF 解释器、`ldd` 与版本信息检查通过；未运行完整 Node.js 测试集及上游 CI。
Project-side Changes: 无源码修改。

Dependency: Python
Version: 3.10.16
Type: 构建工具
Reason: 执行 `configure.py` 与 GYP。
Root Cause: Node.js v22 构建过程需要 Python 3.10。
Changes: 使用编译设备用户目录中的 Python 3.10.16；从编译设备默认 APT 源获取 `libbz2-dev` 1.0.8-2，并在用户目录解压开发文件；使用 bubblewrap 隔离环境构建 Python `_bz2` 扩展。
Compatibility Impact: Python 与 bzip2 开发文件只参与构建，不属于 Node.js 运行依赖。
Upstream Status: 未修改 Python 与 bzip2 源码。
Validation: Python 3.10 导入 `bz2` 并完成压缩操作；Node.js v22 配置与构建通过。
Project-side Changes: 无源码修改。

Dependency: Kylin V10 SP1 系统运行库
Version: glibc 2.28；系统 `libatomic`、`libc`、`libdl`、`libm` 与 `libpthread`
Type: 软件包运行依赖
Reason: 满足 LoongArch64 ELF 动态链接要求。
Root Cause: Node.js 运行时需要目标设备系统动态库。
Changes: Debian 包声明 `libc6 (>= 2.28)`、`libstdc++6`、`libgcc1` 与 `libatomic1`。
Compatibility Impact: 产物可以直接在编译设备上运行，不需要替换系统运行库。
Upstream Status: 未修改系统库源码。
Validation: ELF 解释器为 `/lib64/ld.so.1`；最高 GLIBC 为 2.28；目标设备直接运行与 Debian 包解包运行均通过。
Project-side Changes: 包采用 xz 压缩，安装文件为 `/usr/bin/node`，未包含 npm、npx 与开发头文件。

产物：`node_v22.23.2_loong64.deb`
