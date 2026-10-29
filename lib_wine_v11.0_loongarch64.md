# Valve Wine 11.0：原生 LoongArch64 核心移植

更新日期：2026-10-29（Asia/Shanghai）。已编译并运行原生 ELF / Winelib 核心，且在真实安装布局中通过普通 main 测试。此为 Proton 的运行依赖核心子集；完整现代 Wine 前缀、GUI、Windows 客体执行和 Steam 游戏仍未完成，未生成冒充完整应用的 `.deb`。

实际适配上游为 <https://github.com/ValveSoftware/wine>，Proton 11.0-2e 固定提交 `e60892cc2398b0a83c332a2078c7b85e47bdb42b`，构建版本字符串 `wine-11.0`。本地目录 `/home/laevatein/ChinaOSProject/wine-proton`，最终已提交检查点为 `0fbb98ef4dcc44d035d314557ea9ce530f645fb7`；其中可复用 core 安装树保持在已实跑的 `a439a17d97357d243fe406e0e7667fe4755bbe6f`。后续提交为已验证的 CRT 平方根实现。未以 Wine8 原型替换这个固定基线；需要维护的正式 Wine fork 是 `ValveSoftware/wine`。

## 隔离环境与原生 ABI

全部编译位于专属 rootless Podman `wine-proton-native-20261028`，工作 `/tmp/wine-proton-native-20261028`，对象 `/work/obj`，最大 `make -j2`。没有向宿主全局安装依赖或修改宿主库。主机/容器基座为麒麟 V10 SP1 / glibc 2.28，旧 LoongArch LP64 ABI：ELF machine 258、flags `0x3`、解释器 `/lib64/ld.so.1`。

原生编译器为真正 ABI v0 的 Clang 22.1.8、LLVM/LLD 22，私有 GCC13 runtime 只在构建作用域使用；GNU8/GAS/GNU ld 另验证关键生成器、DWARF、原子和 CRT 头行为。配置为 `--prefix=/opt/wine-proton-native --enable-win64 --enable-archs= --without-mingw --disable-tests`，C/C++ `-O2 -g`。

`--enable-archs=` 只选择 native ELF；没有声明不存在的 LoongArch Windows COFF 编译器。`loongarch64-windows` 中的 PE placeholder/data-only 模块只提供 Wine 元数据，执行代码来自 `loongarch64-unix/*.dll.so`。

## 必要源码提交

分支严格按 `upstream/main → debian10 → loong64` 传播。通用修复先在 debian10 提交，再合并到架构分支。所有 tracked 源码修改已提交；官方构建所需的未跟踪 configure/config.h.in/syscalls/Vulkan 生成文件保留供后续构建，没有混入补丁。临时测试、容器 helper、日志和 stage 在仓库外。

| 提交 | 修改 |
| --- | --- |
| `c9a014f` | makedep 区分原生 placeholder 目录和文件名。 |
| `3ec68f3` | 原生架构、Winelib 指针/CONTEXT 类型和真实 r21 TEB ABI。 |
| `e246713` | native ELF builtin、IAT、delay import 和 relay 生成器。 |
| `9142b41` | 检测旧内核头的 userfaultfd write-protection 能力，保留真实平台 syscall 编号，使用已有软件 write-watch 回退。 |
| `c0f829f` | 真正链接探测并使用 libatomic 实现 128 位 Interlocked。 |
| `28cb841` | 原生 preloader，合法低镜像地址、映射 ELF/PHDR 和 Linux syscall 错误归一。 |
| `6175b31` | CRT-first 时在 typedef 之前建立正确 64 位指针 ABI。 |
| `2194e5c` | 原生 ELF 64 KiB 对齐、PE 元数据位于函数前，避免负 RVA 转 DWORD 后加 4 GiB。 |
| `258ce35` | 真实非局部跳转和优化编译下的异常帧登记。 |
| `313d425` | native ntdll syscall、线程、信号、上下文和 DWARF 展开后端。 |
| `4f8da27` | 原生 kernelbase DebugBreak 指令。 |
| `9a9f0fa` | 用官方工具单独同步固定 Valve 源码原本陈旧的 protocol 生成头；与架构新增区分。 |
| `640d80e` | server 传输原生 GPR 和完整 256 位 LASX 上下文。 |
| `d260755` | builtin 搜索缓冲区按实际架构目录长度分配，修复 installed 模式 Loong 较长路径损坏堆。 |
| `bb85b01` | 无不同 paired ABI 时不把当前 loader 选作 alternate，修复 installed Winelib 错转 start.exe。 |
| `0fbb98e` | 使用原生 fsqrt.s/fsqrt.d 实现 Wine11 新增的 CRT sqrt/sqrtf 底层接口；GNU8/Clang22 均实跑 100 组精确平方、signed zero、NaN 和 infinity。 |

