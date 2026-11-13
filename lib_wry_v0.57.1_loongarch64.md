# Wry 0.57.1：兼容 WebKitGTK 2.38–2.40 的全部 Cookie 接口探测

日期：2026-11-12。被适配仓库： https://github.com/tauri-apps/wry 。精确上游提交 `2446feadae623d6f2ac8d2136e3ccef3d7197e0e`，crate `wry 0.57.1`。2026-11-13 已在 `debian10` 创建最小源码提交 `c8bb6e63fd696160609d7795bbb32566e8ed6086`，并将 `loong64` 快进至同一提交；`main` 保留上游基线，工作树干净。尚未推送源码：目标 ChinaOSProject/wry 仓库目前不可访问（GitHub API 404、SSH Repository not found）。本报告和补丁侧文件随二进制独立归档。

## 问题与结果

原版 Linux 后端在 `src/webkitgtk/mod.rs` 中直接声明和调用 `webkit_cookie_manager_get_all_cookies` 与 `webkit_cookie_manager_get_all_cookies_finish`。官方 WebKitGTK 4.1 API 文档标明两者均自 2.42 提供，而 Wry manifest 启用的最低 WebKit feature 为 `v2_38`（本次 Clash v2.5.10 消费图启用 `v2_40`）。实际私有 WebKitGTK 2.40.5 没有导出这两个符号，最终链接因此失败。

补丁只修改该文件：使用 GLib 已有的 GModule 设施，从已加载主程序及其链接模块查找这对符号。只有两者均存在且非空才调用。缺少全部或一部分符号时，在当前拥有的 MainContext 上异步完成 callback，返回 `gio::IOErrorEnum::NotSupported`；不把失败当成功，不返回伪造空 cookies，不用 domain/URI 枚举近似全部 cookies。现代接口仍调用原始 start/finish 流程，原始 `GError` 和完整 `SoupCookie` 列表按原来的 transfer-full 规则处理。

没有修改 Wry Cargo.toml、应用源码或 Cargo.lock，没有依赖降级，没有修改 registry 缓存。没有运行、停止、重启或向 Magpie 发信号，也没有执行 Magpie 的版本入口。

## ABI、生命周期与兼容范围

- 两个函数指针采用官方 C 声明的精确参数、返回值和 `unsafe extern "C"` ABI；解析后的 pointer-to-function 转换有对应的 ABI safety 注释。
- GModule 主程序句柄在异步 userdata 中与 ThreadGuard callback 一起保存，直到 finish、转换与 callback 结束才 close；部分解析失败立即 close。不缓存全局或跨 Session 的能力结果，不存在只解析一个符号后空指针调用的路径。
- 缺少能力时使用 `MainContext::spawn_local` 保持异步和所属线程，避免同步发回结果后调用者进入阻塞式 GTK iteration 的问题。失败分支不提前分配 native callback userdata。
- 保留上游 MainContext ownership 检查以及 ThreadGuard 的线程约束。
- 新增直接 native 依赖 `libgmodule-2.0.so.0`；用到的 `g_module_open`、`g_module_symbol` 和 `g_module_close` 均为历史 GLib API，没有引入新的 GLib 2.70 API floor。未新增 Cargo 依赖。
- 本次复用的 `clash-libglib2.0-0 2.70.5-1loong1:loongarch64` 已包含 `libgmodule-2.0.so.0.7000.5` 和 SONAME 链接；最终应用依赖仍须由主 Agent 根据实际 ELF 声明到这一运行库包，开发链接属于既有 GLib SDK。

## 验证

全部编译与运行验证均在已有专用 Podman 容器 `codex-v0.166.0-v8-directory-probe-20261107` 内，持宿主共享锁 `/tmp/chinaos-heavy-build-20261105.lock`，使用已验证的 Rust 1.99.1-dev consumer snapshot 和 app 已编译的 GTK/WebKit rlibs。没有宿主编译或新建容器，构建阶段没有网络操作。独立测试程序从生产文件逐字提取整个 `mod ffi`，未复制或改变算法；基线也同样从精确上游文件提取。

| 检查 | 结果 |
| --- | --- |
| 真实私有 WebKitGTK 2.40.5 动态符号核验 | 两个全部-cookie API 均不存在 |
| 精确原版 FFI 与该库链接 | 失败，明确缺少上述两个符号 |
| 修复版与真实 2.40.5 运行 | 异步 NotSupported；同线程、一次完成、callback capture 一次销毁 |
| 受控部分 ABI（仅 start 符号） | 异步 NotSupported，部分 start 未被调用 |
| 受控完整现代 ABI 成功 | 两条真实 SoupCookie，经 finish 转换后 domain/value/path 完整；secure 和非根路径保留 |
| 受控完整现代 ABI 错误 | 原始 G_IO_ERROR_PERMISSION_DENIED 正确传播 |
| 修复版独立测试 ELF | LoongArch，flags `0x3`，解释器 `/lib64/ld.so.1`，最高 GLIBC_2.28 |

