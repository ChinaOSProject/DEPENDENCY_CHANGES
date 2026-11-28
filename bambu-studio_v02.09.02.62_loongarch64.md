# Bambu Studio 02.09.02.62 银河麒麟 V10 SP1 LoongArch64 适配报告

## 上游与构建目标

- 上游正式版为 [Bambu Studio 2.9.2 Public Release](https://github.com/bambulab/BambuStudio/releases/tag/v02.09.02.62)，标签 `v02.09.02.62` 指向 `23a33758933e712c6041e1f8f135f7d6b50004aa`。GitHub API 的发布记录为 `2026-11-24T14:00:16Z`，`prerelease` 为 `false`。
- 源码归档 SHA-256 为 `0c6b27d302ddb70fb34d3bd7e81aca001952a859945392978527672b82e12112`。
- 目标为银河麒麟桌面操作系统 V10 SP1 2503，`dpkg --print-architecture` 输出 `loongarch64`。使用发行版 GCC 8.3.0、CMake 3.16.3 和目标 sysroot，设置 `CMAKE_BUILD_TYPE=Release`、`BBL_RELEASE_TO_PUBLIC=1`、`BBL_INTERNAL_TESTING=0`、`SLIC3R_WX_STATIC=ON`、`SLIC3R_GTK=3`、`SLIC3R_STATIC=OFF`、`OPENVDB_USE_STATIC_LIBS=ON`、`USE_BLOSC=ON`。`FLATPAK=ON` 在这里让安装流程使用发行版的 FFmpeg 共享库。

## 源码与依赖

- 以正式版标签为基线，依次迁移 `debian10-v02.08.04.57` 和 `loong64-v02.08.04.57` 的适配。保留 GCC 8、Boost 1.71、目标 multiarch 库搜索及 wxWidgets GTK 3 静态链接所需修改。[完整源码适配补丁](patches/bambu-studio_v02.09.02.62_loongarch64.patch)已在干净的上游源码上通过 `git apply --check`。
- `std::filesystem` 在 GCC 8 上需要单独链接 `libstdc++fs`。构建先探测标准库直接链接能力，再探测补充库。具备直接链接能力的工具链走第一条路径。
- 缓存中保留 wxWidgets 3.1.5 Bambu Lab fork、OCCT 7.6.0、oneTBB 2021.5.0、OpenVDB 8.2.0、OpenCV 4.6.0 及本次未变化的静态依赖。Boost 1.71、GMP、MPFR、GTK 3、WebKitGTK 和 FFmpeg 由目标发行版提供。
- OpenMeshCraft 使用上游固定提交 `0e8d12c3df54804393593ab5d86c05caa340cbee` 及 Studio 2.9.2 的补丁与包装层重建。CMake 优先引用新包的头文件和静态库。独立验证、ABI、打包判定及与旧版行为一致的共面几何偏差详见 [OpenMeshCraft 适配报告](lib_OpenMeshCraft_v1.0.0_loongarch64.md)。
- DeviceWeb 使用已存在的 Node.js 20.20.2、pnpm 10.12.1、Rollup WASM、esbuild WASM、Lightning CSS WASM 和 Tailwind Oxide WASI。前端清单补丁仅应用于 LoongArch64 二进制构建目录。其他架构使用上游清单。Vite 保留上游 WebKit CSS 降级和检查。
- 下载与依赖复用缓存位于 `/mnt/chinaos-cache`。机械盘在写入 GCC 预编译头时阻塞，活跃编译、安装暂存和冒烟测试使用本机 NVMe 临时目录。

## Debian 打包依据

参考 Debian trixie 的 [prusa-slicer 2.9.2+dfsg-1](https://packages.debian.org/trixie/prusa-slicer) 与 [slic3r-prusa 打包源码](https://sources.debian.org/src/slic3r-prusa/2.9.2%2Bdfsg-1/debian/control/)。将切片器与工作所需的应用资源作为一个图形应用包交付。沿用 `bambu-studio` 包名以及已有 `/opt/bambu-studio` 安装布局，提供 `/usr/bin/bambu-studio`、桌面入口和 hicolor 图标。`Depends` 由最终 ELF 和目标发行版的共享库元数据生成。OpenMeshCraft 与 Shewchuk 谓词的交付判定见依赖报告。复用的发行版包及此前构建产物沿用已有版本。

## 成品与验证

- Debian 包为 [`bambu-studio_v02.09.02.62_loongarch64.deb`](https://github.com/ChinaOSProject/binaries/blob/main/bambu-studio/loongarch64/bambu-studio_v02.09.02.62_loongarch64.deb)，大小 153,037,168 字节，SHA-256 为 `5879e83fbc1e9cd1aefd5d67e3342d555f546e0777129f99bf49561235556dea`。
- 包内主程序为 ELF64 LoongArch，解释器为 `/lib64/ld.so.1`。最高符号版本为 `GLIBC_2.28`、`GLIBCXX_3.4.25`、`CXXABI_1.3.11` 和 `GCC_4.5.0`，由目标系统提供。使用目标 sysroot 对解包后的 `/usr/bin/bambu-studio` 执行 `ldd -r`，所有运行库和符号均已解析。
- `desktop-file-validate` 和 `apt-get -s install` 均成功。APT 模拟从当前麒麟软件源解析 16 项额外运行依赖，以及主程序包。
- DeviceWeb 的 Vite 生产构建与上游 WebKit CSS 检查通过，检查范围为 215 KB CSS。
- 解包后的 CLI 显示完整版本号。`--info` 识别 20 mm STL 立方体的 12 个三角面和封闭网格。A1 0.4 mm 喷嘴、0.20 mm Standard 与 Bambu PLA Basic 预设切片并导出有效的 3MF，包内 `Metadata/plate_1.gcode` 为 215,063 字节。虚拟 X11 中 CLI 的 GLEW 初始化失败，切片继续并跳过缩略图生成。
- Xvfb 图形冒烟测试通过。全新配置显示 820×660 的 Setup Wizard。临时配置选用 A1 预设后，1200×800 的编辑器主窗口显示并保持运行。测试脚本在验证窗口和进程后发送 `SIGTERM` 结束应用。

## 验证范围

验证覆盖原生运行时链接、图形启动和离线切片。真机打印及可选的 Bambu 网络插件未测试。OpenMeshCraft 在部分重叠、面共面的立方体上有既有布尔运算体积偏差，详见依赖报告。
