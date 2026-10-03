# build-deps v2026.910.121303：LoongArch64 ABI v0 依赖适配

完成日期：2026-10-03（Asia/Shanghai）。仅依赖项目，用于 Sunshine v2026.914.233613；不生成无实际用途的应用 `.deb`。

## 结果与交接

完整可用 prefix：`/tmp/sunshine-build-deps-20261003/prefix`，含 `include/`、`lib/libavcodec.a`、`libavutil.a`、`libswscale.a`、`libcbs.a`、`libx264.a`、`libx265.a`。旧 libva2.7 头文件、可迁移 pkg-config、许可证、`BUILD.md` 和 `SOURCE_VERSIONS.json` 一并提供。

H264/HEVC 软件编码 -> CBS解析/重写 -> 软件解码与 RGB/YUV转换均通过实际运行验证。六个静态库共491个ELF对象全部为LoongArch ABI v0、flags `0x3`。完整PIE链接/运行程序使用 `/lib64/ld.so.1`，最高GLIBC要求 `2.28`，C++/GCC运行库静态链接，没有新增动态GLIBCXX要求。

当前 AMD OLAND 驱动只读查询未公布 H264/HEVC 编码能力；硬件编码没有执行验证。该限制与已构建、注册的FFmpeg VAAPI编码器区分记录，应用仍可使用已验证的软件编码。

## 来源与实际版本

