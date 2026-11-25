# Firefox 160.0.1-2 LoongArch64 适配报告

## 源码与构建

- 上游版本为 `FIREFOX_160_0_1_RELEASE`，在本地 `loong64` 分支的 `378eff036ef415dbaae27d48186f52fe0aa49149` 上构建。LoongArch ABI v0 所需的 libwebrtc、链接、XPCOM 和 RLBox 修复见 [160.0.1-1 适配报告](firefox_v160.0.1_loongarch64.md)。
- `mozconfig-official` 启用 `--enable-official-branding`，使用 `browser/branding/official` 的 en-US 字符串和图标。构建保留 `--disable-updater`，由 Debian 包管理器负责升级。
- 使用 rootless Podman 容器 `firefox-mozilla-build-160-official`，复用 Debian 10 LoongArch 构建镜像、Clang 22.1.8、ABI v0 Rust 1.98.1、cbindgen 0.29.4 和对象缓存。初次启动使用 `mach build` 默认的 idle 优先级，中断初次编译后以 `mach build --priority normal -j3` 续编。缓存根目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/firefox-160.0.1`。
- 首次打包时，增量构建按修改时间复用了旧品牌的本地化链接和暂存图标。将旧资源及暂存目录移入缓存备份后，重新运行正式品牌 `content` 和 `locales` 的 `misc` 目标，再执行 Mozilla `stage-package` 生成 Debian 打包所需的完整文件树。
- Firefox 的 Linux 内容进程沙箱配置仍排除 LoongArch，构建未定义 `MOZ_SANDBOX`。

## Debian 包

- 包名 `firefox`，版本 `160.0.1-2`，架构 `loongarch64`。本地路径为 `../binaries/firefox/loongarch64/firefox_v160.0.1-2_loongarch64.deb`。Debian 修订号用于从 `160.0.1-1` 升级。
- 依照 Debian `firefox` 的打包惯例计算运行库依赖。包内保留 `/usr/lib/firefox` 应用目录、`/usr/bin/firefox` 入口和浏览器私有库。运行依赖由 `dpkg-shlibdeps` 根据实际 ELF 计算。[Debian 打包源码](https://deb.debian.org/debian/pool/main/f/firefox/firefox_160.0.1-1.debian.tar.xz)
- 沿用 `scratch3_3.31.1_loongarch64.deb` 的应用菜单集成方式。安装 `firefox.desktop`、HTTP/HTTPS 类型关联和八种尺寸的 `hicolor` 图标。应用菜单通过 `/usr/bin/firefox %u` 启动，`StartupWMClass` 取自 `application.ini` 的 `RemotingName`。
- 重新链接后的 NSS、NSPR、SQLite、NSS 命令行工具和 wasm2c 共 14 个产物通过构建前后 SHA256 比对，继续使用 `160.0.1-1` 交付时已发布的配套包。比对日志为 `logs/firefox-official-components.log`。
- Mozilla 要求带 Firefox 商标的修改版在公开分发前取得书面许可。包在本地交付，待确认许可后公开发布。[Mozilla 软件分发政策](https://www.mozilla.org/en-US/foundation/trademarks/distribution-policy/)

## 验证

- 包内 MD5 校验通过。解包后核对 Firefox 名称、两份品牌本地化字符串、关于页面 SVG 和八种桌面图标。打包器会去除本地化注释，字符串逐条与 Mozilla 源码一致，SVG 和图标逐字节一致。
- 包内 23 个 ELF 均为 LoongArch ABI v0，flags 为 `0x3`。动态可执行文件使用 `/lib64/ld.so.1`，最高 GLIBC 导入符号版本为 `2.28`。`Depends` 与 `160.0.1-1` 一致。
- 在 glibc 2.28 目标宿主从解包入口运行，版本输出 `Mozilla Firefox 160.0.1`。无头浏览器通过 WebAssembly、Canvas 和 Cookie 测试并生成 PNG。
- 在 glibc 2.28 隔离容器中，APT 完成从 `160.0.1-1` 到 `160.0.1-2` 的升级并更新图标。包管理器完整性、依赖和桌面条目检查均通过。普通用户通过 `gtk-launch firefox` 打开本地测试页面，可见 X11 主窗口的 UTF-8 标题包含 `Mozilla Firefox`，`WM_CLASS` 与 `StartupWMClass=firefox-default` 对应。卸载后，菜单入口、八种图标和 MIME 缓存记录均已清理。
- 首次窗口检测在 C locale 下使用 `xwininfo` 读取 UTF-8 标题，无法转换标题中的字符。改用 `xprop` 读取 `_NET_WM_NAME` 后，在已完成升级的同一容器内复测通过。升级与窗口复测分别记录在 `logs/firefox-official-desktop.log` 和 `logs/firefox-official-desktop-window.log`。
- 包大小为 81863276 字节，SHA256 为 `399628ae343ce4f1ebace7f4aa61fb27e6e4254461b3765923c7f723e8c96c86`。
- 原生构建、品牌刷新、暂存、Debian 打包、品牌比对和无头运行日志分别为 `logs/firefox-official-build.log`、`logs/firefox-official-brand-refresh.log`、`logs/firefox-official-stage-package.log`、`logs/firefox-official-deb.log`、`logs/firefox-official-brand.log` 和 `logs/firefox-official-smoke.log`。运行及 ELF 明细位于 `firefox-official-verify.aouiFg/result.json`。
