# Bambu Studio v02.08.04.57 LoongArch64 依赖适配报告

## wxWidgets

Dependency: wxWidgets
Version: 3.1.5，Bambu Lab fork revision `7c74234803455026ec7287827bd545e5a8ed79b7`，archive SHA256 `e5fc01faa0382ac34914ce92f41de7d189fa06d38e1887d50d0389ff8214a282`
Type: 源码编译的静态 GUI 依赖。
Reason: 项目使用 wxWidgets 3.1 开发接口，并采用 GTK 3 后端。
Root Cause: 目标默认 APT 提供 wxWidgets 3.0.4 开发包，项目需要 wxWidgets 3.1 接口。
Changes: 使用固定 Bambu Lab fork 构建 wxWidgets 3.1.5 GTK 3 静态库；将 wxWidgets 静态链接选项与 `SLIC3R_STATIC` 分开；按静态构建配置 `wxDEBUG_LEVEL=0` 并补充 X11 链接库。
Compatibility Impact: wxWidgets 代码静态进入应用程序；GTK 3、X11 与图形运行库由目标 APT 提供。其他构建继续使用各自的 `SLIC3R_WX_STATIC` 设置。
Upstream Status: 使用指定 fork revision；项目构建适配保存在 LoongArch64 分支。
Validation: 目标设备 `wx-config --version` 输出 `3.1.5`；正式版及 Public Beta 均通过 wxWidgets 配置检查。
Project-side Changes: `src/CMakeLists.txt`、`src/slic3r/CMakeLists.txt`。

## Boost

Dependency: Boost
Version: 1.71.0；目标默认 APT candidate `1.71.0-6kylin6k5k0.1`。
Type: APT 提供的头文件及共享库。
Reason: 目标 Kylin V10 SP1 运行环境提供 Boost 1.71.0，OpenMeshCraft 也使用该版本构建。
Root Cause: 项目与 OpenMeshCraft 原有 Boost 版本要求高于目标 APT 提供的版本；部分调用形式也需要兼容目标 Boost 接口。
Changes: 将最低 Boost 版本设为 1.71.0；Linux 下使用 FindBoost 模块；修正 `copy_file`、`weak_ptr`、`error_code` 与 `wxString::FromUTF8` 调用；将 Boost.Log 依赖声明在实际使用它的静态目标上。
Compatibility Impact: 使用目标 APT 的 Boost 1.71.0；后续 Boost 版本仍可使用相同接口构建。
Upstream Status: 项目侧兼容修改尚未提交 upstream。
Validation: 两个版本的 CMake 配置均找到 Boost 1.71.0；Public Beta ELF 动态依赖由目标 sysroot 解析，APT 模拟安装成功。
Project-side Changes: 根目录 `CMakeLists.txt`、`src/imgui/CMakeLists.txt`、`src/libslic3r/CMakeLists.txt`、`src/libslic3r/GCode/GCodeProcessor.cpp`、`src/libslic3r/utils.cpp`、`src/slic3r/GUI/Auxiliary.cpp`、`src/slic3r/GUI/AuxiliaryDataViewModel.cpp`、`src/slic3r/GUI/BitmapCache.cpp`、`src/slic3r/GUI/HttpServer.cpp`、`src/slic3r/GUI/MediaFilePanel.cpp`、`src/slic3r/GUI/PartSkipDialog.cpp`、`src/slic3r/GUI/Plater.cpp`、`src/slic3r/GUI/Printer/PrinterFileSystem.cpp`。

## CGAL

Dependency: CGAL
Version: 5.0.2；目标 APT package `libcgal-dev 5.0.2-3kylin0k1`。
Type: APT 提供的头文件依赖。
Reason: 目标构建使用发行版 CGAL headers 与 CMake modules。
Root Cause: 目标 package 布局需要从 multiarch 前缀发现 CGAL 配置与依赖设置模块。
Changes: 增加 `FindCGAL.cmake`，搜索 CGAL headers、版本文件与依赖模块，并创建 `CGAL::CGAL` 导入目标。
Compatibility Impact: 已安装 CMake package config 时继续优先使用 config package；未找到时使用模块发现路径。
Upstream Status: 本地查找模块；尚未提交 upstream。
Validation: CMake 配置确认找到 CGAL，并使用 header-only CGAL 模式构建。
Project-side Changes: 新增 `cmake/modules/FindCGAL.cmake`。

## OpenCASCADE

