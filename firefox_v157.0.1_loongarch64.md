# Firefox 157.0.1 LoongArch64 适配报告

2026-10-28 已完成原生编译、Debian 打包和实际运行验证。交付包适用于本机龙芯旧 ABI v0、glibc 2.28；不是 x86 转译版本。本地构建使用默认非官方 Nightly 品牌、en-US 语言，未启用自动更新。上游尚未支持 LoongArch Linux 内容进程 seccomp 沙箱，本包沿用上游默认配置，具体限制见下文。

## 交付

- 包名 `firefox`，版本 `157.0.1-1`，架构 `loongarch64`。
- 路径：`../binaries/firefox/loongarch64/firefox_v157.0.1_loongarch64.deb`。
- 大小：79,988,136 字节。
- SHA256：`67cea3a9f31550106a267c08e29b72dfae5083e388d44d3adc2aec4461af3e34`。
- [GitHub 二进制](https://github.com/ChinaOSProject/binaries/blob/90e1e5e70824574c53df24dffb57f94170b7d390/firefox/loongarch64/firefox_v157.0.1_loongarch64.deb)，发布提交 `90e1e5e70824574c53df24dffb57f94170b7d390`。
- 发布后从该固定提交下载完整线上副本，SHA256 与本地交付包一致。
- `/usr/bin/firefox` 为指向应用目录启动器的相对符号链接；实际程序和私有运行库位于 `/usr/lib/firefox`。启动器仅在应用进程范围加入自身目录的库搜索路径，不修改宿主系统配置。

## 源码与最小修复

上游：<https://github.com/mozilla-firefox/firefox>。固定标签 `FIREFOX_157_0_1_RELEASE`，原始提交 `0c469c2352451630bc69fc328c9f0c589c6c534d`。修改先提交至 `debian10`，再 fast-forward 至 `loong64`；两分支最终均为 `d7ef396c4fd88eab78215ba2dd092ed041c7a5d0`，源码工作树干净。

1. `4e12201c1c0ac047730e960a254ecffb9f54296c`：删除 vendored libwebrtc `platform_thread_types.cc` 中冗余的 `<asm/unistd_64.h>` 四行区块；已有 `<sys/syscall.h>` 提供所需声明。真实线程实现经八个并发线程验证 ID、引用与名称，见 [libwebrtc 报告](lib_libwebrtc_v9ea5afcad0_loongarch64.md)。
2. `d7ef396c4fd88eab78215ba2dd092ed041c7a5d0`：仅在 Linux `TARGET_CPU=loongarch64` 的 libxul 链接中增加源树相对的 `loongarch64.ld` 与 `EXTRA_LINK_DEPS`。标准 `INSERT BEFORE .text` 脚本将 `.plt` 放到 `.text` 前，解决 GCC13 `crtbeginS.o::__do_global_dtors_aux` 调用 `__cxa_finalize@plt` 的旧 SOP 短分支溢出。真实调用距离由 147,816,936 降至 -43,656 字节；最终正常 mach 构建重新链接成功。没有修改 SDK、篡改 ELF flags 或忽略重定位错误。

## 隔离环境与工具

目标宿主为 Loongson-3A5000、4 核、16 GiB RAM、Kylin V10 SP1，实际 glibc `2.28-10.kylin.35k1.3`，ELF machine 258 / flags `0x3`，解释器 `/lib64/ld.so.1`。全部编译在 rootless Podman `firefox-build-20261028` 中完成；镜像 `localhost/firefox-build:157.0.1-buster-loongarch64-online`，ID `8bcc5b1f391af65b4d0bc56bba2a3bc8e397594be0f39e0ef43435cd83cc4c07`。构建容器使用 host 网络和 `seccomp=unconfined`。依赖、对象、缓存和临时工具留在专用容器或 `/tmp/firefox-build-20261028`，未向宿主全局安装构建工具或库。

- 原生 C/C++：匹配 Loongnix ABI v0 的 Clang/libclang `22.1.8`，复用正式 LLVM22 后端和 LLD。由源码在专用容器内编译，详见 [Clang 报告](lib_clang_v22.1.8_loongarch64.md)。GCC `13.4.0` 提供头文件与 C++ 运行库。
- [Rust SDK 1.98.1-2](https://github.com/ChinaOSProject/binaries/blob/main/rustc/loongarch64/rustc_v1.98.1_loongarch64.deb)：200419948 字节，SHA256 `c875e0565f04b3989d008faf283f8583d200bf3aeb140926e0ec3329f3f0e734`。
- [LLVM22 SDK 22.1.8+rust1.98.1-2](https://github.com/ChinaOSProject/binaries/blob/main/llvm-22/loongarch64/llvm-22_v22.1.8_loongarch64.deb)：619740072 字节，SHA256 `c844ce421e4265ab154f414849fce25b6d1d73f45d21a52030866bcfe7911f44`。
- cbindgen `0.29.4` 使用固定 Cargo.lock 原生构建并实际生成 Firefox FFI，见 [cbindgen 报告](lib_cbindgen_v0.29.4_loongarch64.md)。Python `3.11.9`，Node.js `24.21.0`。
- 六项 RLBox Wasm 库均保留；独立 Wasm 目标使用 Clang `21.1.8` 和 [WASI SDK 27.0](https://github.com/WebAssembly/wasi-sdk/releases/tag/wasi-sdk-27)，不用于生成本体原生 ELF。WASI sysroot SHA256 `7110ac48f5d0b1f6ab67d57aecf52450540dddd790cafdc45f0fdfb429bdab84`，Wasm compiler-rt 归档 SHA256 `9e0f382110a3cf9196f02432c8f2e54d151515de36f9311c8c16073f6e6b16d3`。

## 有效构建配置

实际 `/work/mozconfig.clang22-v0`：

```sh
mk_add_options MOZ_OBJDIR=/work/obj-clang22-v0
mk_add_options MOZ_MAKE_FLAGS=-j3
ac_add_options --enable-application=browser
ac_add_options --disable-bootstrap
ac_add_options --disable-debug
ac_add_options --disable-debug-symbols
ac_add_options --disable-tests
ac_add_options --disable-updater
ac_add_options --enable-optimize=-O2
ac_add_options --disable-stdcxx-compat
ac_add_options --enable-linker=lld
ac_add_options --with-clang-path=/work/clang22-v0-20261028/install/bin/clang
ac_add_options --with-libclang-path=/work/clang22-v0-20261028/install/lib
ac_add_options --with-wasi-sysroot=/work/tools/sysroot-wasm32-wasi
export CC="/work/clang22-v0-20261028/install/bin/clang --gcc-toolchain=/opt/firefox-tools/gcc13 -mcmodel=large"
export CXX="/work/clang22-v0-20261028/install/bin/clang++ --gcc-toolchain=/opt/firefox-tools/gcc13 -mcmodel=large"
export HOST_CC="/work/clang22-v0-20261028/install/bin/clang --gcc-toolchain=/opt/firefox-tools/gcc13 -mcmodel=large"
export HOST_CXX="/work/clang22-v0-20261028/install/bin/clang++ --gcc-toolchain=/opt/firefox-tools/gcc13 -mcmodel=large"
export LDFLAGS="-L/opt/firefox-tools/gcc13/lib -Wl,--dynamic-linker=/lib64/ld.so.1"
export HOST_LDFLAGS="-L/opt/firefox-tools/gcc13/lib -Wl,--dynamic-linker=/lib64/ld.so.1"
export RUSTC=/work/tools/rust-v0/usr/bin/rustc
export CARGO=/work/tools/rust-v0/usr/bin/cargo
export RUSTFMT=/work/tools/rust-v0/usr/bin/rustfmt
export RUSTFLAGS="-Ccode-model=large"
export CBINDGEN=/work/tools/cbindgen/bin/cbindgen
export NODEJS=/opt/firefox-tools/node/bin/node
export WASM_CC=/work/tools/bin/wasm-clang
export WASM_CXX=/work/tools/bin/wasm-clang++
```

构建入口通过进程级 `MACH_BUILD_PYTHON_NATIVE_PACKAGE_SOURCE=none`、`MOZCONFIG`、`MOZBUILD_STATE_PATH`、`CARGO_HOME` 以及工具链 PATH 选择上述工具。`LIBRARY_PATH=/opt/firefox-tools/gcc13/lib` 用于配置阶段遗漏 LDFLAGS 的链接探测；构建进程的 `LD_LIBRARY_PATH` 包含 GCC13、Clang22 与工具 LLVM21 的私有目录。这些构建路径不进入交付程序的 RPATH 或 DT_NEEDED。执行 `/work/build-env-clang22-v0.sh build -j3` 和同一入口的 `package`，完整日志保存在仓库外。

C/C++ 和 Rust 统一使用 large code-model：正式 Rust 标准库的 bitcode 已使用 large，应用默认模型会在跨 crate LTO 合并时报 4/3 冲突；本包保留 LTO，统一使用 `-Ccode-model=large`。旧 ABI LLD 没有 LoongArch thunk，而默认 C++ 短分支在大型 libxul 超过 ±128 MiB，因此本体和宿主 CC/CXX 均增加 `-mcmodel=large`。独立约 256 MiB DSO 的远调用、回调和跨远栈帧异常处理均实测返回 42。没有通过关闭 LTO、JS、Wasm 或 SIMD 绕过问题。

原 GCC13 不接受 Firefox 所需的 `-mlsx`；现成 Clang21 生成的原生对象又是 flags `0x43`，不适用于本机旧 ABI，因此使用真正的 Clang22 ABI v0 前端。真实 libvpx LSX 对象与 Skia LASX 对象均为 flags `0x3`，反汇编分别确认 `vld`/`vldx` 与 `xvld`/`xvldi` 指令。

`build-clang22-04.log` 的 Rust optimized LTO 已通过；全部编译结束后审计最终 libxul 直接输入与六份 native 静态库 response lists，共 2447 个唯一对象，旧对象数 0、全部 machine 258 / flags `0x3`。先前构建中途的旧对象是等待 pre-compile 依赖的后置调度，未回改时间戳或强制跳过目标。接入 PLT 布局修复后，`build-clang22-05.log` 正常重新链接并以 0 退出（2 分 04 秒）；`package-clang22-01.log` 以 0 退出（3 分 05 秒），生成真正 packager 目录 `dist/firefox`，打包未使用 `dist/bin` 的符号链接树。

## Debian 包与 ABI

由于可选 stdc++compat shim 与 GCC13 私有 `std::thread::_State` 不兼容，配置使用 `--disable-stdcxx-compat`。包内携带经 strip 的私有 `libstdc++.so.6.0.32` 及其许可证，连同 Firefox MPL 和组件许可证；没有要求宿主升级全局 libstdc++。不带私有启动器时系统 GCC8 库明确缺少 `GLIBCXX_3.4.31`；使用最终应用相对启动器后，所有实际测试均确认加载包内运行库。

全部依赖通过标准 `dpkg-shlibdeps` 计算，没有使用 ignore-missing-info。仅在打包计算目录内使用私有库归属元数据并排除 firefox 自依赖，未安装全局公共 shlibs。构建容器的 libgcc symbols 将已有 GCC_4.5.0 错配到较新发行修订号，故先逐符号核验目标宿主实际库：35 个所需 GCC 符号全部存在、缺失 0，再在外置 dpkg 元数据副本中使用目标宿主的真实 symbols 文件重新计算。最终得到 `libgcc1 (>= 1:8.3.0-6.lnd.vec.36.1)`，与宿主一致；不是手工削减 Depends。全部 27 项 Depends 均由宿主已安装版本满足，不含系统 libstdc++6 升级依赖。

最终归档再次解包后审计 23 个 ELF：全部 LoongArch machine 258 / flags `0x3`，可执行文件使用 `/lib64/ld.so.1`，所有 GLIBC 需求不高于 2.28，无 GLIBC_PRIVATE、路径形式 DT_NEEDED 或绝对 RPATH/RUNPATH。最终包的 md5sums 校验通过。

## 实际运行验证

测试均使用最终 `.deb` 的解包或安装入口，清除 SDK 的 LD_LIBRARY_PATH/LD_PRELOAD，使用独立新 profile 和本地 HTTP 测试页；未访问个人浏览器 profile。测试页实际验证 JavaScript、Wasm 返回 42、Canvas 像素、XHR 和 Cookie，三次均全部通过，并通过 `/proc` 映射确认包内私有 C++ 运行库被实际加载。

| 环境 | 实测结果 |
| --- | --- |
| Kylin 宿主 CLI | 最终归档解包入口 `--version` 输出 `Mozilla Firefox 157.0.1`，`--help` 正常，均 exit 0 |
| Kylin 宿主 headless | 功能全部 true，exit 0，2.288 秒，生成 1024×768 PNG（26727 字节） |
| Kylin 宿主 X11 GUI | 功能全部 true，exit 0，5.156 秒；实际窗口 `IsViewable`，标题 `Firefox LoongArch smoke — Nightly`，仅关闭本次测试窗口 |
| 干净 Debian buster 容器 | 无 SDK 挂载，直接 apt 安装最终包及测试驱动；非 root 用户从安装入口完成 CLI/headless，功能全部 true，exit 0，2.25 秒 |
| 包管理完整性 | 干净容器 `apt-get check`、`dpkg --audit`、`dpkg --verify firefox` 均 exit 0；系统仍使用 GCC8 运行库 `vec.43` |

干净运行容器为 `firefox-runtime-20261028`，基于 `cr.loongnix.cn/library/debian:buster-slim`，基座 glibc 2.28，不依赖构建容器或 SDK 路径。上述为实际功能和安装 smoke，未运行完整 Mozilla 自动化测试套件。

## 已知限制：内容进程沙箱

上游 `toolkit/moz.configure::sandbox_default` 的 Linux CPU 白名单只有 x86/x86_64/ARM/AArch64。`seccomp_macros.h` 缺少 LoongArch 寄存器、ucontext 与 SECCOMP_ARCH 定义，`linux_syscall_ranges.h`、`linux_signal.h` 和 `seccomp-bpf/syscall.cc` 也缺少对应后端。不是添加一个 configure 条件就能完成的安全移植。本包沿用上游原生配置，未定义 MOZ_SANDBOX；保留的六项 RLBox 库不等同于进程 seccomp 沙箱。

宿主、构建容器和测试内容进程实际记录 `NoNewPrivs=0`、`Seccomp=0`。测试 JSON 分别记录 `functional_ok=true`、`sandboxed_content_observed=false`、`process_sandbox_status=upstream_unsupported`；没有将未启用的沙箱报告成已通过。后续新增该能力需单独实现和验证完整 LoongArch 进程沙箱后端。

## 本地证据

宿主证据根目录 `/tmp/firefox-build-20261028`：`build-clang22-05.log`、`package-clang22-01.log`、`native-rebuild-final.json`、`native-simd-final-proof.log`、`libxul-crt-call-layout-proof.log`、`firefox-final-deb-elf-audit.json`、`host-libgcc-symbol-audit.json`、`firefox-host-dependency-audit.json`、`final-deb-cli-version.log`、`final-deb-cli-help.log`、`final-deb-md5-check.log`，以及 `smoke-final-host-headless/result.json` 和 `smoke-final-host-gui/result.json`。安装和干净运行证据位于 `/tmp/firefox-runtime-20261028/install-final-deb.log` 与 `results/smoke-final-headless/result.json`。临时脚本、对象和日志未提交到 Firefox 源码仓库。
