# Node.js v20.20.2

Dependency: Clang
Version: 13
Type: 构建工具链
Reason: 在 Debian 10 编译当前 Node.js 源码。
Root Cause: 上游 BUILDING.md 的 Linux GCC 最低要求为 10.1，Debian 10 默认 GCC 为 8。
Changes: 构建环境使用 Clang 13，使用 Debian 10 系统头文件及运行库。
Compatibility Impact: 目标系统保留 glibc 2.28；产物最高 GLIBCXX 为 3.4.21，最高 CXXABI 为 1.3.7。
Upstream Status: 未修改上游源码。
Validation: 已完成编译、ABI 检查；Debian 10 容器安装包后版本号为 v20.20.2，平台为 linux:x64。未运行完整 Node.js 测试集及上游 CI。
Project-side Changes: 无源码修改。

Dependency: Debian 10 系统运行库
Version: glibc 2.28；libstdc++6 和 libatomic1 8.3.0-6
Type: 软件包运行依赖
Reason: 满足 ELF 动态链接依赖。
Root Cause: 产物需要 libc、libstdc++、libgcc_s 和 libatomic。
Changes: deb 声明 libc6 (>= 2.28)、libstdc++6 (>= 8.3.0)、libatomic1。
Compatibility Impact: 使用 Debian 10 自带运行库。
Upstream Status: 未修改系统库源码。
Validation: Debian 10 容器安装 libatomic1 后，deb 安装及启动通过；最高 GLIBC 为 2.28。
Project-side Changes: 本地打包采用 xz，安装文件为 /usr/bin/node。

产物：node_v20.20.2_amd64.deb。包内仅包含 node 可执行文件，未包含 npm、npx 和开发头文件。本说明仅在本地保存。
