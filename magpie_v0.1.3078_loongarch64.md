# Magpie v0.1.3078：LoongArch oldworld 原生桌面版

仓库：https://github.com/yetone/magpie

2026-11-11，精确稳定版本 v0.1.3078，commit `ef176e21f6c40e5a623709bb0e369647173964de`。2026-11-11 重新请求官方 `https://usemagpie.ai/api/latest?channel=stable` 返回 HTTP 200，正文 `channel=stable`、`version=0.1.3078`（published `2026-11-05T23:45:23Z`），与已构建、已安装版本一致。2026-11-10 曾返回 403，原失败证据保留。最新原始响应位于安装任务 `logs/magpie-stable-feed-20261111.{headers,body}`，正文 SHA256 `1e190cc60050071b457cf72340cdb6b3f0579234b81a631122c4c818204390d7`。main/debian10/loong64 在上述提交，Magpie 工作树保持干净。当前安装修订为 `0.1.3078-1loong2`；相对于初版新增的 Wails 依赖修复与 RUNPATH 见后文，不能再将当前修订描述为所有依赖均未修改。

## 构建

在既有 Podman 容器 `codex-v0.166.0-v8-directory-probe-20261107`（glibc 2.28）中构建，未创建容器，未宿主编译。使用已验证 Go 1.27.1 oldworld SDK，`production,gtk3`、CGO_ENABLED=1、GOMAXPROCS=2、`go build -p=2 -trimpath -buildvcs=false`。全程持宿主共享 flock。GTK3 发行包仅下载并提取到外盘私有 sysroot；Clash 的 WebKitGTK 2.40.5/GLib 2.70.5/libsoup 3.6.5 SDK 按校验清单只读复制，私有 .pc 文件重定位。原始 .pc 备份保留；处理了缺 prefix、变量引用次序和原备份已含旧前缀导致的重复重定位问题。源码与模块缓存未 patch。

构建命令退出 0，只有具名 GTimeVal 弃用告警。原生二进制 77,113,912 字节，SHA256 `93ae4ea6fe7b6cc5aba39fce8ade4405072994394939233b000273927d4fd926`。

## 初版 Debian 包与历史验证

`binaries/magpie/loongarch64/magpie_v0.1.3078-1loong1_loongarch64.deb`

- Package: magpie；Version: 0.1.3078-1loong1；Architecture: loongarch64。
- 大小 22,417,800 字节；SHA256 `75700c0402ccb0de78f68acedd7dacff3f07409d9207d961ce7070beab7de9b4`。
- 同一原生二进制提供 GUI、CLI 和嵌入 Web；没有导出的 SONAME/SDK/开发文件。Web 资源编译进二进制，无可独立安装的数据目录；因此不空造 -dev/-data 包。
- 运行依赖逐项依据 DT_NEEDED 声明：系统 GTK3/Wayland/Pango/ATK/Cairo/GdkPixbuf/fontconfig/freetype/X11、libc，以及既有 clash-* 私有 GLib/libsoup/JavaScriptCore/WebKit 包；HTTPS 系统证书依赖 ca-certificates。可选 Git/进程集成使用 Recommends git/procps。
- 发行惯例参考 Debian Go 应用 Hugo 与 GTK/WebKit 应用 Liferea。已读取 Loongnix Buster 衍生仓库 Liferea 1.12.6 control；现代 WebKit 4.1 依赖参考 https://packages.debian.org/en/trixie/liferea 。区别在于 Magpie 的资源嵌入二进制、无 Python/GIR 插件需求；宿主没有发行版 WebKit 4.1，故使用已拆分的私有运行库家族，未替换系统 GLib。
- `/usr/bin/magpie` 是仅为本进程设置私有 LD_LIBRARY_PATH 的 launcher；原生 ELF 位于 `/usr/lib/magpie/magpie`。菜单入口 `/usr/share/applications/magpie.desktop` 与 SVG 图标、上游 LICENSE 和第三方声明均随包安装。没有 maintainer scripts。

## 初版宿主验证（用户禁止重启之前完成）

- apt 模拟只新增 magpie，0 升级/0 移除；实际安装成功。dpkg --verify、包状态、`magpie --version`（magpie v0.1.3078）和 apt 模拟 check 通过。
- ELF LoongArch LP64 flags 0x3，loader `/lib64/ld.so.1`，最高 GLIBC_2.27（低于目标 2.28），宿主动态依赖全部解析。构建目录路径未写入运行时 RPATH。
- 真正启动已安装 `/usr/bin/magpie app`，使用宿主 Xvfb、现有桌面 D-Bus 与临时 XDG 应用目录。X11 原生窗口 class Magpie 出现，WebKit 页面实际显示 Get started、Agents、Providers 与 Codex 0.166.0；截图已人工查看。13 秒后进程存活，SIGTERM 正常退出码 0。
- 后续以普通用户真实桌面 DISPLAY=:0 启动已安装应用，观察到 PID 1793597 对应原生窗口（X11 ID 0x08200003）；窗口保留供用户使用。日志与探测结果为安装根 `logs/magpie-real-desktop-20261111.{json,log}`。
- 第一次使用隔离 dbus-run-session 的 fixture 因 UKUI portal 激活超时，35 秒内没有窗口，不计为通过；失败日志保留。第二次使用现有桌面 bus 成功。日志仍有 Gdk window-state assertion 警告，不宣称无告警或全功能 GUI 回归全绿。
- 之前 stable nogui 的 66 pass/0 fail/1 skip 证据由原 worker 保留，本次未重跑；原生 UI 是独立新验证，未用 CLI/Web 替代 GUI。

## 证据与当前边界

