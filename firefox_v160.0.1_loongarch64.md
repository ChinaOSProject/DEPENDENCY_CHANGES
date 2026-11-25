# Firefox 160.0.1 LoongArch64 适配报告

## 源码与构建

- 上游标签 `FIREFOX_160_0_1_RELEASE` 对应 `58fed86b1b6864ad71b83c910a879d6f1a6b678b`。本地 `debian10` 和 `loong64` 均指向 `378eff036ef415dbaae27d48186f52fe0aa49149`。[Mozilla Firefox 源码](https://github.com/mozilla-firefox/firefox/tree/FIREFOX_160_0_1_RELEASE)
- 沿用两项补丁。`996d395d18` 移除 vendored libwebrtc 对架构专用 `asm/unistd_64.h` 的冗余包含，使用已有的 `sys/syscall.h`。`6c93a0a664` 在 Linux LoongArch 的 libxul 链接中将 `.plt` 放到 `.text` 前，避免 GCC CRT 的旧 ABI 短分支重定位溢出。
- 本次新增 `32e84ac076`。旧 ABI 汇编器拒绝 Firefox 160 源码中 XPCOM 的 `call36`、`ret`，同一实现还用了 `tail36`。三个位置改用大范围 PC 相对地址加载 `la.local` 和基础指令 `jirl`。ABI v0 Clang 22 和 ABI v1 Clang 21 均已通过汇编检查。目标宿主使用修复后的两个汇编对象，调用相距超过 128 MiB 的辅助函数，验证了参数传递、返回值和尾跳转。测试日志为 `logs/xptcall-runtime-probe.log`。
- 本次新增 `378eff036e`。LoongArch 的 wasm2c 使用显式线性内存边界检查，避免无 LSX 编译时的 SIMD 汇编约束错误。保留 RLBox、Wasm SIMD 和 mmap 分配。用本次 wasm2c 生成的测试模块在目标机验证，合法 SIMD 读取返回 42，跨界读取触发 `WASM_RT_TRAP_OOB`。日志为 `logs/rlbox-bounds-probe.log`。
- 使用 rootless Podman 容器 `firefox-mozilla-build-160-v0`，镜像 `localhost/firefox-build:157.0.1-buster-loongarch64-online`。在 `/mnt/chinaos-cache/ChinaOSProject-cache/firefox-160.0.1` 保存构建缓存、配置、脚本和日志。
- 原生工具链使用 Clang/LLVM 22.1.8、GCC 13.4.0 头文件和 C++ 运行库、正式 ABI v0 Rust SDK `1.98.1-2` 及 cbindgen 0.29.4。C/C++ 与 Rust 均使用 large code model。原生 LLD 使用 `/lib64/ld.so.1`，Wasm 使用独立的 Clang 21 与 WASI SDK 27.0。[Clang 分包报告](lib_clang_v22.1.8_loongarch64.md) · [cbindgen 报告](lib_cbindgen_v0.29.4_loongarch64.md)
- 最初使用的暂存 Rust SDK 生成 ABI v1 对象，旧 ABI LLD 拒绝链接。替换为正式的 ABI v0 SDK 后重试。正式 SDK 包的 SHA256 为 `c875e0565f04b3989d008faf283f8583d200bf3aeb140926e0ec3329f3f0e734`。
- 构建沿用非官方 Nightly 品牌和 en-US 语言，使用 `--disable-updater`，由包管理器提供版本升级。

## Debian 包与桌面集成

- 包名 `firefox`，版本 `160.0.1-1`，架构 `loongarch64`。交付路径为 `../binaries/firefox/loongarch64/firefox_v160.0.1_loongarch64.deb`。
- 参考 Debian Firefox `160.0.1-1` 的 `debian/control` 和 `debian/browser.desktop.in`。本包维持 157.0.1 的 `/usr/lib/firefox` 应用目录与 `/usr/bin/firefox` 入口，保留应用私有的 NSS、NSPR 和 C++ 运行库。运行依赖由 `dpkg-shlibdeps` 根据实际 ELF 计算，构建用的 Clang、Rust 和 cbindgen 独立于浏览器运行依赖。[Debian 打包源码](https://deb.debian.org/debian/pool/main/f/firefox/firefox_160.0.1-1.debian.tar.xz)
- Firefox 通过运行时探测加载支持的系统 FFmpeg，因此将 `libavcodec` 声明为推荐包，目标基座可使用 `libavcodec58`。
- 用 Debian 10 的 `libxcomposite1 1:0.4.4-2.1` 逐项验证浏览器导入的 XComposite 符号，再用该发行包的 `shlibs` 元数据计算依赖。旧基座升级测试使用同一版本的库。
- 参考 `scratch3_3.31.1_loongarch64.deb` 控制归档中的文件清单和安装脚本，安装 `/usr/share/applications/firefox.desktop` 和八种尺寸的 `hicolor` 图标。入口包含 HTTP、HTTPS 和 HTML 类型关联，执行 `/usr/bin/firefox %u`，窗口匹配按 `application.ini` 的 `RemotingName` 设置。桌面数据库和图标缓存由 Debian 的 `desktop-file-utils`、`hicolor-icon-theme` 触发器更新。
- 本次构建生成供独立使用的 NSS、NSPR 和 wasm2c 软件包。`chinaos-` 包使用私有前缀，可与发行版的原包共同安装。Firefox 的独立安装包内保留浏览器私有运行库。[NSS 分包](lib_chinaos-libnss3_v3.132-1+chinaos1_loongarch64.md) · [NSPR 分包](lib_chinaos-libnspr4_v4.40-1+chinaos1_loongarch64.md) · [wasm2c 分包](lib_chinaos-wabt_v0~git20260916.e197d0b7-1+chinaos1_loongarch64.md)

## 内容进程沙箱

Firefox 160.0.1 的 `toolkit/moz.configure` 仍将 LoongArch 排除在 Linux 内容进程沙箱默认启用的架构之外。本次构建的配置未定义 `MOZ_SANDBOX`。

## 构建与运行验证

- `mach build -j3` 和 `mach package` 均成功。正式包解包后的文件通过包内 MD5 校验。
- 检查了包内 23 个 ELF，均为 LoongArch ABI v0、flags `0x3`。动态可执行文件使用 `/lib64/ld.so.1`，最高 GLIBC 符号版本为 `2.28`。
- 在 glibc 2.28 目标宿主从解包入口运行，`--version` 返回 `Mozilla Firefox 160.0.1`。无头浏览器通过 WebAssembly、Canvas 和 Cookie 测试并生成 PNG。
- 在 glibc 2.28 隔离容器中从 `157.0.1-1` 升级，包管理检查和桌面文件校验通过。普通用户使用 `gtk-launch firefox` 打开可见 X11 主窗口。卸载后检查启动项、代表性图标及 MIME 缓存已清理。
- 日志位于构建缓存的 `logs/firefox-build.log`、`logs/firefox-package.log`、`logs/firefox-deb.log`、`logs/firefox-smoke.log` 和 `logs/firefox-desktop.log`。运行与 ELF 明细记录在 `firefox-verify.J3mXwQ/result.json`。

## 交付

- 包大小 81975304 字节，SHA256 `37dc8a36124dbc82261dbbabe5f862948c92654ab32d7a6c422eb220135aa080`。
- Firefox 和 12 个配套包已发布于 `ChinaOSProject/binaries` 提交 `f8b8143175ff02d7c8a4551df43cf37259da2f3b`。从远端重新下载了全部 13 个 Git LFS 对象，SHA256 逐项与本地一致，记录为 `logs/firefox-publish-lfs-check.log`。[Firefox 安装包](https://github.com/ChinaOSProject/binaries/blob/f8b8143175ff02d7c8a4551df43cf37259da2f3b/firefox/loongarch64/firefox_v160.0.1_loongarch64.deb)
- 软件包依赖为 `libasound2 (>= 1.0.16), libatk1.0-0 (>= 1.12.4), libc6 (>= 2.28), libcairo-gobject2 (>= 1.10.0), libcairo2 (>= 1.10.0), libdbus-1-3 (>= 1.9.14), libffi7 (>= 3.3~20180313), libfontconfig1 (>= 2.12.6), libfreetype6 (>= 2.6), libgcc1 (>= 1:8.3.0-6.lnd.vec.36.1), libgdk-pixbuf2.0-0 (>= 2.22.0), libglib2.0-0 (>= 2.39.90), libgtk-3-0 (>= 3.13.7), libpango-1.0-0 (>= 1.14.0), libpangocairo-1.0-0 (>= 1.14.0), libx11-6, libx11-xcb1 (>= 2:1.6.9), libxcb-shm0, libxcb1, libxcomposite1 (>= 1:0.3-1), libxcursor1 (>> 1.1.2), libxdamage1 (>= 1:1.1), libxext6, libxfixes3 (>= 1:5.0), libxi6, libxrandr2 (>= 2:1.4.0), libxrender1`。
