# Proton 11.0-2e：原生 LoongArch64 移植记录

更新日期：2026-10-29（Asia/Shanghai）。状态：已完成本体原生构建规则、FFmpeg、lsteamclient 两侧模块编译链接，并以真实安装的现代 Wine core 通过 Proton Session 执行普通原生 Winelib 程序。尚未完成完整原生 Proton、Windows PE 程序或 Steam 游戏运行，未生成 Proton `.deb`。

用户要求坚持原生龙芯，优先完成 Firefox；随后授权在现有配置与可复用实现不足时修改上游源码。Firefox 已另行完成交付。本记录区分已运行的原生组件和未解决的运行时接口。

## 固定源码与目标

- 本体上游：<https://github.com/ValveSoftware/Proton>，`proton-11.0-2e`，`bc896217ffcd67f892f952b010f898fe7bd0f0a1`。
- 固定 Wine：<https://github.com/ValveSoftware/wine>，`e60892cc2398b0a83c332a2078c7b85e47bdb42b`；独立工作树 `/home/laevatein/ChinaOSProject/wine-proton`，未以旧 Wine 8.21 替换该基线。
- 本体工作树：`/home/laevatein/ChinaOSProject/proton`。固定 FFmpeg 子模块已浅初始化，其余依赖按实际需要获取；未下载官方 x86 Steam Runtime SDK 充当原生 SDK。
- 本机为 Loongson-3A5000、麒麟 V10 SP1，glibc 2.28，旧 ABI；原生 ELF machine 258、flags `0x3`，动态解释器 `/lib64/ld.so.1`。
- 所有编译在任务专属 rootless Podman 中进行；工具链以只读挂载或容器内安装提供，未向宿主全局安装依赖、修改宿主库或服务。

## 本体源码改动

本地分支遵循 `upstream/main → debian10 → loong64`。通用 Wine source override 和 Steam 封装的特性检测改动位于 `debian10`；架构分支从其继承。

| 提交 | 修改及目的 |
| --- | --- |
| `870ca24cceecb352c4ca7d3f444fb44995cd3c0d` | `--wine-source` / `WINE_SOURCE` 支持独立 Wine 端口，并把真实外置目录挂入构建容器；默认仍使用原子模块。 |
| `6e6d960ac4a51448b88163b3084b0d55d7e2fefc` | LoongArch 原生 Unix triple/libdir、CMake/Meson/Cargo 和组件构建规则、默认 prefix 目录。要求显式提供原生 SDK，没有虚构 LoongArch PE compiler triple。 |
| `acba35223b1a28749af57665a38ec7100f05f86d` | 启动器选择 `bin-loongarch64/wine` 和 wineserver，缺失时明确失败；区分原生 Wine builtin 与 x86 Windows 客体 DLL 路径。 |
| `33c3622c2beaed19c98d9f81a21a10451f328dae` | SDK 检查同时核对 uname 和 `/bin/sh` 的真实 ELF64、小端、EM_LOONGARCH=258，防止仅凭宿主架构误认 x86 容器。 |
| `8f6efbf` | lsteamclient 的生成器和已有生成代码统一以 Wine `_WIN64` 选择 64 位结构、RTTI 和 WoW64 封装；`sysv_abi` 属性限于 x86，其他 Unix 架构使用真实本地 ABI。实际编译错误确认后，机械替换 6987 处位宽条件，未改接口内容或伪装 CPU 宏。 |
| `47456a2` | 允许 `STEAM_COMPAT_NATIVE_CLIENT_PATH` 指定兼容的原生 Steam 库完整路径；保留既有三类 SDK 默认路径，无默认且未指定时明确返回 `STATUS_NOT_SUPPORTED`，不虚构龙芯 SDK。 |
| `4619305` | 将上述两项通用改动从 `debian10` 合并至 `loong64`。 |

未提交临时探针、Dockerfile、对象文件、构建缓存或本机绝对路径。尚未创建远端 fork 或推送这些源码改动；最终需要维护的本体仓库为 `ValveSoftware/Proton`。

## 实际构建与运行验证

专属环境 `proton-native-20261028`，工作目录 `/tmp/proton-native-port-20261029`。目录名中的日期是任务标识。

