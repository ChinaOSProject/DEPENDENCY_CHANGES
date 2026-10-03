# Sunshine v2026.914.233613：LoongArch ABI v0 / Debian 10 适配报告

完成日期：2026-10-03，Asia/Shanghai。来源仓库：[LizardByte/Sunshine](https://github.com/LizardByte/Sunshine)。本次构建官方稳定版本 [v2026.914.233613](https://github.com/LizardByte/Sunshine/releases/tag/v2026.914.233613)，不追踪移动中的上游分支。

## 交付与验证结果

正式包：[`sunshine/loongarch64/sunshine_v2026.914.233613_loongarch64.deb`](https://github.com/ChinaOSProject/binaries/blob/main/sunshine/loongarch64/sunshine_v2026.914.233613_loongarch64.deb)，实际文件大小 **20,347,668 bytes**。

SHA256：`54b81e74452f74a74a11bf74edd2d90808f274bcbf3e67c1a9c7eacea4db8973`。

本机 dpkg 使用的架构名称为 **loongarch64**；ELF 为 LoongArch ABI v0、flags `0x3`，解释器 `/lib64/ld.so.1`。没有用现代 ABI v1 的 `0x43` 文件冒充旧 ABI。包内全部19个原生 ELF，包括应用、11个私有库、7个插件，均完成 ABI 检查。最高 GLIBC 要求为2.28，私有 Qt 最高 GLIBCXX_3.4.21 / CXXABI_1.3.9，目标已有 GCC8 运行库满足要求。

- 完整上游 gtest：601项、89个 suite，594通过、7按平台/设备条件跳过、0失败、0错误，耗时13.917秒。新增3项字符串与5项 GBM 回归测试均通过。
- 同一 `.deb` 在独立 buster 最小容器中实际安装，非 root 启动；Web 未认证返回401、认证后配置与首页返回200，版本为2026.914.233613，私有 Qt/XCB 插件实际加载。
- 同一 `.deb` 通过真实 Moonlight 协议：PIN配对、客户端证书认证、Desktop启动、加密RTSP、H264视频和Opus音频接收/解码。8秒测试解码242帧视频、551个音频包，客户端错误0；视频包含59个不同帧，音频包含实际440Hz测试音而非仅静音包。
- 目标机实际 loader 解析121个原生库，全部 ABI v0；19个交付 ELF 在目标库环境下 `ldd -r` 均无缺库、未解析符号或 GCC13 构建运行库误入。包声明的55项 Depends 均被目标现有包名/版本满足，未安装任何宿主构建依赖。

## 来源、分支与最小源码修改

上游稳定 commit：`63d35f702ee9e362e43263742981836ec0710384`。官方默认分支实际名为master，本次以固定稳定标签为基线。

传播顺序：上游稳定 commit -> `debian10` -> `loong64`。最终 `debian10` 和 `loong64` 均为 `e0627e50193a296365eb86de96498bbc63c144fb`；本体没有额外架构专属源码补丁，也没有架构分支反向合入通用分支。

| 单点提交 | 必要问题与最终行为 |
| --- | --- |
| `d845e5b1b5ec13689cc7bb169a14005e771476d7` | GCC13有 `std::format`，但没有本处使用的 C++23 `std::ranges::to`。新增小型 `util::join_strings` 替换KMS日志中该用法；保留顺序、分隔符、空元素和嵌入NUL语义，不改采集逻辑。 |
| `e0627e50193a296365eb86de96498bbc63c144fb` | 旧GBM缺少 `gbm_bo_get_fd_for_plane` 和 `gbm_bo_create_with_modifiers2`。用运行时 `dlsym` 检测；现代接口可用时保留原路径/usage flags及失败语义，旧接口只导出明确的单平面缓冲区，拒绝无法正确导出的多平面缓冲区。 |

最终差异只含6个必要源码/测试文件，187行新增、6行删除，其中大部分为回归测试。没有依赖源码混入本体提交，也没有 Dockerfile、脚本、产物、报告、临时源或本地绝对路径进入源码commit。`git status` 干净，`git diff --check` 通过。

以下补丁可以在上述稳定 commit 上按顺序 `git am`；先建立 debian10，再让 loong64 继承该分支：

- [0001：替换KMS日志的 ranges::to](patches/sunshine_v2026.914.233613_loongarch64/0001-fix-linux-format-KMS-display-names-without-ranges-to.patch)
- [0002：旧GBM接口检测与后备](patches/sunshine_v2026.914.233613_loongarch64/0002-fix-linux-support-legacy-GBM-buffer-interfaces.patch)

ChinaOSProject/Sunshine 源码fork本次不存在，因此使用报告仓库中的独立提交补丁归档；没有向LizardByte创建issue/PR或推送分支。

## 隔离构建与工具链

所有应用、依赖和Web资源编译、打包都在项目专用 rootless Podman 容器内完成。没有宿主 apt/pip/npm 或全局工具链安装，也没有替换宿主Qt/DRM/GBM。

| 项目 | 实际环境 |
| --- | --- |
| 基座 | `cr.loongnix.cn/library/debian:buster-slim`，GLIBC2.28、LoongArch ABI v0 |
| 项目容器 | `sunshine-build-20261003` |
| 最终编译镜像 | `localhost/sunshine-build:2026.914.233613-buster-loongarch64` |
| 最终镜像ID | `c791729ce0f8701f1baf673c87dc83785511e44426319edfe5e8777acc0fe706` |
| 依赖容器 | `sunshine-deps-20261003`，另有独立 build-deps 镜像和报告 |
| 干净包验证容器 | `sunshine-package-check-20261003`，直接从buster-slim生成，无系统Qt/ICU67、GCC13/Node/LATX挂载 |
| C/C++ | 只读复用 GCC13.4.0；使用该工具链原有 as-loong64 wrapper，旧binutils实际生成 `0x3` 对象 |
| 构建工具 | CMake3.27.9、Ninja、Python3.10.16；无源码修改的Meson1.5.2只安装在容器内 |
| 前端 | Node24.21.0、npm11.19.1，原版package-lock，Vite8.3.0 / Rolldown1.2.8 |

Podman使用VFS存储；本机旧内核不支持所需overlay userxattr，采用容器安装后commit生成专用镜像。容器指定 `--security-opt seccomp=unconfined`，适应该Podman/旧LoongArch环境；没有修改宿主安全配置。项目Agent位于独立Herdr Sunshine space。

GCC13最初调用旧as时不识别 `-mabi=lp64d`。复用已有 `as-loong64`，仅将该参数翻译为旧as支持的 `-mabi=lp64`，并在容器中放到GCC原有查找路径；不是事后改写ELF标记。链接显式指定旧loader，并静态链接应用的C++/GCC运行库。编译器自身的 LD_LIBRARY_PATH 仅服务构建工具；运行依赖扫描和包启动验证均清除 LD_LIBRARY_PATH/LD_PRELOAD。

源码与临时材料分别位于 `/home/laevatein/ChinaOSProject/sunshine`、`/tmp/sunshine-build-20261003`，容器内为 `/src`、`/work`。临时脚本、测试客户端、环境与日志均在源码仓库外。

## 依赖处理

| 依赖 | 实际版本与处理 |
| --- | --- |
| build-deps | 官方v2026.910.121303，适配commit `0e1eff16622bf8e70f6a637136a60e38208feb19`。需要源码修改时由主Agent单独clone并委派依赖Agent，保持本体与依赖提交分离。 |
| FFmpeg | RELEASE9.0.1，锁定 `bf1b838f2ab88b4f8fd83443325c782ea0e0f7fa`，静态avcodec/avutil/swscale/CBS。 |
| x264 | 0.165.3222 b35605a，8/10-bit静态库；测试日志实际选择LSX/LASX。 |
| x265 | 4.1 / API215，portable C++、8-bit静态库、关闭assembly。 |
| libva | 官方2.7.0头文件匹配目标已有VA API1.7，避免容器较新2.10头引入宿主缺失API；不交付新版私有libva。 |
| Boost | 1.89.0，原版源码静态构建；Locale使用glibc内置iconv，无额外动态Boost或新版libiconv。 |
| libdrm | 官方2.4.124，原版源码私有编译；SHA256 `ac36293f61ca4aafaf4b16a2a7afff312aa4f5c37c9fbd797de9e3c0863ca379`，ELF `0x3`、GLIBC最高2.27。旧2.4.97/2.4.101缺少新FB2/HDR/connector接口，私有库补齐编译接口，不升级宿主库。 |
| Qt/ICU | Loongnix Qt5.15.2、ICU67.1私有封装；库/插件均已逐项检查旧ABI和GLIBCXX要求。 |
| 其他 | 原版JSON3.11.3、锁定Gtest；私有libdouble-conversion.so.1补齐目标另一SONAME不能提供的接口。 |

FFmpeg、CBS、x264、x265的源码处理、默认行为保护、491个归档ELF成员的检查、实际H264/HEVC编解码和目标libatomic验证，详见[独立build-deps报告](lib_build-deps_v2026.910.121303_loongarch64.md)。仅依赖项目没有人为打包应用 `.deb`。

关键锁定子模块：

| 子模块 | commit |
| --- | --- |
| moonlight-common-c | `62e066388f1a1b133e0bee947b9a374311a3354b` |
| tray | `c329d9fd0d39dfb47f0f2fb5467e1db2e8a1d623` |
| libvirtualhid | `53e1a949fc0784af716b782ddfa6c647cafd1f05` |
| libdisplaydevice | `6e9722f89103320c948dc1199066c9e17a69e88a` |
| lizardbyte-common | `f9d91e1d29b7473f58e43acde4579da4e56c4abe` |

## Web资源与编译配置

LoongArch原生Rolldown绑定不可用；官方WASM后备在本机V8上缺少Wasm SIMD。为保留原版锁定前端，不改package.json/lock：仅在容器内使用已有LATX x86_64模拟器和官方同版本x64 Node编译架构无关Web资源。Node x64归档SHA256为 `fd8e59d5a511510f6a298afb548f18c7d2b1be404d8b4a27d94fbe49f56cb2d6`；Boost归档SHA256为 `85a33fa22621b4f314f8e85e1a5e2a9363d22e4f4992925d4bb3bc631b5a0c7a`。

首次LATX默认选项在输出阶段崩溃；明确设置 `LATX_AVX_CPUID=0 LATX_CLOSE_PARALLEL=1 LATX_OPTIMIZE=0 RAYON_NUM_THREADS=1 UV_THREADPOOL_SIZE=1` 后原版构建通过。2297模块、9个HTML入口、81个Web资源完成，入口本地依赖无缺失；正式CMake Web构建也成功。交付包没有Node、LATX、x86 ELF或node_modules。

正式CMake使用Release、Ninja、BUILD_TESTS=ON、BOOST_USE_STATIC=ON，安装前缀 `/usr`、资源 `share/sunshine`。通过上游 BRANCH/BUILD_VERSION/COMMIT/TAG 机制写入实际版本及完整commit，避免默认0.0.0。发布者和问题归档链接设为ChinaOSProject。

关键配置：

```text
CMAKE_EXE_LINKER_FLAGS=-static-libstdc++ -static-libgcc -Wl,--disable-new-dtags -Wl,--dynamic-linker=/lib64/ld.so.1
CMAKE_INSTALL_RPATH=/usr/lib/sunshine
FFMPEG_PREPARED_BINARIES=/work/ffmpeg-prefix
LIBVA_INCLUDE_DIR=/work/ffmpeg-prefix/include
LIBDRM_LIBRARIES=/opt/sunshine-runtime/lib/libdrm.so
SUNSHINE_ENABLE_CUDA=OFF
SUNSHINE_ENABLE_VULKAN=OFF
SUNSHINE_ENABLE_KWIN=OFF
SUNSHINE_ENABLE_PORTAL=OFF
```

保留X11、DRM/KMS、VAAPI、Wayland、Qt托盘及H264/HEVC软件编码。关闭项基于旧系统实际缺少的编译/运行能力；未为绕过错误删除必要软件H264功能。完整外部复现配置为 `/tmp/sunshine-build-20261003/configure-sunshine.sh`、`env.sh`，然后 `cmake --build /work/cmake-build-loongarch64 --parallel 3`，构建目录符合上游要求的 `cmake-build-` 前缀。

## 正式包布局与运行环境

- `/usr/bin/sunshine` 是链接，真实原生程序为 `/usr/lib/sunshine/sunshine`。
- 私有Qt、ICU、DRM、double-conversion位于 `/usr/lib/sunshine`；Qt插件位于其 `qt/plugins`。`qt.conf` 仅位于私有程序目录，没有安装 `/usr/bin/qt.conf` 影响其他Qt应用。
- 应用使用绝对DT_RPATH `/usr/lib/sunshine`，没有 `/work`、`/home`、`/opt` 等构建路径进入交付ELF的RPATH。不依赖LD_LIBRARY_PATH加载私有库，也没有弱化上游file-capability环境清理。
- postinst解析真实可执行文件并设置 `cap_sys_admin,cap_sys_nice+p`；在独立容器中已实际成功。安装不会启动服务器或触发真实设备。
- 包含Web、默认应用、着色器、图标/桌面文件、上游udev规则、uhid module-load配置及可选systemd用户服务；不会自动启用用户服务。
- 许可证和Debian运行说明随包安装到 `/usr/share/doc/sunshine`。依赖由完整 `dpkg-shlibdeps` 扫描计算，私有库不要求宿主安装Qt5.15/ICU67或新版DRM。

用户安装包后，以桌面用户运行 `sunshine`，打开 `https://localhost:47990` 创建Web账号并配对Moonlight。可选使用 `systemctl --user enable --now sunshine.service`；真实设备访问取决于桌面会话权限和内核UHID支持。

## 测试证据与适用边界

上游gtest在UID1000、独立Xvfb:97及D-Bus会话运行。7项跳过为Windows专用UTF1项、缺少真实tray manager的2项托盘生命周期/视觉测试、环境限制的2项绝对鼠标输入测试、无实际硬件的NVENC/VAAPI编码器2项。没有把跳过项记为通过。证据：`upstream-tests.log`、`gtest-results/upstream.xml`。

最小包验证容器没有构建工具挂载或系统Qt/ICU67：认证API401/200、首页200、UID1000、私有Qt/XCB映射及清洁环境均通过。证据：`package-check/results/smoke-result.json`、`web-config.json`、`native-maps.log`、`cli-version.log`。

实际串流验证使用锁定官方moonlight-common-c编译的临时客户端，运行包安装后的程序，独立Xvfb:96、Pulse null sink和loopback端口49189；不接触宿主桌面/扬声器。LAN/WAN加密均强制为2，Moonlight请求全部加密；日志中encryptionSupported/Requested/Enabled均为7，session为 `rtspenc://`。客户端完整配对并验证服务端签名，使用客户端证书完成HTTPS访问、启动Desktop与取消会话。

H264实际接收242帧、1个IDR，解码后242帧/59个不同帧；Opus551包、264480采样，PCM1,057,920bytes、RMS5661.47，错误0。测试音为私有设备中的440Hz音，视频为持续改变颜色的虚拟桌面。第一次fixture禁用本地回放使Sunshine切换音频sink而仅收到静音；将fixture localAudioPlayMode改为1后完整通过，产品源码与包未变，失败日志保留。所有本次测试进程均已精确停止。

证据：`primary-validation/results/final-protocol-result.json`、`stream-20261003T145817/protocol.log`及对应Sunshine日志、H264/PCM/framemd5。包hash与测试对象一致。这是640x360x30、8秒的功能验证，不代表1080p/4K、长时间或WAN性能。

目标AMD OLAND / amdgpu、Mesa20.0.8 / VA API1.7的只读能力查询成功，但当前驱动未公布H264/HEVC编码能力；未执行硬件编码，应用可使用软件路径。本次没有验证真实物理显示的KMS/Wayland采集、实际键鼠/手柄输入、真实托盘交互或物理Moonlight终端。

FFmpeg自身LSX/LASX intrinsics因GCC13不支持相关选项自动关闭，x264汇编路径已实际使用LSX/LASX；x265为8-bit portable C++。未交付SVT软件AV1、CUDA/NVENC、Vulkan、KWin或Portal路径，不声明HDR/10-bit HEVC等未验证能力。

所有本地证据位于 `/tmp/sunshine-build-20261003` 与独立依赖工作目录。正式报告和源码补丁在源码仓库外统一归档；最终`.deb`通过二进制仓库Git LFS交付，未上传测试私钥或临时构建材料。