当前被适配仓库 GitHub 地址：[LizardByte/build-deps](https://github.com/LizardByte/build-deps)。独立本地源码位于 `/home/laevatein/ChinaOSProject/sunshine-build-deps`，upstream SSH地址 `git@github.com:LizardByte/build-deps.git`。

| 项目 | 实际版本/锁定方式 | 来源 commit |
| --- | --- | --- |
| build-deps | 官方标签 [v2026.910.121303](https://github.com/LizardByte/build-deps/tree/v2026.910.121303)；SSH ls-remote 与 fetch 确认标签指向本次锁定commit | a1fe2841cbc0d8c4501a1006d2d1cb88f219cc8c |
| FFmpeg/FFmpeg | 官方 release/9.0 的锁定源码，RELEASE文件为9.0.1；构建运行版本字符串实际为 `bf1b838` | bf1b838f2ab88b4f8fd83443325c782ea0e0f7fa |
| x264 | 生成的 `X264_POINTVER` 为 `0.165.3222 b35605a`，API165，HEAD历史计数3222 | b35605ace3ddf7c1a5d67a2eb553f034aef41d55 |
| x265_git | 官方镜像标签4.1与锁定HEAD一致，API build215 | 1d117bed4747758b51bd2c124d738527e30392cb |
| libva | 原版官方标签2.7.0，VA API/pkg-config版本1.7.0；无需源码修改 | c2be378312b0a17c796509defae42afba7351272 |
| SVT-AV1 | 保留官方锁定gitlink，本次关闭构建 | 9292ec8e32bce26f781f277ec8739b53426c4300 |

FFmpeg库版本：libavcodec63.1.101、libavutil61.1.101、libswscale10.1.101。上游 `project(VERSION 0.0.0)` 是配方占位值，本报告采用已确认的真实官方发布标签，不把0.0.0伪造为软件完整版本。

FFmpeg、x264、x265 的源码从 Sunshine 已下载且锁定的子模块 `git clone --no-hardlinks --no-checkout` 至独立仓库，随后明确checkout锁定HEAD。没有硬链接，没有并发修改应用源码；原始子模块保持干净。x264独立副本获取必要历史，x265独立副本获取官方4.1标签，以正确生成版本元数据。

## 分支传播与单点提交

传播顺序为 `v2026.910.121303(a1fe284)` -> `debian10` -> `loong64`。官方默认上游分支实际名为master，本次严格以稳定标签commit为基线，不让架构分支独立追踪移动中的upstream。

- debian10：`1cb1150733e3656eea46d9c21afd3b2f667960fa`，只含两个通用提交。
- loong64：`0e1eff16622bf8e70f6a637136a60e38208feb19`，当前checkout，继承全部通用修改。
- 架构提交没有反向合入debian10。后续通用debian10修改继续传播至loong64。

| 提交 | 分支来源 | 单一逻辑及原因 |
| --- | --- | --- |
| `a0702a2251dcba3972d00c93620d3c69485f17be` | debian10 | 添加默认OFF的 `BUILD_FFMPEG_LIBVA_SYSTEM`；选择已有libva，避免旧设备被迫升级到默认2.24.1。默认仍使用官方新库构建流程。 |
| `1cb1150733e3656eea46d9c21afd3b2f667960fa` | debian10 | 保存调用方 `FFMPEG_EXTRA_CONFIGURE` 并在配方默认选项之后应用；原先 `--disable-all` 覆写了前方指定的解码器/解析器，导致测试所需选项失效。默认无附加选项时行为保持原配方。 |
| `0e1eff16622bf8e70f6a637136a60e38208feb19` | loong64 | CBS与x264识别 `loongarch64`/`loong64`，分别映射到FFmpeg的loongarch目录与x264的loongarch64；原架构映射保持。 |

源码差异仅4个CMake文件、17行新增。没有独立依赖源码补丁、重构、格式化或环境路径进入commit。最终 `git status` 干净，`git diff --check` 通过。

三个独立提交已导出至 `/tmp/sunshine-build-deps-20261003/patches/`，另有可直接 `git am` 的合并mbox：`/tmp/sunshine-build-deps-20261003/build-deps-v2026.910.121303-loongarch64.mbox`。主Agent最终归档附带这些补丁；没有ChinaOS源码fork，不向LizardByte组织推送。

## 官方原有补丁

只在隔离的 `/work/build/FFmpeg/` 构建副本按官方配方应用：

- `patches/FFmpeg/FFmpeg/cbs/01-explicit-intmath.patch`：CBS源码显式引入intmath，支持官方拆出的CBS子集。
- `patches/FFmpeg/FFmpeg/cbs/02-remove-register.patch`：删除register存储说明符，支持C++消费者；本次C++测试实际包含get_bits/CBS H264/H265头文件并编译成功。
- `patches/FFmpeg/x265_git/01-cmake-minimum_required.patch`：官方CMake最低版本修正。
- `patches/FFmpeg/x265_git/02-uint8_t-undefined.patch`：官方uint8_t声明修正。

上述补丁检查和应用均成功。没有对原始上游submodule打补丁，也没有额外修改FFmpeg/x264/x265/libva源码；因此没有触发额外依赖源码委派。

## 专用镜像与隔离环境

项目专用镜像：`localhost/sunshine-deps:2026.910.121303-buster-loongarch64`，image ID `275a62bdbeb10ba1b2e102d8b34fa738f8779b227f3e9f9b7645f6872632f600`。

由已commit的 Sunshine buster 项目镜像 `localhost/sunshine-build:2026.914.233613-buster-loongarch64`（e138bf58ac68）派生；独立容器 `sunshine-deps-20261003`。基础系统为Loongnix Debian buster、GLIBC2.28。Rootless Podman6.1.2、VFS、pasta，必须 `--security-opt seccomp=unconfined`；采用run/exec安装/commit方式，避免5.4内核overlay userxattr错误。

所有编译和链接测试均在本项目专用容器内，没有宿主apt/pip/npm或工具链全局安装，没有控制或停止应用、scratch、zig容器。最终依赖 Agent 位于Herdr w2:p2，状态通过指定文件交接。

只读工具链挂载：

- GCC13.4.0：`/home/laevatein/node-debian10-build/toolchain/gcc-13`。
- Python3.10：`/home/laevatein/node-debian10-build/toolchain/python-3.10`。
- CMake3.27.9：`/home/laevatein/src/zig-0.16.0-container/cmake-build` -> `/opt/cmake-build`，其源码Modules目录另挂载至 `/work/cmake-3.27.9`。
- 本仓库挂载 `/src:ro`；所有可修改源副本、生成文件和安装prefix位于本任务专有 `/work`（宿主 `/tmp/sunshine-build-deps-20261003`）。

容器内补充autoconf、automake、libtool/libtool-bin、ninja-build、libxext-dev、libxfixes-dev；apt使用 `https://pkg.loongnix.cn/loongnix DaoXiangHu-stable`。

GCC13直接调用旧as的最初探针因不识别 `-mabi=lp64d` 失败。复用该GCC13原有 `/home/laevatein/node-debian10-build/build-tools/as-loong64`，只在独立容器复制到同路径。该已有wrapper把 `-mabi=lp64d` 转为Loongnix as支持的 `-mabi=lp64`；实际使用旧Loongnix binutils2.31.1生成flags0x3，而非改写现代0x43对象的标记。链接显式指定 `/lib64/ld.so.1`。C/C++探针实际运行成功；静态C++探针仅要求GLIBC2.27。

## libva旧基座与AMD路径

容器预装libva-dev为2.10.0，目标已有运行库为2.7.0。若直接用较新头文件可能引入目标缺少的VA API，因此无需修改源码地获取官方libva2.7.0，并在本容器配置：

```sh
source /work/env.sh
cd /work/libva-source-2.7.0
CFLAGS='-O2 -fPIC' ./autogen.sh --prefix=/work/libva-2.7 \
  --enable-shared --disable-static --enable-drm --enable-x11 \
  --disable-glx --disable-wayland
make -j2
make install
```

这是为FFmpeg提供目标2.7基座的开发头文件/探测库，不要求应用交付私有新版libva。prefix/include/va包含该基座头文件，应用应优先使用，以防容器较新头文件提高API需求。FFmpeg所需全部37个VA函数逐项与目标已有libva/libva-drm/libva-x11运行库导出比对，缺失0个；详见 `libva-symbol-audit.json`。

只读AMD能力查询采用目标已有libva2.7.0-2kylin0k1.0 runtime副本、目标已有libdrm2/libdrm-amdgpu1 2.4.101-2kylin9k0.2 runtime副本和只读Mesa驱动目录，设备为 `/dev/dri/renderD128`，临时测试容器均 `--rm`。最初复用容器旧DRM时目标Mesa驱动缺少 `amdgpu_cs_query_reset_state2`；改用目标原有flags0x3 DRM runtime后成功初始化，不安装/替换宿主库。

实际查询：VA-API1.7；Mesa Gallium20.0.8 AMD OLAND，DRM3.35.0，kernel5.4.18-168-generic，LLVM8.0.1。profile0/1为MPEG2Simple/Main，仅entrypoint1（VLD）；profile-1仅entrypoint10（VideoProc）。H264与HEVC编码profile均为0。只做能力读取，没有编码硬件帧，因此不得把硬件编码称为已验证。

## 构建配置与步骤

锁定源码副本准备完毕后，本仓库配方通过以下外部脚本配置；临时脚本没有提交进源码仓库：

```sh
#!/bin/bash
set -euo pipefail
source /work/env.sh
cmake -S /work/source -B /work/build -G "Unix Makefiles"   -DCMAKE_BUILD_TYPE=Release   -DCMAKE_C_FLAGS="-fPIC -ffunction-sections -fdata-sections"   -DCMAKE_CXX_FLAGS="-fPIC -ffunction-sections -fdata-sections"   -DCMAKE_EXE_LINKER_FLAGS="$LDFLAGS"   -DCMAKE_INSTALL_PREFIX=/work/prefix   -DFFMPEG_INSTALL_PREFIX=/work/prefix   -DPARALLEL_BUILDS=2   -DBUILD_ALL=OFF -DBUILD_FFMPEG=ON   -DBUILD_FFMPEG_AMF=OFF -DBUILD_FFMPEG_MF=OFF   -DBUILD_FFMPEG_NV_CODEC_HEADERS=OFF -DBUILD_FFMPEG_CUDA_LLVM=OFF   -DBUILD_FFMPEG_VULKAN=OFF -DBUILD_FFMPEG_SVT_AV1=OFF   -DBUILD_FFMPEG_LIBVA=ON -DBUILD_FFMPEG_LIBVA_SYSTEM=ON   -DBUILD_FFMPEG_X264=ON -DBUILD_FFMPEG_X265=ON   -DENABLE_ASSEMBLY=OFF   "-DFFMPEG_EXTRA_CONFIGURE=--enable-pic;--enable-decoder=h264,hevc;--enable-parser=h264,hevc;--extra-libs=/home/laevatein/node-debian10-build/toolchain/gcc-13/lib/libstdc++.a;--extra-ldflags='$LDFLAGS'"
```

然后执行：

```sh
source /work/env.sh
bash /work/configure.sh
cmake --build /work/build --parallel 2
cmake --install /work/build
bash /work/validate.sh
```

`env.sh` 使用上述只读GCC13/CMake/Python，`PKG_CONFIG_PATH=/work/libva-2.7/lib/pkgconfig`，编译工具的 `LD_LIBRARY_PATH` 仅供其自身运行；链接选项为 `-Wl,--dynamic-linker=/lib64/ld.so.1 -static-libgcc -static-libstdc++`。

保留：CBS、全部官方BSF、libx264、libx265、mpeg2video、h263p、h264/hevc/mpeg2 VAAPI、h264_v4l2m2m、avcodec/avutil/swscale。新增H264/HEVC解码器及解析器用于真实运行验证。

关闭项：AMF、Media Foundation、CUDA/NVENC、Vulkan、SVT-AV1；AV1 VAAPI因旧VA API1.7自动检测关闭。没有关闭必要的软件H264编码或大量基本功能来规避编译问题。

FFmpeg的LSX/LASX intrinsics检测失败，因为GCC13不支持 `-mlsx/-mlasx`，自动选择portable C；x264的独立汇编检测通过，构建并实际选择LSX/LASX。x265采用官方portable C++路径，`ENABLE_ASSEMBLY=OFF`，本次为8-bit静态库，没有提供10/12-bit x265多库组合。

首次FFmpeg x265 pkg-config探测失败：x265.pc包含绝对静态libstdc++.a，configure将其归到普通flags并放在-lx265之前，造成C++符号未解析。通过现有FFMPEG_EXTRA_CONFIGURE的 `--extra-libs` 在末尾追加同一旧ABI静态C++库修正；没有修改x265源码。最终prefix的.pc规范为相对pcfiledir前缀、显式 `-Wl,-Bstatic -lstdc++ -Wl,-Bdynamic` 的私有运行库参数，并补齐x264.pc。应用直接使用C++链接器和静态运行库选项；pkg-config --static消费者另由交付.pc明确限定C++库静态选择。补充链接检查发现GCC13不会自动将显式-lstdc++转换为静态，所以已在仓库外交付元数据中修正，六个归档未变。libcbs.pc保留官方0.0.0占位字段，真实CBS身份按同一FFmpeg锁定源码和真实build-deps发布标签记录，不捏造独立版本。

## ABI与库清单

逐一解析GNU ar成员ELF头，校验ELF64、小端、EM_LOONGARCH=258、e_flags=3；不是只抽样第一个成员。没有0x43库混入。

| 归档 | ELF对象数 | flags | bytes | SHA256 |
| --- | ---: | --- | ---: | --- |
| libavcodec.a | 223 | 0x3 | 62036908 | `43bc30c14d695480e026ee002c9740e211af2e62ddaeba60c279098683f4a00f` |
| libavutil.a | 97 | 0x3 | 8949132 | `2b12d2a5fc902cc77c964230b76ed2cccbab6601a1ea5771339d0d491ecdcf5a` |
| libcbs.a | 12 | 0x3 | 1445676 | `ec53842d68188f1b627ae28273832571f845cad3c62f0381d76cb46621dab424` |
| libswscale.a | 31 | 0x3 | 14721766 | `fd0a4ff65dc693f1b59d00baf3d9ada00d9238df6867f9e14751ad7cf7764443` |
| libx264.a | 77 | 0x3 | 7068184 | `2ae49a55dfb825a7c07977263ba9d74d33f46557f86ebcaecdcd7d7f26f0149d` |
| libx265.a | 51 | 0x3 | 6435784 | `4548bd1c33072bba06ff7525ace33d9564429caa14321985b43d4a7610d01b1c` |

最终核查基准为通过交付pkg-config --static元数据链接并实际运行的 `codec-pkgconfig-link`（SHA256 `ba1a72cdb1b272d28a8858a04f92fa410d610de6c68732d647cf2652f8a1a0d7`）：flags0x3、loader `/lib64/ld.so.1`、GLIBC最大2.28。其直接 `DT_NEEDED` 清单为：`libatomic.so.1`、`libpthread.so.0`、`libm.so.6`、`libdl.so.2`、`librt.so.1`、`libnuma.so.1`、`libva.so.2`、`libva-drm.so.2`、`libc.so.6`、`ld.so.1`；C++/GCC运行库静态。`libdrm.so.2` 由 `libva-drm.so.2` 传递加载。正式报告、prefix/BUILD.md、SOURCE_VERSIONS.json的final_test_elf及ABI_AUDIT.json统一使用此最终清单；元数据另保留较早直接归档链接测试 `codec-smoke` 的独立清单。

目标已有 `libatomic1:loongarch64 8.3.0-6.lnd.vec.36.1`，实际文件 `/usr/lib/loongarch64-linux-gnu/libatomic.so.1.2.0`（SHA256 `2cd2429b88d7fab01fa99603efd88e9b03e678b5ec7f971fc2ab3cf01bdf239d`）提供SONAME `libatomic.so.1`。它为LoongArch ABI v0、flags0x3，最高GLIBC要求2.27，仅依赖libpthread.so.0和libc.so.6，导出LIBATOMIC_1.0、1.1、1.2。最终ELF和六个归档的未解析atomic符号数均为0，最终ELF没有LIBATOMIC版本要求，缺失符号数0；该NEEDED来自交付 `libavutil.pc` 的 `Libs` 字段显式 `-latomic`。

已在独立依赖容器用 `LD_LIBRARY_PATH=/work/target-runtime-atomic:/work/target-runtime-libva:/work/target-runtime-drm` 和 `LD_BIND_NOW=1` 运行最终ELF。ldd明确解析到目标原有libatomic副本、libva2.7和DRM副本；即时符号解析和全部软件编码/CBS/解码测试通过，目标现有libatomic满足本次最终ELF的符号与ABI需求，无需新增私有新版libatomic或抬高宿主floor。原补充链接日志为 `pkgconfig-static-link-check.log`，最终运行与解析证据为 `final-validation.log`、`final-elf-ldd.log`、`final-elf-readelf.log`、`target-libatomic-readelf.log` 和 `target-libatomic-audit.json`。

## 实际运行验证

仓库外 `codec-smoke.cpp` 以C++17编译，实际包含导出的config/get_bits/CBS H264/H265头文件，使用完整静态库组、PIE和静态C++运行库链接；CBS归档置于首位，实际验证拆出的libcbs。

最终pkg-config消费者重用同一测试源码，已加载目标libatomic/libva/DRM实际重跑。合成输入为160x96 RGB24，由本libswscale转成YUV420P，分别软件编码，再用本libcbs解析和重写包，最后由本libavcodec解码。检查帧格式/尺寸、全部帧数、参数集数量和解码luma PSNR>28dB，最终运行结果如下：

| 编码器 | 完整解码帧 | packet | CBS units | parameter sets | 码流bytes | 最低luma PSNR |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| libx264 | 24/24 | 24 | 29 | 4 | 9747 | 52.982dB |
| libx265 | 8/8 | 8 | 12 | 3 | 4193 | 53.499dB |

x264运行日志明确 `using cpu capabilities: LSX LASX`。还检查h264_vaapi、hevc_vaapi、mpeg2_vaapi、h264_v4l2m2m、mpeg2video、h263p编码器注册。该160x96功能测试不能代表1080p/4K实时编码性能。

证据均在 `/tmp/sunshine-build-deps-20261003/`：`validation.log`、`abi-audit.json`、`libva-symbol-audit.json`、`vaapi-capabilities.log`、`abi-probe.log`、`simd-probes.json`、`configure.log`、`build-4.log`；首次链接失败、较新DRM符号缺失的原始日志也保留，没有用 `|| true` 或跳过真实构建失败。测试程序、日志、构建脚本、镜像材料和产物都在源码仓库之外。

## 应用使用、限制与交付边界

主Agent/应用Agent已能读取prefix。应用容器可将其复制到已有/work挂载位置，指定对应容器路径的FFMPEG_PREPARED_BINARIES；优先prefix/include/va，复用GCC13原有as-loong64，并显式静态链接C++/GCC运行库及旧loader。系统运行依赖包括目标已有libatomic和VA/DRM/NUMA栈；X11消费者另用目标已有VA-X11/X11栈，实际最终ELF清单见前文，无新增私有libva或libatomic。

限制：FFmpeg自身LSX/LASX关闭，x265 scalar C++且仅8-bit；未构建SVT软件AV1，没有CUDA/NVENC/Vulkan/AMF；当前AMD驱动不公布H264/HEVC编码能力，实际硬件编码未验证；未测试应用完整启动/采集或高分辨率实时性能，该阶段由应用Agent负责。

依赖Agent未单独推送源代码、二进制或报告，也未人为打包依赖应用deb。主Agent统一归档本报告及适配补丁，应用最终打包/发布由主Agent处理。此报告始终位于源码仓库外。

## 可复现最小差异

附带以下独立提交补丁；从官方锁定 commit 创建 debian10 后按编号应用前两项，再创建 loong64 应用第三项：

- [0001-Allow-using-system-libva-for-FFmpeg-builds.patch](patches/lib_build-deps_v2026.910.121303_loongarch64/0001-Allow-using-system-libva-for-FFmpeg-builds.patch)
- [0002-Apply-caller-FFmpeg-options-after-recipe-defaults.patch](patches/lib_build-deps_v2026.910.121303_loongarch64/0002-Apply-caller-FFmpeg-options-after-recipe-defaults.patch)
- [0003-Recognize-LoongArch64-for-CBS-and-x264-builds.patch](patches/lib_build-deps_v2026.910.121303_loongarch64/0003-Recognize-LoongArch64-for-CBS-and-x264-builds.patch)


```diff
diff --git a/CMakeLists.txt b/CMakeLists.txt
index df45e82..bd12d14 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -24,6 +24,7 @@ option(BUILD_FFMPEG_NV_CODEC_HEADERS_PATCHES "Apply FFmpeg NV Codec Headers patc
 option(BUILD_FFMPEG_SVT_AV1 "Build FFmpeg SVT-AV1" ON)
 option(BUILD_FFMPEG_SVT_AV1_PATCHES "Apply FFmpeg SVT-AV1 patches" ON)
 option(BUILD_FFMPEG_LIBVA "Build FFmpeg with libva support" ON)
+option(BUILD_FFMPEG_LIBVA_SYSTEM "Use system libva instead of building libva" OFF)
 option(BUILD_FFMPEG_LIBVA_PATCHES "Apply FFmpeg libva patches" ON)
 option(BUILD_FFMPEG_X264 "Build FFmpeg x264" ON)
 option(BUILD_FFMPEG_X264_PATCHES "Apply FFmpeg x264 patches" ON)
diff --git a/cmake/ffmpeg/ffmpeg.cmake b/cmake/ffmpeg/ffmpeg.cmake
index 0b1871f..c3087b5 100644
--- a/cmake/ffmpeg/ffmpeg.cmake
+++ b/cmake/ffmpeg/ffmpeg.cmake
@@ -1,3 +1,6 @@
+set(FFMPEG_USER_CONFIGURE "${FFMPEG_EXTRA_CONFIGURE}")
+set(FFMPEG_EXTRA_CONFIGURE)
+
 if(BUILD_FFMPEG_ALL_PATCHES OR BUILD_FFMPEG_CBS_PATCHES)
     file(GLOB FFMPEG_CBS_PATCH_FILES ${CMAKE_CURRENT_SOURCE_DIR}/patches/FFmpeg/FFmpeg/cbs/*.patch)

@@ -12,6 +15,8 @@ elseif (${arch} STREQUAL "ppc64le")
     set(CBS_ARCH_PATH ppc)
 elseif (${arch} STREQUAL "amd64" OR ${arch} STREQUAL "x86_64")
     set(CBS_ARCH_PATH x86)
+elseif (${arch} STREQUAL "loongarch64" OR ${arch} STREQUAL "loong64")
+    set(CBS_ARCH_PATH loongarch)
 elseif (${arch} STREQUAL "mips")
     set(CBS_ARCH_PATH mips)
 else()
@@ -145,6 +150,8 @@ if(CMAKE_CROSSCOMPILING)
     endif()
 endif()

+list(APPEND FFMPEG_EXTRA_CONFIGURE ${FFMPEG_USER_CONFIGURE})
+
 # convert list to string
 # configure command will only take the first argument if not converted to string
 string(REPLACE ";" " " FFMPEG_EXTRA_CONFIGURE "${FFMPEG_EXTRA_CONFIGURE}")
diff --git a/cmake/ffmpeg/libva.cmake b/cmake/ffmpeg/libva.cmake
index 0476b52..5a29ad9 100644
--- a/cmake/ffmpeg/libva.cmake
+++ b/cmake/ffmpeg/libva.cmake
@@ -1,3 +1,10 @@
+if(BUILD_FFMPEG_LIBVA_SYSTEM)
+    find_package(PkgConfig REQUIRED)
+    pkg_check_modules(LIBVA REQUIRED libva libva-drm libva-x11)
+    add_custom_target(libva)
+    return()
+endif()
+
 CPMGetPackage(libva)

 set(LIBVA_GENERATED_SRC_PATH ${libva_SOURCE_DIR})
diff --git a/cmake/ffmpeg/x264.cmake b/cmake/ffmpeg/x264.cmake
index 4b3b29e..b4dbbea 100644
--- a/cmake/ffmpeg/x264.cmake
+++ b/cmake/ffmpeg/x264.cmake
@@ -14,6 +14,8 @@ elseif (${arch} STREQUAL "ppc64le")
     set(X264_ARCH powerpc64le)
 elseif (${arch} STREQUAL "amd64" OR ${arch} STREQUAL "x86_64")
     set(X264_ARCH x86_64)
+elseif (${arch} STREQUAL "loongarch64" OR ${arch} STREQUAL "loong64")
+    set(X264_ARCH loongarch64)
 elseif (${arch} STREQUAL "mips")
     set(X264_ARCH mips)  # TODO: unknown if this is the correct value
 else()
```