Dependency: OpenCASCADE / OCCT
Version: 7.6.0；archive SHA256 `28334f0e98f1b1629799783e9b4d21e05349d89e695809d7e6dfa45ea43e1dbc`。
Type: 源码编译的静态几何依赖。
Reason: 应用程序需要 OCCT 建模组件，目标架构构建需要使用 APT 提供的 FreeType 与 X11 headers。
Root Cause: OCCT 构建需要从 LoongArch64 multiarch 目录发现 FreeType 文件，系统 X11 headers 的默认搜索顺序也需要明确处理。
Changes: 显式发现 FreeType include 与 library 路径；将目标 X11 include 目录作为 `-idirafter`；增加 `FindOpenCASCADE.cmake` 搜索依赖前缀中的 OCCT headers 与组件库。
Compatibility Impact: OCCT 静态进入应用程序；FreeType 与 X11 运行库由目标 APT 提供。
Upstream Status: 构建适配保存在 LoongArch64 分支；尚未提交 upstream。
Validation: OCCT 所需组件通过 CMake 检查；Public Beta 应用程序完成链接与目标 X11 启动验证。
Project-side Changes: `deps/OCCT/OCCT.cmake`、新增 `cmake/modules/FindOpenCASCADE.cmake`。

## OpenMeshCraft

Dependency: OpenMeshCraft
Version: Bambu Lab revision `0e8d12c3df54804393593ab5d86c05caa340cbee`；archive SHA256 `632CD806CE932D6A1D76DF0E86ECF8BFC22480E76F80A638A28474F5262E2B9E`。
Type: 源码编译的静态 mesh boolean 依赖。
Reason: LoongArch64 构建启用 OpenMeshCraft boolean backend，并使用目标系统现有 Boost、TBB、Eigen、GMP 与 MPFR 开发文件。
Root Cause: fork 原有配置要求较高版本 Boost、oneTBB、Eigen 子项目及 x86 SIMD 设置，与目标 APT 版本和处理器架构不符。
Changes: 将 Boost 要求调整到 1.71.0；优先发现目标依赖前缀中的 Eigen、TBB、GMP 与 MPFR；LoongArch64 关闭 SSE2、AVX、AVX2、FMA；扩展配置目标的 TBB 链接信息。
Compatibility Impact: OpenMeshCraft 静态链接；目标依赖缺少时仍使用固定 archive 中的子项目源码。
Upstream Status: 基于 Bambu Lab fork；本地兼容修改尚未提交 upstream。
Validation: OpenMeshCraft boolean backend 在目标工具链下完成编译并进入最终链接。
Project-side Changes: `deps/OpenMeshCraft/0001-reduce-compat.patch`、`deps/OpenMeshCraft/CMakeLists.txt.in`、`deps/OpenMeshCraft/OpenMeshCraft.cmake`、`deps/OpenMeshCraft/OpenMeshCraftConfig.cmake.in`。

## oneTBB

Dependency: oneTBB
Version: 2021.5.0；archive SHA256 `83ea786c964a384dd72534f9854b419716f412f9d43c0be88d41874763e7bb47`。
Type: 源码编译的静态并行运行库。
Reason: LoongArch64 的 `tbb` 与 `tbbmalloc` 目标需要通过 oneTBB 架构检查；目标不使用 ITT instrumentation。
Root Cause: oneTBB 架构检查未列出 LoongArch64，该架构也不属于 ITT instrumentation 支持的处理器集合。
Changes: 在 TBB 与 TBBmalloc 的处理器检查中加入 LoongArch64，并使用 `0002-LoongArch-disable-ITT.patch` 关闭不适用的 ITT 分支。
Compatibility Impact: 补丁仅改变 oneTBB 对 LoongArch64 的构建判断，其他已支持架构保留原有检测路径。
Upstream Status: 本地补丁；尚未提交 upstream。
Validation: oneTBB、`tbbmalloc` 与应用程序均完成编译链接；最终运行依赖不包含 oneTBB 动态库。
Project-side Changes: `deps/TBB/TBB.cmake`、新增 `deps/TBB/0002-LoongArch-disable-ITT.patch`、`cmake/modules/FindTBB.cmake.in`、`src/libslic3r/CMakeLists.txt`。

## OpenVDB

Dependency: OpenVDB
Version: 8.2.0，patched revision `a68fd58d0e2b85f01adeb8b13d7555183ab10aa5`；archive SHA256 `f353e7b99bd0cbfc27ac9082de51acf32a8bc0b3e21ff9661ecca6f205ec1d81`。
Type: 源码编译的静态体素处理依赖。
Reason: 应用程序启用 OpenVDB，并使用目标 APT 的 Boost 1.71.0、TBB 与 Blosc 开发文件。
Root Cause: OpenVDB 外部构建的 Boost config 搜索方式无法稳定发现目标 multiarch Boost 目录。
Changes: 传入 Boost include 与 iostreams library 目录，关闭 Boost CMake config package 搜索，并在 LoongArch64 选择静态 OpenVDB 与 Blosc。
Compatibility Impact: OpenVDB 与 Blosc 静态进入应用程序；最终 ELF 不依赖外部 OpenVDB ABI。
Upstream Status: 本地构建配置适配；尚未提交 upstream。
Validation: CMake 找到 OpenVDB ABI 8；静态 OpenVDB 目标编译与应用程序链接通过。
Project-side Changes: 根目录 `CMakeLists.txt`、`deps/OpenVDB/OpenVDB.cmake`、`cmake/modules/FindOpenVDB.cmake`。

