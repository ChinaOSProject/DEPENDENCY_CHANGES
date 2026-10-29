# Wine 8.21：原生 LoongArch64 端口原型实验

更新日期：2026-10-29（Asia/Shanghai）。用途为 Proton 原生后端的架构验证素材；不是完整 Wine/Proton 成品，尚未交付应用 `.deb`。

实际被适配的 GitHub 仓库：<https://github.com/loongarch-moe/la64-wine>，固定 `9f5367cc56defca16452a63ad253c7c49f848871`，实际版本 Wine 8.21，原提交说明 `pre support`，仓库已归档。本机源码 `/home/laevatein/ChinaOSProject/wine`。长期 Proton 基线仍为单独维护的 `ValveSoftware/wine` 固定提交；没有用旧原型替换 Proton11 所固定的 Wine。

## 构建与 ABI

专属 rootless Podman 容器 `wine-native-20261028`；工作 `/tmp/wine-native-20261028`。仅在容器中安装和构建依赖，未污染宿主系统。GNU8 与真正的 Clang22.1.8 ABI v0 工具链分别验证；LLVM21 的新 ABI native 输出未混入本次运行时。

实际 loader/wine64、wineserver、Unix ntdll.so、native ntdll.dll.so 已编译。已检查的原生 ELF 均为 EM_LOONGARCH=258、flags `0x3`，动态解释器 `/lib64/ld.so.1`；核心 ABI 检查的 GLIBC 需求最高 2.27。Clang 生成的 16 字节原子调用使用真实 native libatomic 以 `-latomic` 链接，没有提供假原子实现。

Clang 的 `loongarch64-w64-mingw32` 和 `loongarch64-pc-windows-msvc` 探针虽编译退出 0，实际输出均为 ELF，不是 COFF/PE。本原型通过 Wine 原有 ELF builtin / Winelib 路径验证原生代码。构建树中的 PE placeholder 用于定位对应 ELF 模块，不能据此宣称已具备 LoongArch PE 编译器。

## 已提交的必要修复

分支为 `upstream/main → debian10 → loong64`。通用 C11 兼容修复放在 debian10，架构修复从其继承。未提交临时探针、容器构建文件、对象或本地环境路径。

| 提交 | 修复 |
| --- | --- |
| `02a931e` | winebuild 的架构 switch 分支使用独立作用域，使 GNU8 的 C11 声明合法。 |
| `eedd366` | basetsd 使用真实 `__loongarch_lp64` 检测；INT_PTR/UINT_PTR/ULONG_PTR 恢复为 8 字节。 |
| `91f7a5a` | 仅 LoongArch 汇编返回使用标准 `jirl $zero,$ra,0`，保留其他 ISA 代码。 |
| `0cb7983` | 外部尾分支使用两套工具链均能汇编、链接和动态解析的形式，去掉 Clang 不支持的 `%plt` 语法。 |
| `07ede48` | 修复原来仅 4 字节的 jump buffer 和无限循环；按 ABI 保存/恢复返回地址、栈、非易失寄存器、FCSR 与浮点寄存器。 |
| `2f4628e` | IAT 导入 thunk 真正加载函数指针再尾跳，保留调用者返回地址；避免对数据槽执行 `bl`，并正确处理非零槽偏移。 |
| `84e90db` | 实现 syscall/thread 入口桥：r21 保存 Wine TEB，保留 Linux r2 TLS，复制第 9 个起的栈参数，并保存完整浮点/向量上下文。 |
| `048d826` | 实现 Winelib 异常分派、注册帧处理和 unwind、上下文捕获/恢复、线程入口及 APC 调度所需路径。 |
| `5379a99` | LoongArch 静态 preloader 放在 `0x7d4000000`；原通用 `0x7d7d00000000` 超出本机地址范围，execve 实际 EINVAL 并 SIGSEGV。configure 使用匹配的 GNU Autoconf 2.72 正规再生，仅新增对应两行。 |
| `9a4196f` | preloader 将 Linux syscall 的负 errno 归一为调用方预期的 -1，避免把 mmap/open 失败误当有效地址或 fd。 |
| `0a40982` | 延迟导入 thunk 保留 a0–a7、f0–f7 和 RA，用真实 GOT 调用解析器并正确计算 descriptor/IAT 偏移；修复原实现把 SP 清零而崩溃。 |
| `a8c3850` | CRT 头文件使用真实 `__loongarch_lp64`，避免在 stdint/stdlib 先包含时把 size_t、ptrdiff_t、uintptr_t 定成 32 位。 |
| `bea1728` | SetupAPI/manifest/INF/wine.inf 使用真实原生目录、`loongarch64` identity、`.ntloongarch64` 节和 machine `0x6264`，建立正确的 64 位原生前缀。 |
| `7e8f247` | 只声明实际具备的原生 LoongArch machine，移除尚无后端的 ARM64/AMD64/i386/ARMNT 声明，避免 Wineboot 错建或运行跨架构目录。 |

