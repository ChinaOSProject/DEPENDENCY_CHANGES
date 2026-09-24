# Node.js v24.21.0 LoongArch64

Dependency: GCC
Version: 13.4.0
Type: 构建工具链
Reason: 在 Kylin V10 SP1 与 glibc 2.28 环境中构建 Node.js v24.21.0。
Root Cause: 编译设备默认 GCC 8 无法完成该版本的 C++20 构建。
Changes: 使用编译设备用户目录中的 GCC 13.4.0 和 Python 3.10.16；使用 bubblewrap 隔离构建；使用 `--openssl-no-asm`、`--partly-static`、`-DHWY_COMPILE_ONLY_EMU128` 与 `/lib64/ld.so.1` 动态加载器。
Compatibility Impact: 产物最高 GLIBC 为 2.28；zlib 与 libatomic 使用目标系统共享库；未发现动态 GLIBCXX 或 CXXABI 版本依赖。
Upstream Status: 使用 `v24-debian10` 分支源码；Highway 的 LoongArch GCC 判断修改待提交。
Validation: 目标机输出 `v24.21.0`、`loong64 linux`；JSON 字符串序列化往返测试通过；`ldd`、ELF 动态加载器和 ABI 版本检查通过；`.deb` 解包后的 `/usr/bin/node` 在目标机 bubblewrap 中运行通过；未运行完整 Node.js 测试集及上游 CI。
Project-side Changes: V8 JSON stringifier 在 Highway scalar target 使用标量实现；Highway 仅在 LoongArch 保留 EMU128；安装文件为 `/usr/bin/node`，未包含 npm 和 Corepack。

Dependency: Kylin V10 SP1 系统运行库
Version: glibc 2.28；zlib、libatomic、libstdc++、libgcc
Type: 软件包运行依赖
Reason: 满足 LoongArch64 ELF 动态链接要求。
Root Cause: Node.js 运行时需要系统 libc、zlib、atomic 及 C++ 支持库。
Changes: Debian 包声明 `libc6 (>= 2.28)`、`zlib1g`、`libatomic1`、`libstdc++6` 与 `libgcc1`。
Compatibility Impact: 不需要替换目标系统运行库。
Upstream Status: 未修改系统库源码。
Validation: ELF 解释器为 `/lib64/ld.so.1`；目标机 `ldd` 可解析所有动态库；最高 GLIBC 为 2.28。
Project-side Changes: Debian 包仅包含 `/usr/bin/node`，采用 xz 压缩。

产物：`node_v24.21.0_loong64.deb`