未推送源码或创建远端 fork；本地分支与已提交补丁可供后续 fork 维护。

## 已实际运行的行为

- syscall/frame 使用 r21 保存 Wine TEB，保留 Linux r2 TLS；真正创建线程、等待和返回 37，主/子 TEB 与 Linux TLS 独立。
- RtlCaptureContext/NtContinue 恢复 PC/SP/GPR 和完整 256 位 LASX；非 self 的 Suspend/Get/Set/Resume 也实际修改并恢复 GPR/LASX，线程继续运行并返回 37。
- Clang `-O2` 的软异常、真实 NULL 写入 SIGSEGV、嵌套 finally/unwind 均通过。关键修复为异常链 push/pop 的 volatile asm + memory clobber，避免 TEB.ExceptionList 写入被优化成死 store；没有禁用优化或跳过真实 fault。
- native ELF 展开使用真正 `_Unwind_Find_FDE` 和 Wine DWARF 解释器；Loong FPR 的 DWARF 编号为 32–63，fs0 为 56。真实 `RtlVirtualUnwind2 → Unix unwind`、`RtlWalkFrameChain` 和 `RtlCaptureStackBackTrace` 已通过，核对返回地址、CFA/SP 和保存的 GPR/FPR。不是空 unwind stub。
- 128 位 CAS 在 GCC8/Clang22 下分别以 4 线程合计 40000 次操作、成功/失败 expected 值更新验证。使用真实 libatomic，不宣称 lock-free。
- 真正的 import/delay probe 覆盖 520 个导入、第二 DLL descriptor、IAT 偏移 ≥4096、10 个整数/10 个 double 和栈参数、解析器故意破坏 f0–f7、跨 DSO RA/TLS；GNU8/GAS/ld 与 Clang22/LLVM 均通过。Relay 的参数和双寄存器返回也双工具链通过。
- CRT-first 先包含 stddef/stdint，再包含 Wine Windows 头；GCC8/Clang22 的 C/C++ 四种组合均验证 pointer/uintptr/size/ptrdiff 为 8、LONG 为 4，并真实执行高位指针 roundtrip。
- 普通 WineCRT0 main 通过实际 argc/argv `(42,37)` 和 return 0。这不是现代 UCRT 全部启动/退出路径验证；旧 Wine8 的 UCRT stdio/_crt_atexit 结果另见其报告。

原始证据位于 `/tmp/wine-proton-native-20261028/`，以下 `.log/.exit-code` 均为退出 0：`native-core-build-10`、`native-hello-run-05`、`native-thread-run-01`、`native-context-run-01`、`native-peer-context-run-01`、`native-seh-run-03`、`native-main-run-01`、`native-dwarf-run-02`、`dwarf-probe-03`、`dwarf-probe-gcc8-01`、`stage-version-01`、`stage-main-pair-03`。机器可读汇总为 `modern-wine-results.json`，CRT-first 结果为 `crt-first-results.json`。

## 真实安装布局和 Proton 接通

使用本项目 `tools/install` 和生成 Makefile 的真实目的地安装已构建 core 子集；外置 `stage-core.make` 没有冒充完整 `make install`。可复用树为 `/tmp/wine-proton-native-20261028/stage-core/opt/wine-proton-native/`：

```
bin/wine
bin/wineserver
lib/wine/loongarch64-unix/wine
lib/wine/loongarch64-unix/wine-preloader
lib/wine/loongarch64-unix/ntdll.so
lib/wine/loongarch64-unix/{ntdll,kernel32,kernelbase}.dll.so
lib/wine/loongarch64-windows/{ntdll,kernel32,kernelbase,apisetschema}.dll
share/wine/wine.inf
share/wine/nls/*.nls
```

preloader 镜像基址使用本机可容纳的 `0x7d4000000`。按实际 preloader 链接路径探测 GNU ld `-Ttext-segment` 或 LLD `--image-base`，保留已映射 ELF/PHDR 和 Wine11 GDB metadata；没有禁用 preloader。Linux syscall 的 `[-4095,-1]` 统一为调用方预期的 -1。

正式 installed 测试将真正 Winelib `.exe.so` 与 Wine 工具生成的对应 placeholder 安装为配对，经公共 `bin/wine` 和实际 preloader 运行并退出 0。测试配对已从可复用 core stage 移除，原始文件留在 probes；移除清单为 `stage-selftest-removed.json`。