- 使用真实原生 SDK 和独立固定 Wine source 执行 configure，退出 0；`make test-container` 实际验证外置源码挂载。缺少 SDK 或未知架构时退出 1，不生成可误用的 Makefile。
- 完整 Makefile 的 x86_64、ARM64、LoongArch 映射检查通过；LoongArch 的 Unix 目标为原生，Windows 客体目标仍为 i386/x86_64，未启用面向 ARM64 的 FEX。
- 通过实际 CMake、Meson、Cargo 规则分别编译并安装原生共享库，再实际加载调用函数，均返回 261。三件均为 machine 258 / flags `0x3`，Rust 探针 GLIBC 需求最高 2.27。
- 启动器的三架构选择、ARM opt-out、缺失原生运行时拒绝及 prefix 链接检查通过；以该环境实际执行旧 Wine 8.21 原型的 native Wine/wineserver `--version` 成功。这只证明核心路径接线，没有把旧原型当成固定 Wine 的完成版本。
- 随后使用现代 Wine 的实际 core 安装树验证正式布局：公共 wrapper 在 `bin/wine`，实际 ELF loader/preloader、Unix ntdll 和 ELF builtins 在 `lib/wine/loongarch64-unix`，Wine placeholder 在 `loongarch64-windows`。按本体 post-build 规则将公共入口移入 `files/bin-loongarch64`，其余保留实际安装布局；真实本机 `platform.machine()`、无架构 mock，使用本体 `Session.init_wine/run_proc` 运行现代普通 `main(42,37)` 返回 0。清除外部 LD_LIBRARY_PATH/LD_PRELOAD/WINEBUILDDIR/WINELOADERNOEXEC/WINEDLLPATH，不设置 bootstrap 环境。
- 使用本体 `default_pfx.setup_dll_symlinks` 对实际 Wine 工具生成的 ntdll/kernel32/kernelbase/native-main metadata 建立原生相对链接，再以 `C:\windows\system32\native-main.exe` 运行同一程序返回 0。这补齐了此前仅用 fixture 证明的真实安装目录和 prefix 链接验证。该测试只调用 Session 的环境/进程接口，未运行完整 Proton 前缀创建和 Steam setup；core 子集缺失 wineboot 的 `c0000135` 和高地址范围预留警告仍保留在日志中。
- 官方固定 FFmpeg `a77521cd5d27e955b16e8097eecefc779ffdcb6d`（4.3.3）通过真实 `ffmpeg-loongarch64-build` 配置、编译、安装；没有修改 FFmpeg 源码。七个共享库均为 machine 258、flags `0x3`，GLIBC 需求不超过 2.28。
- 使用上述 FFmpeg 库与实际启用的 matroska demuxer，通过 custom AVIO 读取本体 `media/blank.mkv`，结果 `FFmpeg=4.3.3 streams=2 packets=305 eof=1`，退出 0、无 stderr。沿用本体原有的精简 FFmpeg 功能配置，没有以不支持的 file protocol 冒充运行验证。
- 使用固定现代 Wine 的真实公共头文件，lsteamclient 的全部 219 个 Unix C++ 源文件原生编译退出 0；与其实际 `ntdll.so` 通过 `-Wl,-z,defs` 链接成完整 `lsteamclient.so`。共享库为 EM_LOONGARCH=258、flags `0x3`，GLIBC 需求最高 2.27；GLIBCXX 需要 3.4.29，不能直接假定宿主旧 C++ 库满足。
- 用真实动态加载器 `RTLD_NOW` 加载该库并调用初始化函数，未配置、空路径、4096 字符超长路径、缺失文件和实际麒麟 Wine8 包的 x86_64 ELF 库五类情况全部按预期失败。测试没有提供模拟 Steam SDK，也没有以失败路径测试冒充 Steam 游戏成功。
- 对代表性完整 Windows Apps 封装使用 Clang 的真实 i686/x86_64/AArch64 Windows target 预处理，修改前后输出逐字节一致。CRT 先包含顺序的原生 Winelib 编译另发现 Wine `corecrt.h` 位宽错误，现代 Wine 已独立修复为 `6175b31`；更新真实头文件后全部 44 个 Windows 侧 C 源文件也原生编译退出 0，并把指针截断警告作为错误。全部 263 个对象和 Unix 共享库均核对为 machine 258 / flags `0x3`。
- 使用现代 Wine 实际编译的 winegcc/winebuild，以及正常构建目标产生的 user32/ws2_32/winecrt0/ucrtbase/kernel32/ntdll import/static 库，按 MSVCRT 模式完整链接 `lsteamclient.dll.so`，`-z defs` 通过。该 ELF builtin 同为 machine 258、flags `0x3`、64 KiB LOAD 对齐、GLIBC 最高 2.27；独立 `RTLD_NOW` 映射成功。另用 Wine 正常工具生成 machine `0x6264` 的 PE placeholder，此文件只有 Wine 加载元数据，不是新的 LoongArch PE 编译器或可执行客体代码。尚未验证这个 DLL 的完整 Windows 导入解析、DllMain 和 Steam 接口运行。
- 已组装实际开发组件树 `/tmp/proton-native-port-20261029/native-components-stage-root`，包含现代 core、两侧 lsteamclient、7 个 FFmpeg 库及私有 GNU13 `libstdc++.so.6`（其自身最高 GLIBC 2.27）。只用该树的库路径、系统 Python 和 `/usr/bin:/bin`，两侧 lsteamclient 的 ELF 映射、7 个媒体库及实际版本函数调用均通过；`/proc/self/maps` 确认私有 C++ 库确实从组件树加载，未加载 `/opt/gcc13`、Clang 或 LLVM 的 SDK 库。此为开发组件树，不是完整 Proton 分发；Windows 导入/DllMain/Steam/guest 验证范围没有因此扩大。证据 `logs/native-components-sdk-free-root.{log,json,exit}`。

