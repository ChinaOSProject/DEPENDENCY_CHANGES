libwebrtc Linux 线程头文件兼容修复（Firefox 157.0.1，原生 LoongArch64）

本次修复 Firefox 实际内置的 libwebrtc，使 `rtc_base/platform_thread_types.cc` 能在旧龙芯 Linux UAPI 上编译。修改仅删除冗余的非 ARM `<asm/unistd_64.h>` 引用区块，共 4 行；保留 `<sys/syscall.h>`、`syscall(__NR_gettid)`、pthread 线程引用和线程命名实现。

源码与版本依据：

- 依赖 GitHub 仓库：[mozilla/libwebrtc](https://github.com/mozilla/libwebrtc)。主 Agent 已独立 clone，核对的 `master` 固定提交为 `1389c76d9c79839a2ca069df1db48aa3f2e6a1ac`，日期 2022-05-12。参考仓库保持 `--no-checkout`，只核对 Git 中的指定源文件对象，未物化工作树，不依据该空工作树的 diff 声明源码修改或干净状态。该旧 fork 的同名源文件本来就只包含 `<sys/syscall.h>`，未含本次失败的 `<asm/unistd_64.h>`；没有对这个不同版本制造补丁或提交。
- 实际修改和编译对象是 [Firefox 固定提交 `0c469c2352451630bc69fc328c9f0c589c6c534d` 内的 vendored 文件](https://github.com/mozilla-firefox/firefox/blob/0c469c2352451630bc69fc328c9f0c589c6c534d/third_party/libwebrtc/rtc_base/platform_thread_types.cc)。`third_party/libwebrtc/README.mozilla.last-vendor` 记载 2026-08-26 更新及基线标记 `9ea5afcad0`；本报告文件名采用此 vendor 标记，不将它宣称为已解析的完整上游 Git SHA。
- GitHub API 对上述短标记无法解析，Google Gitiles 访问超时。独立旧 fork 与 Firefox vendor 的 namespace、其他平台实现和 Mozilla 补丁也不同，未用旧 fork 替换 Firefox 的依赖目录。
- 修复前该源文件 SHA-256：`2c95081db70d708ce07c0c628bca867ba5ac90b176820de7b8c6189c6a00b857`；修复后：`53e116a421caafb2884159155acbd41e6db7e0edb5173eab3fc6f2b80c766d69`。

失败原因与兼容性依据：

2026-10-28 的 `build-clang22-02.log` 在 `132:02.47` 记录该文件编译失败：`fatal error: 'asm/unistd_64.h' file not found`。目标开发头文件提供 `<asm/unistd.h>`，由 `<sys/syscall.h>` 按实际架构包含，继而提供 `__NR_gettid`；该架构不提供被硬编码的 `<asm/unistd_64.h>`。

现代系统兼容性依据是保留 libc 的架构选择入口，而不直接假定内核头文件布局。例如 [glibc 2.42 的 Linux `sys/syscall.h`](https://github.com/bminor/glibc/blob/glibc-2.42/sysdeps/unix/sysv/linux/sys/syscall.h) 仍包含 `<asm/unistd.h>` 和 `<bits/syscall.h>`。这属于源码核对依据；本次未在另一台现代发行版上执行运行测试。

原生验证在专属 rootless Podman 容器 `firefox-build-20261028` 完成，未向宿主全局安装依赖。使用原生旧 ABI Clang 22.1.8、正式 LLVM 22 LLD，以及项目隔离的 GCC 13 libstdc++/头文件；编译定义 `WEBRTC_LINUX`、`WEBRTC_POSIX`，以 C++20、`-O2 -Wall -Wextra -Werror -pthread` 实际编译修复后的源文件，而非替代实现。链接明确指定旧系统 loader `/lib64/ld.so.1`。

临时行为验证程序链接上述真实对象，并执行以下检查：

- 主线程 `CurrentThreadId()` 等于进程 ID 和内核 `gettid`，线程引用等于 `pthread_self()`。
- 8 个同时存活的工作线程具有各自稳定、互不相同且不同于主线程的线程 ID；每个 ID 与内核结果一致。
- `CurrentThreadRef()` 和 `IsThreadRefEqual()` 正确比较本线程及其他仍存活的线程引用。
- 主线程及全部工作线程经 `SetCurrentThreadName()` 设置名称，使用 `pthread_getname_np()` 读回一致；工作线程还使用 `prctl(PR_GET_NAME)` 再次读回一致。
- 全部工作线程成功 join，屏障正常销毁；程序返回 0，并输出 `PASS platform thread behavior: 8 concurrent workers`。

ABI 核查对真实对象和测试程序均通过：ELF64 little-endian，机器号 258，`e_flags=0x3`。测试程序的 interpreter 为 `/lib64/ld.so.1`，直接所需 GLIBC 符号版本最高 2.27，低于目标 glibc 2.28；没有 `GLIBC_PRIVATE` 或 `RPATH` / `RUNPATH`。未改写 ELF flags，未伪造系统头文件，未关闭 WebRTC 或忽略编译错误。

验证记录位于构建目录外置工作区：

- 宿主路径 `/tmp/firefox-build-20261028/webrtc-platform-thread-v0/`，容器中对应 `/work/webrtc-platform-thread-v0/`。
- `run-probe.sh`、`thread_probe.cc`：验证命令和临时程序，未提交到软件源码仓库。
- `probe.log`：线程行为结果、源文件与产物 SHA-256、自动 ABI 检查结果。
- `elf-audit.log`：`readelf -h -l -d --version-info` 输出。
- `platform-thread-linux-uapi.patch`：供 Firefox 单点审查的实际 4 行删除补丁；`git diff --check` 通过。

修复和上述单元的原生验证已完成，容器已明确交还 Firefox Agent 继续增量编译。实际 vendor 修改由主项目审查，以 Firefox 单点提交 `4e12201c1c0ac047730e960a254ecffb9f54296c` 集成。完整 Firefox 链接、浏览器运行、最终包 ABI 与交付状态以 Firefox 主报告为准；这里不将单个源文件验证当作完整浏览器验证。此项目作为 Firefox 内置依赖，不额外创建无独立用途的 `.deb` 包；报告由主 Agent 统一归档发布。