主 Agent 又将该实际树复制到 Proton 的 `files/`，按其规则移公共入口至 `bin-loongarch64`，使用真实 `Session.init_wine/run_proc` 执行同一普通 main 返回 0；没有 mock 架构或设置 bootstrap 环境。用本体 `default_pfx.setup_dll_symlinks` 对实际 native placeholder 建立相对链接，再从 `C:\windows\system32\native-main.exe` 启动也返回 0。这仅验证 core 的路径/环境/链接，不等于完整 Proton setup。证据为 `/tmp/proton-native-port-20261029/logs/modern-core-session-root.{log,json,exit}`。

最终 `stage-audit.json` 核验 89 文件、8 ELF 和 4 PE 元数据：ELF 全部 machine 258 / flags `0x3`，有解释器者均为 `/lib64/ld.so.1`，最高 GLIBC 2.27，无 GLIBC_PRIVATE、路径型 NEEDED 或绝对 RPATH/RUNPATH；preloader 为静态。core 需要系统 libatomic.so.1、libgcc_s.so.1，不需要 libstdc++。Unix ntdll.so 为 3159088 字节，SHA256 `5f4377debd508bf547466fb2d8203f969c7f648dd9dafca973f141d7c0e6d2ed`。

## 剩余运行边界

- 继续构建现代正常前缀时，UCRT/MSVCRT 以真实 `-z defs` 链接失败，缺 `get_exception_pc`、`call_catch_handler`、`call_unwind_handler`、`handle_fpieee_flt` 四个架构接口，日志 `normal-prefix-build-05.log`。前三项涉及 MSVC funclet/nonvolatile-frame ABI，而当前 native ELF 展开不等于 PE 语言处理器或 `UNWIND_CONSOLIDATE` 已实现；没有用普通 C 调用或空实现把它算作完成。图形/安装 API 的独立标准目标在 `normal-prefix-build-06` 退出 0；没有忽略 CRT 的真实失败。
- 对真实 C++ try/catch + destructor 做编译器对照：`loongarch64-pc-windows-msvc` 在 `cleanupret` 指令选择失败，退出 1 且无对象；`loongarch64-w64-windows-gnu` 虽退出 0，却产出带 `__gxx_personality_v0` 的 Linux ELF，与相同源码的 Linux target 对象 SHA256 完全一致。同一 Clang 的 ARM64 MSVC 正控制成功产出 COFF-ARM64、`.pdata/.xdata` 和 `__CxxFrameHandler3`。这验证当前工具链没有提供可用的原生 Loong Windows/MSVC 异常 ABI；不能用接受 target 字符串替代支持。原始命令、对象和诊断见 `probes/modern-crt-target/results.json`。
- 七个文件的正常前缀实验（metadata、单参数 setjmp、GDI 原生路径）保存在仓库外 `normal-prefix-unverified-owner-takeover.patch`，未能进行端到端验证，已从实际 tracked 分支还原；不混入这 16 个已验证源码提交。后续完整 CRT 需要真正的 Windows EH/funclet 或其他可验证 ABI 实现，独立 native core 检查点保持可用。
- core stage 尚无 wineboot.exe 和其完整 DLL 闭包，自动初始化实际报 `c0000135`；通过的 core 程序日志保留这条错误。现代 normal prefix 未完成，旧 Wine8 正常前缀不能代替它。
- 本机无法映射通用高段 `00007ffffe000000-00007fffffff0000`，preloader reserve 警告保留；不得宣称该高段映射已完成。
- Win32k user callbacks、跨机器/WoW context、LoongArch PE language-specific unwind、非空 instrumentation callback 尚未实现，对应路径明确失败。Native ELF CFI 展开已实现。
- 没有 x86/ARM Windows CPU 后端或真正 LoongArch Windows COFF 工具链；server 只声明实际原生 LoongArch，不虚报已有 guest 能力。
- 全 Wine DLL/程序集、现代 GUI/Direct3D、兼容 Steam 库真实调用、完整 Proton 前缀和游戏均未验证。ARG_INT128 relay 保持直接函数调用，既有 relay 类型串未编码其宽度，不宣称覆盖该 tracing 情况。

本阶段无需 fork LLVM、Autoconf 或其他未改源工具。正式需要维护 `ValveSoftware/wine`；Proton 本体和 Firefox 的实际改动分别见其报告。旧 `loongarch-moe/la64-wine` 仅在独立长期维护 Wine8 原型时需要额外 fork。