受控现代 ABI 测试通过 C 函数构造 GTask，并用真实 GLib/GIO/libsoup 分配返回对象、调度 async callback 和传播 GError。它验证解析、调用签名、finish、容器/对象所有权与错误路径，**不是**真实 WebKitGTK 2.42+ 的集成测试；本机未安装该现代库。未启动 GTK 窗口，没有修改宿主显示会话或代理配置。全量 Wry GUI/examples/dev-dependency suite 未运行；Clash 最终集成、链接、GUI/安装验证由主 Agent 负责。

测试环境首次尝试中的夹具问题如实保留：generic CookieManager 缺少私有初始化导致析构失败；Rust 构造封装要求 GTK 初始化；随后正式 WebsiteDataManager C 构造路径需要编译时固定的 WebKitNetworkProcess 安装前缀。最终使用正式 ephemeral data manager，并仅在已有构建容器中临时建立 runtime 指向 SDK 的链接，EXIT trap 已移除，最终四组运行均 exit 0。`WEBKIT_EXEC_PATH` 在此 non-developer WebKit 构建中不生效，已根据原版 `ProcessExecutablePathGLib.cpp` 确认其受 ENABLE(DEVELOPER_MODE) 约束；未更改该源码或关闭沙箱策略。

## Debian/Ubuntu 分包审查

2026-11-12 查询 Debian、Ubuntu 官方全 suite、全 section、全 architecture 名称索引，精确 `librust-wry-dev` 搜索均明确返回“Sorry, your search gave no results”。Debian Sources 的 `rust-wry` 精确 API 返回 `{"error":404}`。因此当前没有确认到常见发行仓库里的 Wry 独立 source-dev 对应包；本次 Wry 仅作为 Rust 静态消费库生成内部中间 rlib，未生成独立 CLI、安装型共享库或 SDK。按 ChinaOSProject AGENTS.md 第6节无对应独立发行包例外，不人为创建无实际消费用途的 `.deb`。GModule 是复用已打包 GLib family 的原有运行库，不重复打包。若之后发行仓库提供 Wry source-dev 包，须重新按其真实 feature/闭包分包审查，不能以 Rust 静态消费一概豁免。

官方查询与原始响应（含最终 URL、HTTP 状态、响应 SHA256）保存在下述 evidence 目录 `references/manifest.json`，成功响应原文同步保留为 `.raw` 和纯文本副本。

## 交付和复现证据

- 正式补丁：`wry-0.57.1-optional-all-cookies.patch`
- 补丁 SHA256：`b87a9756f14cc949fdea2ac09d4352bcc6e54454c767e58e0df0b62535ef6fc0`
- 完整生产文件快照 SHA256：`c4ac316bb83a7ef5756951c6aae61979d28129a5770c5ff0d2e083c0d6b9d135`
- evidence 目录：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/wry-compat-20261112`
- `source-production/mod.rs`：主 Agent 应用集成快照；生产修改已冻结。
- `logs/baseline-link.log`、`logs/patched-build.log`、`logs/runtime-validation.log`：基线、编译和最终运行证据。
- `logs/actual-webkit-symbols.txt`：真实旧库符号表。
- `tests/run-tests-container.sh`、`tests/fixture.c`、`tests/{baseline,patched}.rs`：仓库外独立验证输入；不进入依赖源码 commit。
- `production-manifest.json`、`HANDOFF.md`：集成入口、范围及限制。

`git diff --check` 通过。Wry 当前仅 `src/webkitgtk/mod.rs` 有生产差异（91 additions / 19 deletions），无临时脚本、日志、构建产物或绝对路径进入源码 diff；报告和补丁按要求位于源码仓库外。

## 官方参考

- https://webkitgtk.org/reference/webkit2gtk/stable/method.CookieManager.get_all_cookies.html
- https://webkitgtk.org/reference/webkit2gtk/stable/method.CookieManager.get_all_cookies_finish.html
- https://docs.gtk.org/gmodule/type_func.Module.open.html
- https://docs.gtk.org/gmodule/method.Module.symbol.html
- https://docs.gtk.org/gmodule/method.Module.close.html
- https://packages.debian.org/search?keywords=librust-wry-dev&searchon=names&suite=all&section=all
- https://packages.ubuntu.com/search?keywords=librust-wry-dev&searchon=names&suite=all&section=all
- https://sources.debian.org/api/src/rust-wry/
