# Bambu Studio v02.08.03.66 LoongArch64 依赖适配报告

## wxWidgets

Dependency: wxWidgets
Version: 3.1.5，Bambu Lab fork revision `7c74234803455026ec7287827bd545e5a8ed79b7`，archive SHA256 `e5fc01faa0382ac34914ce92f41de7d189fa06d38e1887d50d0389ff8214a282`
Type: 源码编译的静态 GUI 依赖
Reason: 目标构建使用项目所需的 wxWidgets 3.1 开发接口，并采用 GTK 3 后端。
Root Cause: 目标 APT 提供 wxWidgets 3.0.4 开发包；LoongArch64 目标需要 3.1.5 GTK 3 静态库。
Changes: 使用指定 Bambu Lab fork 源码构建 3.1.5；将 wxWidgets 静态链接选项与 `SLIC3R_STATIC` 分开；按静态构建配置 `wxDEBUG_LEVEL=0`，并补充 X11 链接库。
Compatibility Impact: wxWidgets 静态代码进入应用程序；GTK 3、X11 及图形运行库由目标 APT 提供。其他构建继续使用各自的 `SLIC3R_WX_STATIC` 设置。
Upstream Status: 使用指定 fork revision；本仓库适配保存在项目构建配置中。
Validation: 目标机 `wx-config --version` 输出 `3.1.5`；LoongArch64 应用程序完成编译，并在目标 Xorg 会话中连续运行 45 秒后由测试定时器结束。bubblewrap 日志记录了 GTK resize critical 与 GVFS fallback warning。
Project-side Changes: `src/CMakeLists.txt`、`src/slic3r/CMakeLists.txt`。

## Boost

Dependency: Boost
Version: 1.71.0，目标 APT candidate `1.71.0-6kylin6k5k0.1`
Type: APT 提供的头文件与共享库
Reason: LoongArch64 目标的 Kylin 运行环境提供 Boost 1.71.0；OpenMeshCraft 同样需要使用该版本构建。
Root Cause: 项目原先要求 Boost 1.83.0，部分源文件还使用了 Boost 1.71.0 中不可用的重载与类型推导形式。
Changes: 将项目最低 Boost 版本设为 1.71.0；Linux 下使用 FindBoost 模块；修正 `copy_file`、`weak_ptr`、`error_code` 和 `wxString::FromUTF8` 的调用；将 Boost.Log 依赖声明在实际使用它的静态目标上。
Compatibility Impact: 项目可使用目标 APT 的 Boost 1.71.0；较新 Boost 版本仍通过同一组可用接口构建。
Upstream Status: 项目侧兼容修改尚未提交 upstream。
Validation: 目标 CMake 找到 Boost 1.71.0；最终程序链接通过；`ldd -r` 未报告缺失共享库或未解析符号。
Project-side Changes: 根目录 `CMakeLists.txt`、`src/imgui/CMakeLists.txt`、`src/libslic3r/CMakeLists.txt`、`src/libslic3r/GCode/GCodeProcessor.cpp`、`src/libslic3r/utils.cpp`、`src/slic3r/GUI/Auxiliary.cpp`、`src/slic3r/GUI/AuxiliaryDataViewModel.cpp`、`src/slic3r/GUI/BitmapCache.cpp`、`src/slic3r/GUI/HttpServer.cpp`、`src/slic3r/GUI/MediaFilePanel.cpp`、`src/slic3r/GUI/PartSkipDialog.cpp`、`src/slic3r/GUI/Plater.cpp`、`src/slic3r/GUI/Printer/PrinterFileSystem.cpp`。

## CGAL

Dependency: CGAL
Version: 5.0.2，目标 APT 包 `libcgal-dev 5.0.2-3kylin0k1`
Type: APT 提供的头文件依赖
Reason: 目标环境使用发行版 CGAL headers 与 CMake modules。
Root Cause: 目标包布局需要从 multiarch 前缀发现 CGAL 配置与依赖设置模块。
Changes: 增加 `FindCGAL.cmake`，按 CMake 前缀搜索 CGAL headers、版本文件和依赖模块，并创建 `CGAL::CGAL` 导入目标。
Compatibility Impact: 已安装 CMake package config 时继续优先使用 config package；未找到时启用模块发现路径。
Upstream Status: 本地查找模块；尚未提交 upstream。
Validation: 配置日志确认找到 CGAL，并以 header-only CGAL 模式构建。
Project-side Changes: 新增 `cmake/modules/FindCGAL.cmake`。

## OpenCASCADE

