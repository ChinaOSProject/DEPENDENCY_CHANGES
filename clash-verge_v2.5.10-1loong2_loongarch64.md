# Clash Verge Rev v2.5.10：LoongArch ABI1 / glibc 2.28 本机交付

## 上游与变更范围

- 仓库：https://github.com/clash-verge-rev/clash-verge-rev
- 精确版本 v2.5.10，提交 `c83f7ad529c7c9cb3d2d54a0955ef8c1b22a411b`。
- 应用源码、Cargo.lock、pnpm-lock.yaml 均未修改；canonical 工作树干净，main/debian10/loong64 先前已按规定传播到此提交。
- 2026-11-13 已将 `debian10`、`loong64` 推送至 https://github.com/ChinaOSProject/clash-verge-rev ，远端提交均核对为 `c83f7ad529c7c9cb3d2d54a0955ef8c1b22a411b`；fork 的默认 `dev` 分支保持原样。依赖适配由各独立源码仓库和补丁归档。
- Cargo.lock SHA256：`0d73d95386162b536ce049c0c26a40e267c021a485ddf17e3c7102f8254e9565`。
- pnpm-lock.yaml SHA256：`b7ae85103846e101e42a62aae69c0ffeae56126d394937e92845176414124546`。
- 两项必要依赖修复均在独立仓库完成：float16 0.1.5 的稳定 LSX 编译能力检测与既有软件 fallback；Wry 0.57.1 对 WebKit2.42 全量 cookie API 的成对可选解析与异步 NotSupported。未修改 registry/module 缓存。
- 构建使用 Cargo paths override；完整 681 节点的包名/版本、依赖边、kind/target 和 features 与原图比较一致，锁未变。原始 Cargo paths warning 保留。

## 编译环境

所有编译/打包在既有 Podman 项目容器 `codex-v0.166.0-v8-directory-probe-20261107` 完成，无新容器、宿主编译或宿主构建依赖安装。活跃对象/日志位于独立 ext4 缓存。编译持共享 flock `/tmp/chinaos-heavy-build-20261105.lock`，最多2 jobs，离线 release。

使用固定 rustc1.99.1-dev / LLVM23.1.1 消费者快照、已验证的私有修复 LLD21.1.8，以及 GTK3.24.5、GLib2.70.5、WebKitGTK2.40.5、libsoup3.6.5 私有 SDK。Rust 对象 flags0x43 与旧 C 对象 flags0x3 经修复链接器自然生成最终 flags0x3；没有修改 ELF ABI 标志的后处理。strip 仅移除发行包调试内容。

前端使用同版原生 pnpm12.16.1、esbuild0.28.2、TSRS0.20.0-ts7.1.0-beta（上游 plain-ptrs 配置）、Rolldown1.2.9 和 LightningCSS1.33.0。原锁冻结安装868个包；完整 `web:build` 的类型检查和 Vite/CSS 构建通过，14569 modules。未重装依赖或降低版本。

