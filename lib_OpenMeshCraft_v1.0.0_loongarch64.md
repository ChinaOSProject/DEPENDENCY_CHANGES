# OpenMeshCraft 1.0.0 银河麒麟 V10 SP1 LoongArch64 适配报告

验证日期为 2026 年 11 月 27 日。源码仓库为 <https://github.com/bambulab/OpenMeshCraft>，固定提交为 `0e8d12c3df54804393593ab5d86c05caa340cbee`。本次为 Bambu Studio `v02.09.02.62` 构建静态 Boolean 后端。

## 构建与接入

构建产物在以下前缀，包含 `libOpenMeshCraft.a`、`libOpenMeshCraftBoolean.a`、`libshewchuk_predicates.a`、`include/OpenMeshCraft/OpenMeshCraftBoolean.h` 和 `lib/cmake/OpenMeshCraft/OpenMeshCraftConfig.cmake`。

```text
/mnt/chinaos-cache/laevatein/idle-builds/bambustudio-loongarch64/build/public-v02.09.02.62/omc-prefix
```

将此前缀放在主程序 `CMAKE_PREFIX_PATH` 首位，旧依赖前缀和 Kylin sysroot `usr` 随后。正在复用的主程序构建目录含有旧库的 CMake 缓存。重新配置时加上 `-U 'OPENMESHCRAFT_*' -U OpenMeshCraft_DIR`，再传入新 `CMAKE_PREFIX_PATH`。独立消费工程用已注入的旧缓存项验证了这一切换，三份静态库和头文件均解析到新前缀。

按顺序应用新版 `0001-OpenMeshCraft-coplanar-crash-fix.patch` 和 `0002-OpenMeshCraft-coincident-winding-labels-export.patch`，使用该版本的 wrapper 和根 CMake 覆盖。保留旧 LoongArch 构建的 Eigen 发现逻辑、GCC 8 顺序归约回退、Boost 1.71 版本门槛和 GCC 8 的空 `OMC_UNUSED` 定义。使用 Kylin GCC/G++ `8.3.0-6.lnd.vec.36.1`、CMake 3.16.3、Release、PIC、两个并发工作，关闭 SSE2、AVX、AVX2、FMA、测试目标和谓词生成器。原 sysroot 的 glibc 动态对象位于 `usr/lib/loongarch64-linux-gnu`，缓存内另建链接视图以满足其链接脚本中的 `/lib/loongarch64-linux-gnu` 路径。

复用旧依赖前缀中的静态 oneTBB 2021.5.0，以及 Kylin sysroot 中的 Eigen 3.3.7、Boost 1.71.0、GMP 6.2.0、MPFR 4.0.2。沿用缓存的 header-only fmt 10.2.1 和 parallel-hashmap 2.0.0。此次同时编译了随 OpenMeshCraft 源码自带的 Shewchuk 谓词对象，保留 Studio 的 `omc_shewchuk_compress` 重命名以避免 zlib 符号冲突。

`omc-prefix` 同级的 `omc-work` 中保存以下 diff。

- `bambu-openmeshcraft-cmake.patch` 可用于 Bambu Studio 源码根目录。已应用到缓存模板并安装验证，为 Linux 的 `OpenMeshCraftBoolean` 导入目标补齐 `TBB::tbb`、Threads、GMP 和 MPFR 的链接传递关系。
- `openmeshcraft-loongarch64-overlay.patch` 以 OpenMeshCraft 源码根目录为基准。其中 `external/CMakeLists.txt` 的基线是 Studio 新配方生成的 external 覆盖文件。实际配置记录见 `build-release/CMakeCache.txt` 和 `configure-release.log`。

## 接口、架构与 ABI

- 三份归档共 11 个对象均为 ELF64 LoongArch、`e_flags=0x3`（Kylin 工具链 `readelf` 显示 LP64）。本机 Debian 包管理架构标识为 `loongarch64`，Debian 官方包索引对应架构名为 `loong64`。
- 已安装头文件与新版 wrapper 头文件逐字节一致。导出两个 C linkage 入口 `bambu_omc_boolean` 和 `bambu_omc_boolean_operands`，旧 wrapper 仅导出前者。
- 三份静态归档通过 `--whole-archive` 和 `-z defs` 的共享链接验证。探针最高符号版本为 `GLIBC_2.27`、`GLIBCXX_3.4.21`、`CXXABI_1.3.11`、`GCC_4.5.0`。Kylin sysroot 提供 glibc 2.28 和 `libstdc++.so.6.0.25`。动态依赖包括 `libgmp.so.10`、`libmpfr.so.6`、`libstdc++.so.6`、`libgcc_s.so.1` 及 Kylin glibc 运行库。TBB 为静态链接。静态交付物无 SONAME。
- 仅链接 `OpenMeshCraftBoolean` 导入目标的独立 CMake 消费程序成功运行。通过非共面立方体的并、交、差，共享同一三角剖分的反向绕序接触面，完全重合的并与差，65 个重合操作数，以及非法输入。通过用例也在显式指定的 Kylin sysroot 动态加载器下运行通过。
- 共面回归用例仍有一项失败。两个单位立方体只沿 X 轴平移 0.5 时，并、交、差的预期体积分别为 1.5、0.5、0.5，实际为 1.166666667、0.833333333、0.166666667。旧 LoongArch 库的同一用例返回完全相同的体积和三角形计数。此偏差记录为既有问题，来源尚未归因。日志为 `smoke-run.log`、`legacy-coplanar.log`、`abi.log` 和 `full-archive-abi.log`。

## Debian 交付判定

Debian 官方包索引用 `openmeshcraft` 和 `shewchuk` 搜索所有套件的二进制和源码包名均无结果。Ubuntu primary archive 的 Launchpad API 对 `openmeshcraft` 和 `shewchuk-predicates` 的精确源码及二进制名称查询，以及 `openmeshcraft` 和 `shewchuk` 的二进制名称模糊查询，均返回空集合。查询结果保存在 `omc-work/package-search/`。Debian 的 `libopenmesh-dev` 与 Ubuntu 的同名包源自 `openmesh` 项目。Bambu Studio 将本次的 OpenMeshCraft wrapper、核心及 Shewchuk 谓词作为一个应用内部的静态构建接口交付。按项目第 6 节免于单独打包。本次交付 `omc-prefix`，复用的现有依赖无需重打包。用于 ABI 和行为验证的探针保存在专属缓存。

- Debian 包名索引 <https://packages.debian.org/search?keywords=openmesh>，源码包记录 <https://packages.debian.org/trixie/source/openmesh>
- Ubuntu 包名索引 <https://packages.ubuntu.com/>，源码包记录 <https://packages.ubuntu.com/source/noble/openmesh>，Launchpad archive API <https://api.launchpad.net/1.0/ubuntu/+archive/primary>