## DeviceWeb 构建依赖

Dependency: Node.js、pnpm、Rollup、esbuild、Lightning CSS 与 Tailwind CSS Oxide WASI package。
Version: Node.js 20.20.2，pnpm 10.12.1，`@rollup/wasm-node` 4.41.1，`esbuild-wasm` 0.28.2，`lightningcss-wasm` 1.30.1，`@tailwindcss/oxide-wasm32-wasi` 4.1.8。
Type: DeviceWeb 构建期工具、WASM package 与第三方源码补丁。
Reason: LoongArch64 的 Node.js 构建没有可用的上游 native extension package，DeviceWeb 仍需生成浏览器资源。
Root Cause: Rollup、esbuild、Lightning CSS 与 Tailwind Oxide 默认 native binding 没有可用的 LoongArch64 产物；Tailwind WASI worker 内部 handle 保持 Node.js 事件循环处于引用状态。
Changes: 使用 LoongArch64 Node.js 与 pnpm 缓存，选择 WASM package；向 `@tailwindcss/oxide-wasm32-wasi@4.1.8` 回移 worker handle `unref()` 修复，并在 `package.json` 与 `pnpm-lock.yaml` 记录 patch hash。
Compatibility Impact: 前端工具与 WASM package 不进入应用程序运行时依赖；补丁只修改 Tailwind WASI worker 路径，native package 路径保持原样。
Upstream Status: Tailwind worker 修复依据 upstream commit `6db2a6d637ed12489f9fd5a64ca394d934eb14f4`，已用于 4.1.18；4.1.8 通过本地 patch 回移。Node.js 与 pnpm 使用已发布版本。
Validation: Node.js 20.20.2、pnpm 10.12.1 下，LoongArch64 `device_page_build` 成功，Vite 生产构建生成 342 个模块。
Project-side Changes: `src/slic3r/GUI/DeviceWeb/CMakeLists.txt`、`src/slic3r/GUI/DeviceWeb/device_page/package.json`、`src/slic3r/GUI/DeviceWeb/device_page/pnpm-lock.yaml`、新增 `src/slic3r/GUI/DeviceWeb/device_page/patches/@tailwindcss__oxide-wasm32-wasi@4.1.8.patch`。

## 目标运行时 APT 依赖

Dependency: Kylin V10 SP1 LoongArch64 runtime package set。
Version: 使用目标默认 APT candidate；关键新增 package 包括 Boost 1.71.0-6kylin6k5k0.1、`libgmpxx4ldbl` 2:6.2.0+dfsg-4kylin0.1k3、NLopt 2.6.1-8kylin2、GLEW 2.1.0-4、GLFW 3.3.2-1、libhpdf 2.3.0+dfsg-1build1、OSMesa 20.0.8-0kylin3k26.4、`libkysdk-config` 与 `libkysdk-utils` 2.5.1.0-0k1.24。
Type: 通过 Debian `Depends` 声明的系统共享库。
Reason: 最终 ELF 动态使用 GTK 3、OpenGL/EGL、FFmpeg、Boost 与 Kylin 图形组件。
Root Cause: 应用程序需要系统共享库；LoongArch64 Debian package 必须声明这些库对应的目标 APT package。
Changes: `Depends` 列出运行所需 package；应用程序目录不包含替代版 glibc、libstdc++ 或动态系统共享库。
Compatibility Impact: 安装器可从目标设备默认 APT 源取得依赖；不替换目标 glibc 或 libstdc++。
Upstream Status: 使用目标设备配置的默认 APT 仓库候选版本。
Validation: APT 模拟安装从默认源解析成功，计划安装 16 个 package；Public Beta ELF 需要 `GLIBC_2.28` 与 `GLIBCXX_3.4.25`，目标 `libstdc++6` 导出所需版本；`ldd -r` 未输出缺失库或未解析符号。Public Beta `--help` 正常启动；安装目录中的 GUI 在目标 X11 会话运行至 25 秒 smoke test 结束。日志有 GVFS peer-to-peer fallback 与 GTK 窗口尺寸警告，进程未崩溃。
Project-side Changes: Debian control 文件 `Depends` 字段；LoongArch64 ELF 与目标运行检查。
