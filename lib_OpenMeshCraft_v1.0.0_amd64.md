# OpenMeshCraft Debian 10 依赖适配说明

GitHub: `https://github.com/ChinaOSProject/OpenMeshCraft`
Version: `1.0.0`
Type: C++ 静态库项目。

## Project changes

- `external/shewchuk-predicates/CMakeLists.txt`的最低 CMake 版本调整为 `3.13`。
- `SAABBTree`根据 `__cpp_lib_parallel_algorithm`选择 `std::reduce`或`std::accumulate`。
- `ExecutionCompat.h`根据标准库能力选择 execution policy；GCC 8 使用顺序算法。
- `FormatCompat.h`与 `OpenMeshCraft_ConfigureFmt.cmake`提供 `OMC::format`，GCC 8 使用 header-only `fmt`。
- GNU 编译器版本低于 9 时，`OMC_UNUSED`使用 GCC 属性形式，保留 `-Werror`检查。
- GCC 8 默认关闭 `PredicatesGenerator`，仍可通过 `OMC_BUILD_PREDICATES_GENERATOR=ON`手动启用。
- 使用 `IotaView`替代缺失的 `std::ranges::iota_view`，补充 OBJ/STL 读取所需的标准库头文件和旧编译器缺少的显式比较运算。

## Dependency changes

Dependency: `fmt`
Version: `10.2.1`
Type: header-only C++ 格式化库。
Reason: Debian 10 的 GCC 8.3.0 不提供 `<format>`。
Root Cause: 当前 libstdc++ 缺少 C++20 `std::format`。
Changes: CMake 支持固定版本 `fmt`下载，也支持通过 `OMC_FMT_SOURCE_DIR`提供本地源码；项目通过 `OMC::format`使用该库。
Compatibility Impact: MSVC 或具备可用 `std::format`的工具链可以继续使用标准库实现；GNU 工具链默认使用 `fmt`。
Upstream Status: 当前项目维护的兼容修改，尚未提交上游。
Validation: GCC 8.3.0 与 GCC 15.2.0 构建通过。
Project-side Changes: 增加 `cmake/OpenMeshCraft_ConfigureFmt.cmake`与`src/OpenMeshCraft/Utils/FormatCompat.h`。

## Validation

- Debian 10.13 sysroot、GCC 8.3.0、CMake 4.2.3：`OMC_BUILD_TEST=OFF`配置与构建通过，生成 `libOpenMeshCraft.a`。
- GCC 8.3.0 构建同时完成 `shewchuk_predicates`与 oneTBB。
- 当前系统 GCC 15.2.0 Debug 构建通过，`PredicatesGenerator`与 `OpenMeshCraft`均成功生成。
- 未进行 AArch64、LoongArch64、CGAL 测试目标及完整数据集运行验证。

当前项目仅提供库目标，未生成 deb 文件。