最终增量 release 构建 exit0，用时22m59s；日志：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/logs/app-native-build-wry-main-20261112.log`。这不是全量清空缓存构建时间，也不代表整个上游测试套件全部通过。

## Debian 分包与安装布局

沿用真实应用名 `clash-verge`，按 Debian 图形程序与公共数据分包惯例（已保存 Transmission/transmission-common 对照），将原生 GUI、两份有不同发行通道的核心和架构无关地理数据分开。常规应用/数据包依赖分离不把编译器、Cargo、源码-dev 包写为运行依赖。核心和数据的上游许可证及来源分别随各自包交付。

- GUI：`clash-verge 2.5.10-1loong2` / loongarch64，15043488 bytes。
- 稳定核心：`mihomo 1.19.33-1loong1` / loongarch64。
- alpha核心：`mihomo-alpha 1.19.34.alpha.0455d0d-1loong1` / loongarch64。
- 数据：`clash-verge-data 2.5.10-1loong1` / all。
- 服务：复用独立适配的 `clash-verge-service 2.7.7-1`，安装后未启动系统服务。

GUI 包：`binaries/clash-verge/loongarch64/clash-verge_v2.5.10-1loong2_loongarch64.deb`。
SHA256：`31926a6dafc97b6810986ad0ac80faca01ea7e306a687b943c2ebc68d9e8559b`。

真实 ELF `/usr/lib/clash-verge/clash-verge`，`/usr/bin/clash-verge` 是 exec 包装器，不导出 LD_LIBRARY_PATH。菜单入口执行包装器。ELF RUNPATH 指向 `/usr/lib/clash-verge/runtime/lib`；公共系统 GLib 不被升级或替换。

数据在 `/usr/share/clash-verge/resources`。应用的 current_exe sibling sidecar 链接及 `/usr/lib/clash-verge/resources` 保留原版路径解析。真实宿主日志确认 Tauri Linux resource_dir 使用 productName `Clash Verge`，故本修订额外提供 `/usr/lib/Clash Verge -> clash-verge`。原 `1loong1` 候选缺少此 productName 路径，是撤回候选。`1loong2` 只增加该路径，全部常规 payload MD5 与前版一致，未重编主程序。

依赖由现有运行库验证容器对最终 ELF 执行 dpkg-shlibdeps 推导，再补运行时 dlopen 的 libappindicator3-1 和真实核心/数据/服务包：

```text
clash-libgcc-s1 (= 13.4.0-1loong1), clash-libglib2.0-0 (= 2.70.5-1loong1), clash-libjavascriptcoregtk-4.1-0 (= 2.40.5-1loong1), clash-libsoup-3.0-0 (= 3.6.5-1loong1), clash-libwebkit2gtk-4.1-0 (= 2.40.5-1loong1), libc6 (>= 2.28), libcairo2 (>= 1.10.0), libdbus-1-3 (>= 1.9.14), libgdk-pixbuf2.0-0 (>= 2.36.9), libgtk-3-0 (>= 3.21.5), libappindicator3-1, mihomo (= 1.19.33-1loong1), mihomo-alpha (= 1.19.34.alpha.0455d0d-1loong1), clash-verge-data (= 2.5.10-1loong1), clash-verge-service (= 2.7.7-1)
```

私有库包按 Debian GLib/WebKit/libsoup/GStreamer/GCC family 拆分，使用 clash 前缀与私有目录以共存。GLib GModule 是 Wry 新直接使用的库，已在 NEEDED 和 clash-libglib2.0-0 依赖中核实。Wry 没有伪造空 cookie 成功；真实旧 WebKit 的全部/部分缺符号路径及受控现代 ABI 夹具均通过，真实2.42+集成未声称通过。

## ABI、安装和宿主验证

原始与打包 ELF 均为 LoongArch flags0x3、解释器 `/lib64/ld.so.1`、最高 GLIBC2.28。安装后 stripped ELF SHA256：`3ef439c70dd51a30790e989cac442fc941e3cc04c9697689c1e85475591606cb`。

既有 `clash-native-family-runtime-v2510` 容器内完整包组安装、dpkg verify、ldd -r、路径/四份资源核验通过，无未解析动态库或符号。没有 maintainer scripts 启动应用或服务。

宿主 apt 模拟首先仅新增 GUI/两个核心/数据4包，无升级删除；资源修订模拟仅升级同一 GUI 包。正式安装及 dpkg verify 通过，桌面文件通过 desktop-file-validate。宿主未安装构建依赖。

真实宿主 GUI 使用短私有 XDG 目录、独立 D-Bus、关闭代理/TUN/自启/热键的 fresh profile。窗口 PID 与本次 app 匹配；主 Agent 人工检查窗口截图，首页/侧栏/卡片已渲染。窗口出现约22.65秒后获得非空白截图。四份资源均复制成功并与正式数据包哈希一致。独立 Mihomo Unix socket 的带认证 `/version` 返回 HTTP200 与 `{"meta":true,"version":"v1.19.33"}`。本次未配置真实订阅或断言上游代理吞吐。

原始第二轮 smoke 自动结果保留为失败：测试期望版本不带 `v`，实际官方 API 带 `v`。该断言在正确响应之后失败；截图、HTTP200、原始响应字典、资源和清理证据均保留，由独立 `reviewed-acceptance.json` 记录有效核验结论，没有把原始失败改写为通过或重复运行 GUI 来掩盖它。首轮5秒截图空白且资源路径错误也保留。

退出仅清理本次独立会话中的已核实进程；系统代理前后快照 SHA 相同，Magpie 两个受保护 PID/启动时间/executable inode 全程相同。没有执行、启动、停止、重启或发送任何信号给 Magpie。系统 service 保持 inactive。

私有桌面 portal 启动超时与一条 GLib D-Bus critical 留在日志；其后 GUI 与核心正常。测试的 XDG 和 D-Bus 隔离使代理关闭时的 dconf 写入留在测试配置中，未写用户原配置。

## 证据与相关交付

- 主目录：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110`。
- GUI构建/依赖图/ELF：`logs/app-native-build-wry-main-20261112.log`、`wry-compat-20261112/app-graph-comparison.json`、`packaging-20261112/app-unstripped-elf.json`。
- 正式候选：`packaging-20261112/app-revision2/clash-verge-package.json`、`container-resource-verification.log`、`host-install-authenticated.log`。
- 原包组静态验证：`packaging-20261112/app-container-verification.json`。
- 宿主 GUI原始结果、截图与复核：`/mnt/chinaos-cache/ChinaOSProject-cache/local-install-20261110/clash-gui-smoke-20261112-2/`。
- float16/Wry报告：`lib_float16_v0.1.5_loongarch64.md`、`lib_wry_v0.57.1_loongarch64.md`，补丁 sidecar 同目录。
- 前端工具与库分包报告：pnpm、esbuild、TSRS、Rolldown、LightningCSS对应报告。LightningCSS110个 source-dev 包/114 crates 已用 fresh Cargo home、解包源码做 locked offline消费者验证，输出 `a{color:red;margin:0}`；与float16五个源码包共存。它们是构建复用交付，非GUI运行依赖。
- 整体 Rust/LLVM SDK分包及匹配Codex helper仍在独立收尾；本报告只确认本GUI实际安装验收，不将未完成SDK或helper宣称为完成。
