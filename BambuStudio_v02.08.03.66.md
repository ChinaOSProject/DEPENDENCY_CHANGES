# 依赖适配说明

## OpenMeshCraft

- Dependency: OpenMeshCraft。
- Version: `0e8d12c3df54804393593ab5d86c05caa340cbee`。
- Type: CMake ExternalProject 第三方源码依赖。
- Reason: 支持 Debian 10 的 CMake 3.13.4、GCC 8 和系统 libstdc++。
- Root Cause: `shewchuk-predicates` 要求 CMake 3.14；目标 libstdc++ 缺少 `std::reduce`；GCC 8 无法接受部分函数参数前的 `OMC_UNUSED` 属性位置。
- Changes: 提供 CMake 3.13 的 shewchuk 构建文件；GNU 编译器版本低于 9 时通过编译定义将 `OMC_UNUSED` 设为空；包围盒归约根据 `<numeric>` 的 `__cpp_lib_parallel_algorithm` 能力宏选择 `std::reduce` 或 `std::accumulate`。后者依次合并包围盒，保持包围范围的计算含义。
- Compatibility Impact: 保持依赖版本，使用系统 glibc、libstdc++ 和头文件。支持归约算法的标准库保留原有调用；属性兼容定义仅用于 GNU 编译器版本低于 9 的构建。
- Upstream Status: 本项目维护的依赖补丁，尚未提交上游。
- Validation: Debian 10 amd64 的 GCC 8.3.0 和当前系统 GCC 15.2.0 通过标准库能力选择及最小值、最大值归约测试；补丁应用检查通过；GCC 15.2.0 在包含空格的目录中成功构建 shewchuk 静态库。验证范围为针对性测试，完整应用构建、运行及 AArch64、LoongArch64 验证尚未完成。
- Project-side Changes: CMake ExternalProject 应用依赖补丁，复制依赖构建文件，并设置旧 GNU 编译器所需的编译定义。

## Debian 10 运行依赖

- Dependency: `libopengl0`。
- Version: `1.1.0-1`，来自 2024-06-12 Debian 10 snapshot。
- Type: 系统运行库。
- Reason: 可执行文件启动需要 `libOpenGL.so.0`。
- Root Cause: 程序链接的 OpenGL 库由该软件包提供。
- Changes: 本地运行镜像安装该软件包，deb 声明该依赖；打包配置未包含在源码提交中。
- Compatibility Impact: 使用 Debian 10 提供的 OpenGL 库。
- Upstream Status: 本地运行环境及打包配置，无上游补丁。
- Validation: 已有 Debian 10 amd64 镜像通过动态库解析、CLI 帮助及 Prusa.stl 切片检查，deb 安装后通过 CLI 帮助检查。这些产物尚未针对本次源码重新构建。
- Project-side Changes: 无。