关键证据：`logs/native-rule-abi.json`、`launcher-validation.json`、`launcher-validation.log`、`configure-wine-source.log`、`configure-sdk-elf.log`、`test-container.log`、`mapping-*.log`、`ffmpeg-native-build.log`、`ffmpeg-native-validation.json`，以及 `lsteamclient-unix-build-root.json`、`lsteamclient-unix-link-root.exit`、`lsteamclient-unix-load-root.json`、`lsteamclient-existing-arches-root.json`、`lsteamclient-winelib-link-root-02.exit`、`lsteamclient-winelib-abi-root.json`、`modern-core-session-root.json`。初次本体交接为 `/tmp/proton-native-port-20261029/PROTON_IMPLEMENTATION_HANDOFF.md`；该交接之后的 Steam 封装修复和实际 core Session 测试以本报告及新提交为准。

## 仍需完成的运行时条件

1. 固定 Wine 上游缺失的原生 LoongArch ntdll/信号/线程/异常后端已在独立端口中实现并实跑验证；实际 core 安装树和 launcher/prefix 链接也已接通。现代 Wine 仍只安装已构建的 core 子集，自动 wineboot 报 `c0000135`；继续标准闭包编译又在现代 UCRT/MSVCRT 暴露四个缺失的异常处理架构接口，真实链接失败。详见 `lib_wine_v11.0_loongarch64.md`。完整正常前缀及图形回调尚未完成；旧 Wine8 原型的正常前缀成功不代替现代基线验证。
2. `lsteamclient` Unix 模块和 Windows 侧 native ELF builtin 均已编译、链接并映射；Windows 导入、DllMain、与实际 Wine 安装树的接通仍需验证。尚未取得或证明可加载兼容的原生 Steam SDK；显式库路径不是跨 ISA ABI 桥。
3. x86 Windows 游戏仍需与原生 Wine 对接的 CPU 执行后端、跨架构调用及上下文桥。目前构建规则没有提供这些实现，原生 Box64 动态翻译能力也不等于其 Windows 接口已支持龙芯。
4. Steam 原生库或跨 ABI/进程桥仍缺失；LoongArch ELF 不能直接 dlopen x86 `steamclient.so`。当前 SDK 也尚未含全套 Windows 客体 PE 交叉工具链，未完成全量默认构建。

初始上游障碍已实际复现：未经修改的固定版本在 `TARGET_ARCH=loongarch64` 或 `loong64` 下均于架构检查退出 2，日志 `/tmp/proton-native-check-20261028/`。这一历史失败已由上述本体规则改动推进，不能继续把“未知 TARGET_ARCH”当作当前代码的最终状态。

## 本地组件与外接缓存

用户授权的外接盘仅使用新增目录 `/media/laevatein/VCLIB/ChinaOSProject-build-cache-20261029-000151`，没有格式化、分区、重挂载或修改原有盘内文件。Linux 活动构建目录继续位于 ext4；NTFS 上以 tar 保存 Linux 文件属性，实际写入/rename/fsync 和完整 SHA256 已验证。

除已验证的 Firefox 对象/容器归档外，新增原生组件编译缓存 `archives/proton-11.0-2e-native-components-20261029-root.tar`（202004480 字节，SHA256 `2b1f7faec188a075e454b305cf4446b93b8e0b0569dc69a0d8a9c91c3bfb180f`）和开发组件树 `archives/proton-11.0-2e-loongarch64-development-components-20261029.tar`（58869760 字节，SHA256 `817510b5b5d265c413df523f375935afcf2ca02d606623d2a3918a5510b7671d`）。两者都以 `tar --compare --acls --xattrs` 对照 ext4 原件通过，原件保留；均不是完整 Proton 发布包。