## 已运行的验证

- 指针宽度修复前 GNU8 的静态断言失败，修复后 GNU8/Clang22 实际执行均报告指针整数类型为 8 字节；GNU13 头文件检查也通过。
- GNU8 与 Clang22 的实际返回/跨 DSO 尾调用通过；x86/i686 与 ARM64 的相关预处理输出与原基线一致。
- 使用实际 Wine setjmp 对象，在两套编译器下分别以 4 个 pthread × 200 轮验证 longjmp 73、0→1 和栈 canary。
- IAT 的真实动态共享库调用在 GNU8/GAS/ld 与 Clang22 下都验证了槽偏移 4096 和返回地址保持。
- `native-hello.exe.so` 经真实 loader、LdrInitializeThunk、kernel32/kernelbase 到达自定义程序入口；WriteFile 输出文本，ExitProcess 返回 0。
- `native-thread.exe.so` 使用真实 CreateThread、Sleep、线程 return 37、WaitForSingleObject 和 GetExitCodeThread，验证主/子 TEB 不同、各自 Linux r2 TLS 不被破坏，退出 0。
- `native-stack-args.exe.so` 调用 11 参数 NtCreateFile，以第 9 参数互斥 CreateOptions 验证 INVALID_PARAMETER，再以正确参数创建、写入、关闭文件，退出 0。
- `native-seh.exe.so` 实际执行 RaiseException、volatile 地址 0 写入触发 SIGSEGV、嵌套 finally/unwind 和外层 except；finally 恰执行一次，退出 0。
- `native-context.exe.so` 通过 RtlCaptureContext/NtContinue 改写并恢复 PC、SP、易失 GPR 和一个 LASX 寄存器的全部 256 位，退出 0。
- 新 preloader 在 GNU8 和 Clang22 下分别实际启动 Wine，退出 0；不存在的目标文件正确报告 could not open 并退出 1，ELF flags 均为 `0x3`。
- GCC8/GAS/ld 与 Clang22 的 520 项真实导入/延迟解析调用均通过，覆盖大于 4096 的 IAT 偏移、栈参数、浮点参数和调用者返回地址；解析器仅在首次需要时调用。
- CRT-first 的 size_t/ptrdiff_t/uintptr_t 为 8 字节、LONG 为 4 字节的断言在 GCC8/Clang22 通过；受影响的已构建核心运行库按正常依赖关系重新编译。

最初的 hello/thread/stack-args/SEH/context 架构探针使用 `WINEBOOTSTRAPMODE=1` 从构建树加载 builtin，自定义入口不是普通 CRT。下面单独记录随后真正完成的普通前缀和 CRT 测试；这些结果仍不证明 Windows PE 程序或游戏已可用。

## 普通前缀和 CRT 的真实结果

