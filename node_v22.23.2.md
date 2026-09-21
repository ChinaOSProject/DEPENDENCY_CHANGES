# Node.js v22.23.2

Dependency: GCC、libstdc++、libgcc
Version: 13.4.0
Type: 构建工具链及静态运行库
Reason: 满足源码 C++20 编译要求，并在 Debian 10 运行。
Root Cause: Debian 10 默认编译器不足以完成本次构建；GCC 13 动态 libstdc++ 引入高于 Debian 10 的 GLIBCXX 依赖。
Changes: 在 Debian 10 构建 GCC 13.4.0；最终 make 使用 LDFLAGS.target="-static-libstdc++ -static-libgcc"。
Compatibility Impact: 最终 node 无动态 GLIBCXX 依赖；最高 GLIBC 为 2.28；目标系统无需替换运行库。
Upstream Status: 未修改上游源码。
Validation: 编译、readelf、ldd 检查完成；纯 Debian 10 容器中 deb 安装及启动通过，版本号为 v22.23.2，平台为 linux:x64。未运行完整 Node.js 测试集及上游 CI。
Project-side Changes: 无源码修改。

Dependency: Python
Version: 3.14.4
Type: 构建工具
Reason: 执行 configure 和 GYP。
Root Cause: Debian 10 Python 3.7 低于本版本构建要求。
Changes: 通过独立 loader 包装器调用宿主机 Python，设置 PYTHONHOME 和 PYTHONEXECUTABLE；Python 动态库搜索路径仅用于 Python 进程。
Compatibility Impact: Python 仅用于构建，不属于 node 的运行依赖。
Upstream Status: 未修改 Python 源码。
Validation: configure、GYP 和构建完成。
Project-side Changes: 无源码修改。

产物：node_v22.23.2_amd64.deb，采用 xz，声明 libc6 (>= 2.28)。包内仅包含 /usr/bin/node，未包含 npm、npx 和开发头文件。本说明仅在本地保存。