`metadata/source-patches-20261029-native` 保存实际 Firefox/Proton/现代 Wine/旧 Wine 原型的补丁系列、固定基线和逐文件 SHA256。Proton 6 个、现代 Wine 16 个、旧 Wine 14 个补丁已在隔离 checkout 上从固定上游逐项重放，Git tree 与对应适配分支完全一致。未验证的现代正常前缀实验及真实编译器失败另存 `metadata/modern-prefix-blocker-20261029`，不混入已验证系列。

## 麒麟 Wine 8 软件包复用核查

2026-10-28 按用户提供的系统仓库线索检查。候选为 `wine8=8.0.58`，实际包名 `wine8_8.0.58_all.deb`，53,296,790 字节，`Architecture: all`、`Multi-Arch: foreign`。通过当前系统 APT 下载，SHA256 `963924cc62dbbcf2954550cce7873143cb889f366434fa5502382e445242c9d5` 与仓库元数据一致。APT 将该 all 包列在 loongarch64 Packages 索引下，不能据此认定其内部机器码是 LoongArch。

解包后逐一检查普通文件的 ELF/PE 头，82 个 ELF 中 44 个为 EM_X86_64、38 个为 EM_386，没有 EM_LOONGARCH；1464 个 PE 为 702 个 AMD64 和 762 个 i386。仅有的压缩文件是文档 changelog，没有另藏原生引擎归档。

| 文件（相对 `/opt/kylin-wine/wine8`） | 实际架构 | PT_INTERP |
| --- | --- | --- |
| `bin/wine` | i386 ELF32 | `/run/wine/lib/ld-linux.so.2` |
| `bin/wine64` | x86_64 ELF64 | `/run/wine/lib64/ld-linux-x86-64.so.2` |
| `bin/wineserver` | x86_64 ELF64 | `/run/wine/lib64/ld-linux-x86-64.so.2` |
| `lib/wine/x86_64-unix/ntdll.so` | x86_64 ELF64 | 共享库无解释器 |

`kylin-wine-internal` 设置包内 PATH/WINEDLLPATH、加载 `x86_64.env` 后 `exec wine`；该环境脚本明确使用 `/run/wine` 的 i386/x86_64 库目录。没有执行安装脚本或启动任何 Wine 前缀。

宿主已安装 `wine-runtime=22.04-22k1`（all），提供只读挂载在 `/run/wine` 的 Ubuntu 22.04 x86 运行库镜像。对可读普通文件的扫描检出 1580 个 x86_64 ELF、940 个 i386 ELF、0 个 LoongArch ELF；权限不可读的 shadow/private/cache 等路径在证据中单独记录。镜像内 libc 为 x86 两架构的 2.35，不是宿主原生 glibc 2.28 的可替换库；镜像中没有安装 Wine/DXVK/vkd3d 包。

宿主实际具有 `lat=1.6.4-2`（loongarch64）；`latx-i386.real` 与 `latx-x86_64.real` 是 LoongArch ELF64、flags `0x3`、解释器 `/lib64/ld.so.1`、最高 GLIBC 需求 2.28。当前 i386/x86_64 binfmt_misc 规则指向对应 LATX wrapper，wrapper 再配置 `/run/wine` 的 x86 库路径。这解释了 x86 Wine 可以借助本机转译设施启动的设计，但本次未执行转译器或验证游戏兼容性。

当前本地 APT 索引中，Wine 8 要求 `inst-translation (>= 3.0.4)`，该依赖候选仅为 `3.0.0`，`apt-get --simulate install wine8=8.0.58` 实际退出 100，报告未满足依赖。此为当前索引的单独安装障碍，不作为所有远端仓库版本的结论；未更改宿主源或安装软件。

复用结论：该包的 Wine 主程序和 Unix 库均为 x86，不能直接作为原生 LoongArch Wine/Proton 后端。字体、配置和启动脚本可作架构无关资源或打包参考；已有 LATX 与 runtime 是单独外架构执行路径的材料，不补齐原生 Wine 的信号/异常/调用约定实现。包内没有可直接取用的 LoongArch 后端源码。后续若找到真正的原生社区实现，应另行核对具体源码和功能，不以本次二进制包检查否定所有社区移植。

原始包、`wine8-payload-inventory.json`、`apt-evidence.json`、`installed-runtime-summary.json`、`installed-runtime-evidence.txt`、`installed-runtime-elf-scan.json` 位于 `/tmp/wine8-reuse-check-20261028/`。此次仅做下载、静态解包、只读系统检查和模拟安装。

