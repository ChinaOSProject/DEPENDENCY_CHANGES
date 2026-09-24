# Node.js v20.20.2 LoongArch64

Dependency: GCC
Version: 8.3.0-6.lnd.vec.36.1
Type: 构建工具链
Reason: 在 Kylin V10 SP1 与 glibc 2.28 环境中构建 Node.js v20.20.2。
Root Cause: 编译设备默认 GCC 为 8.3.0，Node.js v20 源码可以在该设备上完成构建。
Changes: 使用编译设备已有的用户级 GCC 8 工具链；使用 bubblewrap 隔离构建过程；使用 `--openssl-no-asm` 选择 LoongArch 无汇编 OpenSSL 配置；链接器使用 `/lib64/ld.so.1`。
Compatibility Impact: 产物最高 GLIBC 为 2.28，最高 GLIBCXX 为 3.4.21，最高 CXXABI 为 1.3.9。
Upstream Status: 未修改上游源码。
Validation: 直接运行 `v20.20.2` 通过；`process.arch` 为 `loong64`；基础 JavaScript 执行通过；bubblewrap 隔离环境运行通过；未运行完整 Node.js 测试集及上游 CI。
Project-side Changes: 无源码修改。包内仅包含 `/usr/bin/node`。

Dependency: Kylin V10 SP1 系统运行库
Version: glibc 2.28；系统 libstdc++、libgcc_s、libatomic
Type: 软件包运行依赖
Reason: 满足 LoongArch64 ELF 动态链接要求。
Root Cause: Node.js 运行时需要 libc、libstdc++、libgcc_s 与 libdl 等系统库。
Changes: Debian 包声明 `libc6 (>= 2.28)`、`libstdc++6`、`libgcc1` 与 `libatomic1`。
Compatibility Impact: 产物可以直接在编译设备上运行，不需要替换系统运行库。
Upstream Status: 未修改系统库源码。
Validation: `ldd`、ELF 解释器、版本信息检查通过；Debian 包解包后运行版本检查通过。
Project-side Changes: 包采用 xz 压缩，安装文件为 `/usr/bin/node`，未包含 npm、npx 和开发头文件。

产物：`node_v20.20.2_loong64.deb`