Dependency: OpenCASCADE / OCCT
Version: 7.6.0，使用项目固定的 upstream archive，SHA256 `28334f0e98f1b1629799783e9b4d21e05349d89e695809d7e6dfa45ea43e1dbc`
Type: 源码编译的静态几何依赖
Reason: 应用程序需要 OCCT 建模组件；目标架构的依赖构建需要明确使用 APT 的 FreeType 与 X11 headers。
Root Cause: OCCT 构建时无法从 LoongArch64 multiarch 目录稳定发现 FreeType 文件，系统 X11 headers 的默认搜索顺序也会与第三方 include 目录发生冲突。
Changes: 显式发现 FreeType include/library 路径；将目标 X11 include 目录作为 `-idirafter`；增加 `FindOpenCASCADE.cmake` 以搜索项目依赖前缀中的 OCCT headers 与组件库。
Compatibility Impact: OCCT 以静态依赖进入应用程序；FreeType、X11 运行库由目标 APT 提供。
Upstream Status: 构建适配保存在本仓库；尚未提交 upstream。
Validation: OCCT 所需组件均通过 CMake 检查；最终链接与目标机图形启动检查通过。
Project-side Changes: `deps/OCCT/OCCT.cmake`、新增 `cmake/modules/FindOpenCASCADE.cmake`。

## OpenMeshCraft

Dependency: OpenMeshCraft
Version: Bambu Lab revision `0e8d12c3df54804393593ab5d86c05caa340cbee`，archive SHA256 `632CD806CE932D6A1D76DF0E86ECF8BFC22480E76F80A638A28474F5262E2B9E`
Type: 源码编译的静态 mesh boolean 依赖
Reason: LoongArch64 构建启用 OpenMeshCraft boolean backend，并使用目标系统现有 Boost、TBB、Eigen、GMP、MPFR 开发文件。
Root Cause: fork 的配置要求 Boost 1.78.0，并默认准备 oneTBB、Eigen 子项目及 x86 SIMD 设置；这些设置与目标依赖版本及处理器架构不符。
Changes: Boost 要求调整到 1.71.0；优先发现目标依赖前缀中的 Eigen、TBB、GMP、MPFR；LoongArch64 关闭 SSE2、AVX、AVX2、FMA；扩展配置目标的 TBB 链接信息。
Compatibility Impact: OpenMeshCraft 保持静态链接；默认依赖缺失时仍使用固定 upstream archive 中的子项目源码。
Upstream Status: 基于 Bambu Lab fork；本仓库的 Boost 与 LoongArch64 构建适配尚未提交 upstream。
Validation: OpenMeshCraft boolean backend 在目标工具链下完成编译并进入最终链接。
Project-side Changes: `deps/OpenMeshCraft/0001-reduce-compat.patch`、`deps/OpenMeshCraft/CMakeLists.txt.in`、`deps/OpenMeshCraft/OpenMeshCraft.cmake`、`deps/OpenMeshCraft/OpenMeshCraftConfig.cmake.in`。

## oneTBB

Dependency: oneTBB
Version: 2021.5.0，archive SHA256 `83ea786c964a384dd72534f9854b419716f412f9d43c0be88d41874763e7bb47`
Type: 源码编译的静态并行运行库
Reason: LoongArch64 的 `tbb` 与 `tbbmalloc` 目标需要被 oneTBB CMake 配置接受，ITT instrumentation 不用于本目标。
Root Cause: oneTBB 架构检查未列出 LoongArch64；该架构不属于 ITT instrumentation 支持的处理器集合。
Changes: 在 TBB 与 TBBmalloc 的处理器检查中加入 LoongArch64，并使用 `0002-LoongArch-disable-ITT.patch` 关闭不适用的 ITT 分支。
Compatibility Impact: 补丁只影响 oneTBB 对 LoongArch64 的构建判断；其他已支持架构保留原有检测路径。
Upstream Status: 本地补丁；尚未提交 upstream。
Validation: oneTBB、`tbbmalloc` 目标和应用程序均完成编译与链接。
Project-side Changes: `deps/TBB/TBB.cmake`、新增 `deps/TBB/0002-LoongArch-disable-ITT.patch`、`cmake/modules/FindTBB.cmake.in`、`src/libslic3r/CMakeLists.txt`。

## OpenVDB