## 原生 Wine 源码路线与仓库边界

除发行包外，继续核对了公开的 [loongarch-moe/la64-wine](https://github.com/loongarch-moe/la64-wine/tree/9f5367cc56defca16452a63ad253c7c49f848871)，固定 `9f5367cc56defca16452a63ad253c7c49f848871`，实际为 Wine 8.21，提交说明 `pre support`，仓库已归档。该分支包含真实 LoongArch CONTEXT、Unix signal、winebuild/winegcc 等架构代码，可作移植素材；但不能直接把它作为已完成的运行时。

[其 NTDLL 用户态架构文件](https://github.com/loongarch-moe/la64-wine/blob/9f5367cc56defca16452a63ad253c7c49f848871/dlls/ntdll/signal_loongarch64.c) 的 `LdrInitializeThunk`、`RtlUserThreadStart`、`KiUser*Dispatcher`、上下文捕获和异常函数仍有 `break 1;ret` 占位，unwind 未实现；Unix 文件的 WoW64/APC/回调也有 `exit(1)`。补齐构建架构名称无法替代这些行为。原生代码正在独立 Wine 项目与专用 Podman 内验证，局部正确修复和实验结果另行归档，不生成冒充完整 Proton 的包。

较新的 Wine 9.9 原生实现仅找到 [Wine-devel 公开线程](https://list.winehq.org/hyperkitty/list/wine-devel@list.winehq.org/thread/WEX5VUTZNNCXRJHVINBI3RA4UIZIOEEQ/)：作者报告内置程序可启动，线程没有公开源码 URL 或补丁。维护者提出 PE 工具链与 ARM64EC 类混合架构要求；不能将该邮件当成已经取得的完整移植源码。

Proton11 的 [ARM64 构建设计](https://github.com/ValveSoftware/Proton/blob/bc896217ffcd67f892f952b010f898fe7bd0f0a1/Makefile.in) 可供借鉴：Unix Wine 原生编译，Windows CPU 执行模块独立构建。Wine11 仍有 ELF builtin 加载路径，故第一个原生 Winelib 验证目标不要求先实现完整 LoongArch COFF 工具链。现有 Clang22 的两个 LoongArch Windows target 探针虽退出 0，实际均生成 ELF64，而非 COFF/PE；仅以编译器退出成功不能认定 PE 工具链已具备。

游戏指令转译层须单独适配。Box64 已有原生 LA64 dynarec，但其 [WoW64 DLL 构建接口](https://github.com/ptitSeb/box64/blob/2da641175cc2525f98f14b71427b914312184173/wine/wow64/CMakeLists.txt#L156) 当前限定 ARM64，EC 上下文也依赖 ARM64EC，不能直接作为龙芯 Windows CPU 插件使用。上述原生 host 路径与整套 x86 Unix Wine/Proton 经 LATX 执行不同。

Steam 集成还有独立 ABI 问题：[lsteamclient/unixlib.cpp](https://github.com/ValveSoftware/Proton/blob/bc896217ffcd67f892f952b010f898fe7bd0f0a1/lsteamclient/unixlib.cpp#L760) 仅选择 sdk32/sdk64/sdkarm64 并直接加载 `steamclient.so`；LoongArch 本地 ELF 不能直接加载 x86 闭源客户端库，需真正的龙芯库或额外 ABI/进程桥。[Proton README](https://github.com/ValveSoftware/Proton/blob/bc896217ffcd67f892f952b010f898fe7bd0f0a1/README.md#arm64-builds) 对 ARM64 也明确要求原生构建环境，不能据该模式承诺现有 x86 Steam 能直接配合原生龙芯 Proton。

后续完整移植需要维护的源码边界为 `ValveSoftware/Proton` 和 `ValveSoftware/wine`；若选用 Box64 的 LA64 执行后端，还需维护 `ptitSeb/box64` 的 Windows 接口改动。旧 la64-wine 是可固定引用的架构素材，不必因此永久把 Wine 8.21 整库替换为 Proton11 基线。LLVM/MinGW、图形和媒体依赖仅在实际验证需要改源时追加，不把所有依赖泛列为必须 fork。

上述为源码能力与实现边界核查；完整原生 Wine/Proton、Windows 程序执行及 Steam 游戏验证尚未完成。静态调查证据和固定源码位置另保存在 `/tmp/proton-native-port-20261029/PROTON_NATIVE_PATH_FINDINGS.md`（目录日期为任务名称，实际检查时间为 2026-10-28）。