- 从空目录建立 `/work/prefix-normal-clean-20261029-01`，以实际原生 loader 运行 `wineboot --init` 返回 0 并打印 configuration updated，全程没有设置 `WINEBOOTSTRAPMODE`。System32 有 124 项，实际 kernel32/ntdll/setupapi/wineboot/rundll32/shell32 placeholder 均为 machine `0x6264`、PE32+ `0x20b`；它们由 native ELF builtin 支持，不是 LoongArch COFF 工具链证明。
- 用真实 Winegcc 的 `-mno-cygwin -nodefaultlibs -fshort-wchar -fno-builtin`、Wine CRT 头和实际 import/static 库编译普通 `int main`，从 `mainCRTStartup` 运行并返回 0。验证 argc/argv、高地址堆指针（`0xFFF...`）、UCRT stdio 文件往返、原生 machine 查询和 wildcard manifest 激活。
- 实际调用 UCRT 导出的 `_crt_atexit`，退出时打印 `normal CRT UCRT exit handler reached`。最初 plain `atexit` 的实验绑定到 glibc，未算作成功，也没有为此修改 UCRT 源码；最终结果来自 Wine 自己的 UCRT 退出表。
- 正常 rundll32 实际执行测试 INF，确认 `.ntloongarch64` 被选中、placeholder 源文件逐字节复制成功、缺少源文件时生成正确的原生 fallback placeholder。SystemSupportedProcessorArchitectures 实际返回唯一原生 `0x000f6264`，随后为零；全新前缀不再尝试 syswow64/sysarm32 rundll32。
- 对最终 loader、preloader、wineserver、ntdll Unix/Winelib 模块和普通 CRT 程序六个实际 ELF 核验，均为 machine 258 / flags `0x3`，最高 GLIBC 依赖 2.27。

证据为 `logs/runtime-wineboot-fresh-native-prefix-07.log`、`runtime-normal-crt-03.log`、`runtime-native-prefix-metadata-01.log`、`native-normal-prefix-validation.json`、`native-prefix-metadata-validation.json`，完整可复现命令见 `/tmp/wine-native-20261028/NORMAL_PREFIX_HANDOFF.md`。早期失败前缀保留了旧 timestamp，最终结果以新建 clean 前缀为准。

## 当前继续处理的边界

按真实加载链补编译了标准 native builtin 依赖，包括 advapi32、ucrtbase、ws2_32、sechost、shell32、shcore、imm32、ole32/combase/oleaut32/coml2 等，并显式重链受新生成器影响的实际 DLL；没有只重建 import library 便宣称生效。普通前缀已完成，但日志仍真实报告 `KeUserModeCallback` 未实现、缺少 explorer.exe 和系统菜单失败；初始化返回 0 不等于图形程序正确。

Win32k 用户回调、WoW64 跨机器上下文、PE unwind / 一般 native ELF DWARF unwind、部分栈回溯与架构 helper 尚未完成。当前注册帧 SEH 测试通过，不等于这些路径全部实现。明确的失败或未实现函数保留并记录，不用返回成功或仅关闭功能掩盖缺口。

现代 Valve Wine 由另一个专属源码目录和容器前移这些已验证实现，并重做 import/delay/relay 生成器；没有把旧原型中的所有代码无审查复制。现代 Wine 的实际能力和提交应以其独立记录为准。

## 证据、缓存及维护边界

证据目录 `/tmp/wine-native-20261028/logs/`：`abi-after.log`、`preprocess-compare.log`、`jmp-probe.log`、`iat-probe.log`、`runtime-native-hello-bridge-final.*`、`runtime-native-thread-bridge-final.*`、`runtime-native-stack-args-bridge-final.*`、`runtime-native-seh-final.*`、`runtime-native-context-final.*`、`preloader-abi-validation-root.json`、`runtime-normal-prefix-root-02.log` 等。探针源位于同一工作目录，未进入源码提交。

用户授权使用的外接 SSD 仅新增本任务缓存目录，没有格式化或更改已有文件。已校验的 Firefox 完成后对象缓存和容器归档约 17.44 GB 移至该盘；活动 Wine/Proton 对象和工具链保留在 ext4。GNU Autoconf 2.72 的原始 tarball也缓存于外盘，SHA256 `ba885c1319578d6c94d46e9b0dceb4014caafe2490e437a0dbca3f270a223f5a`，仅在专属容器内构建，无源码修改。

此仓库是可固定引用的移植素材。若单独长期维护 Wine8 原型，需要 fork `loongarch-moe/la64-wine`；完整 Proton11 应维护 `ValveSoftware/wine` 并前移必要实现，不必永久并行维护两个完整 Wine 分支。Autoconf、LLVM 等未改源工具不因此需要 fork。