Dependency: OpenVDB
Version: 8.2 patched revision `a68fd58d0e2b85f01adeb8b13d7555183ab10aa5`，archive SHA256 `f353e7b99bd0cbfc27ac9082de51acf32a8bc0b3e21ff9661ecca6f205ec1d81`
Type: 源码编译的静态体素处理依赖
Reason: 应用程序启用 OpenVDB 功能，并使用目标 APT 的 Boost 1.71.0、TBB 与 Blosc headers/libraries。
Root Cause: OpenVDB 外部构建的 Boost config 搜索方式无法稳定找到目标多架构 Boost 目录。
Changes: 直接传入 Boost include 目录与 iostreams library 目录，关闭 Boost CMake config package 搜索，并在 LoongArch64 选择静态 OpenVDB 与 Blosc。
Compatibility Impact: OpenVDB 与 Blosc 静态进入应用程序；运行时不依赖外部 OpenVDB ABI。
Upstream Status: 本地构建参数适配；尚未提交 upstream。
Validation: CMake 找到 OpenVDB ABI 8；静态 OpenVDB 目标编译与应用程序链接通过。
Project-side Changes: 根目录 `CMakeLists.txt`、`deps/OpenVDB/OpenVDB.cmake`。

## DeviceWeb 构建依赖

Dependency: Node.js、pnpm、Rollup、esbuild、Lightning CSS
Version: Node.js 20.20.2（SHA256 `f2511b997f50d80dc3495c1eb44fb576f3c094eb35a04cf471080b0fc651a77c`）、pnpm 10.12.1（SHA256 `b276da51dc8ca5b0d3ee3371695b50fc8b3244b281b091c63a3f082a88dadeb9`）、`@rollup/wasm-node` 4.41.1、`esbuild-wasm` 0.28.2、`lightningcss-wasm` 1.30.1
Type: 仅用于 DeviceWeb 构建的工具与 WASM 包
Reason: LoongArch64 上游 Node 原生扩展包缺少目标架构构建，DeviceWeb 仍需生成浏览器资源。
Root Cause: Rollup、esbuild 与 Lightning CSS 默认依赖的 native binding 没有可用的 LoongArch64 产物。
Changes: 为 LoongArch64 选择 Node.js/pnpm 缓存；构建阶段使用 WASM 包、TypeScript transpile plugin，并关闭 Vite 的 esbuild 与 CSS minify 路径。
Compatibility Impact: 这些工具不会进入 Debian 运行依赖；`package.json` 中的 WASM overrides 当前作用于所有执行此分支前端构建的架构。
Upstream Status: 使用已发布的 npm 包；Node.js/pnpm LoongArch64 可执行文件由构建机外部缓存提供，缓存下载来源未记录，二进制校验值已记录在 Version 字段。
Validation: DeviceWeb `device_page_build` 目标完成；静态资源安装到应用程序 resources 目录。
Project-side Changes: `src/slic3r/GUI/DeviceWeb/CMakeLists.txt`、`src/slic3r/GUI/DeviceWeb/device_page/package.json`、`pnpm-lock.yaml`、`vite.config.ts`。

## 目标运行时 APT 依赖

Dependency: Kylin V10 SP1 LoongArch64 运行库
Version: 模拟安装中由目标默认 APT 源提供的新增包为 Boost 1.71.0-6kylin6k5k0.1、NLopt 2.6.1-8kylin2、GLEW 2.1.0-4、GLFW 3.3.2-1、libhpdf 2.3.0+dfsg-1build1、OSMesa 20.0.8-0kylin3k26.4。
Type: 系统共享库，通过 Debian `Depends` 声明。
Reason: 最终 ELF 动态依赖 GTK 3、OpenGL/EGL、FFmpeg、Boost 与其它 Kylin 系统共享库。
Root Cause: 默认 APT 可提供所需共享库；LoongArch64 包还需要显式声明所有直接 ELF 依赖的 Debian 包名。
Changes: Debian control 文件列出 libc6、libstdc++6、libgcc1、Boost、GTK 3、OpenGL、FFmpeg、WebKitGTK、X11、Wayland、OSMesa 等运行包；未将这些系统共享库复制进应用程序目录。
Compatibility Impact: 安装器从目标已配置 APT 源解析缺少的软件包；应用程序没有高于 glibc 2.28 的符号需求。
Upstream Status: 使用目标设备配置的 APT 仓库候选版本。
Validation: `apt-get -s --no-install-recommends install` 解析成功；`ldd -r` 未发现缺失库或未解析符号；ELF 最高符号需求为 GLIBC_2.27、GLIBCXX_3.4.25。
Project-side Changes: Debian 包控制文件 `Depends` 字段；LoongArch64 运行时检查。