任务根：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/magpie-stable-v0.1.3078/`

- `scripts/main-gui-env-20261111.sh`、外盘 Magpie `scripts/main-build-gui-20261111.sh`：隔离环境与精确构建命令。
- `logs/gui-build-main-20261111T001842Z-1739559.log`：成功构建及告警。
- `logs/gui-host-native-prepackage-20261111.json`：宿主 ELF、版本和动态依赖。
- `gui-packages/package-manifest.json`：候选包与二进制哈希。
- `/mnt/chinaos-cache/ChinaOSProject-cache/local-install-20261110/magpie-gui-smoke-20261111-2/{result.json,window.png,window-tree.txt,magpie.log}`：原生 UI 与正常退出。
- 同一安装根的 `installation-manifest.json`：本轮全部应用安装状态。

本轮仅完成本机交付；尚未将本次 GUI 包/报告暂存、commit 或 push。新编译产物仅 Magpie 应用；Go SDK、Clash 原生库家族等均复用既有已分包产物，发行版 GTK/X11 依赖不重复重打包。

## 2026-11-11 修复修订：0.1.3078-1loong2

用户明确禁止修复期间启动、停止或重启 Magpie，因现有会话由其接管。本次未执行 Magpie，包括版本检查、`app`、`gtk-launch` 或 GUI 启动探测；未向现有进程发信号。

### 快捷方式与运行库查找

用户的 `~/.local/share/applications/magpie.desktop` 覆盖系统入口，且与桌面快捷方式一样使用 `Exec=/usr/lib/magpie/magpie %u`。它绕过 `/usr/bin/magpie` 包装脚本，缺少私有 WebKitGTK/libsoup/JavaScriptCore 搜索路径。两份用户入口仅将 Exec 改为 `/usr/bin/magpie %u`，分别保留 0644/0700 权限，`desktop-file-validate` 通过（原 Categories 的提示保留）。

上游自注册仍使用 `os.Executable()`，将来可能再次写入原始 ELF 路径。因此修订包通过编译链接为 ELF 添加 `/usr/lib/clash-verge/runtime/lib` 的 RUNPATH，使该路径也能自行找到私有运行库；没有修改 Magpie 的自注册源码。清除 `LD_LIBRARY_PATH` 后的依赖检查全部解析，四项关键 WebKitGTK/libsoup/JavaScriptCore/GLib 均来自私有运行库。未改变系统全局加载配置或公共 GLib。

### GDK 状态查询

独立 Wails 仓库锁定 `v3.0.0-beta.24`，commit `f5457ed12f5c53883c4820437d106d48f4340f1e`。官方归档补齐被忽略规则漏加的已跟踪文件后，完整树哈希与原始提交哈希均通过核验。独立依赖 Agent 在 `linux_cgo_gtk3.go` 的三个状态查询中添加 18 行保护：GTK widget 尚未创建或没有原生 GDK window 时返回 false，原状态位表达式不变。

使用仓库外独立源码快照与临时 `-modfile` 替换依赖构建，未修改 Magpie 源码、锁文件或 Go 模块缓存源码。测试在既有 Podman 容器内运行；没有创建容器或宿主编译。新增回归测试在无显示的测试子进程中将 GTK critical 设为致命：原版会因 `gtk_widget_get_window` 的 `GTK_IS_WIDGET` 断言失败，补丁后的该测试和既有状态事件测试通过，包耗时 0.071s。首轮缺少 `matryer/is` 的测试环境失败单独保留；补齐依赖后按官方 Wails go.sum 校验通过。未测真实未实现窗口和窗口管理器状态切换；不宣称新修订已完成真实 GUI 回归。

独立依赖报告及补丁见 `lib_wails_v3.0.0-beta.24_loongarch64.md` 和对应 patch sidecar。生产源码修改仍未提交。

### 当前包与安装

- 正式包：`binaries/magpie/loongarch64/magpie_v0.1.3078-1loong2_loongarch64.deb`。
- Version `0.1.3078-1loong2`，Architecture `loongarch64`，22,418,572 字节。
- SHA256：`a4ce6cd81c6f7734b8af515da2d4581683fd307b7401808716e88cda49eb7691`。
- 包内 ELF SHA256：`06f6ab247f140fc99f4ef1a831330048cc5ea0d1dafad5780c7a47e2f7042e79`；flags `0x3`、解释器 `/lib64/ld.so.1`、最高 GLIBC `2.27`。
- 解包的全部 8 项 md5 通过，包内 ELF 与已验候选相同；依赖声明保持原版。新旧包均没有维护脚本。
- 本机 `dpkg` 安装成功；只读元数据确认当前为 `magpie 0.1.3078-1loong2 loongarch64 install ok installed`。没有运行版本命令。
- 安装前后 PID 2874900、2880849 的启动时间、可执行映射 inode 和哈希均保持不变。旧映射显示为 `(deleted)` 是包文件被替换后保留原映射的结果，不表示进程退出。现有会话继续运行旧映射，修复代码只会在用户以后正常重新开启应用时使用；没有热替换现有进程的代码。

本轮修复证据位于任务根 `repair-20261111/`：`baseline-test.log`、`patched-test.log`、`build.log`、`static-binary-verification.json`、`package-verification.json`、`host-install.log`、`host-install-verification.json`。快捷方式原件和修改记录位于安装根 `magpie-launch-repair-20261111/`。所有已知验证边界和禁止重启约束已写入安装 manifest。

2026-11-12 stable 通道追加只读核验：官方 latest API 返回 HTTP200、channel=stable、version=0.1.3078，与安装版一致。原始响应保存在安装任务 logs/magpie-stable-feed-20261112.{headers,body}，正文 SHA256 `1517acfd0976b330feb58a78f144099266a7269c554fbdd040f1250acd5cfdbc`。本次没有运行、启动、停止、重启或向 Magpie 发信号；此前 GUI 验证范围不变。
