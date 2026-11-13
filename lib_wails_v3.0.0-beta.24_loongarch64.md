# Wails v3.0.0-beta.24 GTK3 窗口状态查询修复报告

适配日期：2026-11-11。目标：Linux/LoongArch64，现有旧 ABI GTK3 / WebKit2GTK 4.1 构建。依赖源码修复、定向回归和应用重新链接已完成；应用包安装经过静态与安装结果核验。遵照用户要求，没有启动、停止、重启、执行或发送信号给 Magpie，也没有执行 `gtk-launch`。当前活跃会话没有用于 GUI 验证。

## 来源、基线和源码范围

- 被适配仓库官方 GitHub 地址：https://github.com/wailsapp/wails
- 官方版本：`v3.0.0-beta.24`。
- 精确提交：`f5457ed12f5c53883c4820437d106d48f4340f1e`。
- 精确 tree：`4153fc2dcf85080d4520af6098f5d8820a313a4c`。
- Go module：`github.com/wailsapp/wails/v3`。
- 官方源码经 codeload archive 与 API 恢复，已核对官方完整 tree 和原始 commit hash；基线恢复记录为下文证据目录 A 的 `wails-source-recovery.json`。
- `main`、`debian10`、`loong64` 的提交基线相同。本次修改保留在 `debian10` 工作区，未 stage、commit、合并或 push；没有假定已经传播到其他分支。

改动仅包含两个文件：

| 文件 | 改动 |
| --- | --- |
| `v3/pkg/application/linux_cgo_gtk3.go` | 增加 18 行，为三个状态查询分别增加 GTK widget 和 GDK window 空值保护 |
| `v3/pkg/application/linux_window_state_gtk3_test.go` | 新增 42 行 GTK3 专用定向回归测试 |

源码、临时构建环境和报告分离：报告及补丁保存在 `DEPENDENCY_CHANGES/`；构建使用证据目录 B 中单独的 `wails-v3` 快照。没有把宿主绝对路径、构建脚本、环境配置或构建产物加入 Wails 源码。

## 问题和最小修复

`WebviewWindow.Run` 在 GTK widget 创建前就设置 `w.impl`；真正的原生窗口稍后在 `InvokeSync(w.impl.run)` 的 `windowNew` 中创建。这个间隙中，公开查询可以观察到非空实现，但 `w.window` 仍为空。原版 `isFullscreen`、`isMaximised`、`isMinimised` 均直接把该空指针传给 `gtk_widget_get_window`，随后无条件调用 `gdk_window_get_state`。同文件既有 `isVisible` 已对相同的延迟创建间隙作过保护。

三个查询现在都先检查 `w.window == nil`，为空直接返回 `false`；取得 GDK window 后再检查其是否为空，为空同样返回 `false`。第二层保护处理 widget 已存在但没有已实现 GDK window 的情况。查询没有强制创建或 realize 窗口，没有增加缓存、锁或销毁路径修改。

原有状态语义保持不变：fullscreen 检查 `GDK_WINDOW_STATE_FULLSCREEN`；minimised 检查 `GDK_WINDOW_STATE_ICONIFIED`；maximised 仍同时要求 `MAXIMIZED` 置位且 `FULLSCREEN` 未置位。没有把 fullscreen 同时视作 maximised。

公开状态查询原先已有 `isDestroyed` 检查，并通过 `InvokeSyncWithResult` 在 GTK 线程中执行查询。新增保护沿用这一线程契约，没有对跨线程 GTK 访问或整个窗口销毁生命周期作额外同步保证。`windowNew` 的现有 `g_object_ref_sink` 及 `destroy`/`close` 的所有权处理均未改动。

修改前执行了只读 GitHub issue/PR 检索。相关上游背景为 [PR #5898](https://github.com/wailsapp/wails/pull/5898) 的延迟创建 visibility 修复，以及 [issue #5966](https://github.com/wailsapp/wails/issues/5966) 的 GTK3 无活动窗口时 screen discovery 问题；两者不是本次三个状态查询的完整替代。具体检索与返回项保存在证据目录 A 的 `wails-upstream-window-state-search.json`。没有创建或修改 issue、评论或 PR。

## 隔离构建和回归证据

所有编译、链接与测试由主 Agent 在既有 Podman 构建容器中进行，复用原有旧 ABI sysroot、SDK 和依赖。没有宿主编译、宿主构建依赖安装、新建容器、全局镜像策略修改或安全策略削弱。

定向命令在容器中 Wails `v3` module 的既有构建环境内执行，对应选择器为：

```sh
go test -tags gtk3 ./pkg/application \
  -run '^(TestGTK3WindowStateBeforeCreation|TestChangedLinuxWindowStateEvents)$' \
  -count=1
```

新测试在测试可执行文件的子进程中设置 `G_DEBUG=fatal-criticals`，直接调用一个尚未创建 GTK widget 的 `linuxWebviewWindow` 的三个查询。这样可以区分“正确返回 false”和“原生库发出 critical 后碰巧返回 false”。子进程只执行 Go package test binary，不执行 Magpie，也不创建图形窗口或启动 Wails 应用事件循环。

| 核验 | 实际结果与证据 |
| --- | --- |
| 初次离线测试准备 | 因缺少既有上游测试依赖 `github.com/matryer/is v1.4.1`，在 `GOPROXY=off` 下 setup failed；保留 `baseline-setup-failure.log`，不把它当作缺陷复现 |
| 上游测试依赖补齐 | 普通 module 下载经官方 Wails `go.sum` 的 module 和 go.mod 校验和核对；见 `matryer-input-fetch.json` 与 `matryer-verified-download.json` |
| 原版源码 + 新回归测试 | `baseline-test.log` 中 `TestGTK3WindowStateBeforeCreation` 失败，包结果 `FAIL ... 1.137s`；子进程在 `gtk_widget_get_window(NULL)` 触发 `GTK_IS_WIDGET (widget)` critical / SIGTRAP |
| 修复后两个选定测试 | `patched-test.log`：`ok github.com/wailsapp/wails/v3/pkg/application 0.071s` |
| 容器 gofmt | `gofmt.diff` 为 0 字节 |
| Magpie 重新链接 | `build.log` 结尾 `build=complete`、`exit_code=0` |
| 补丁复现核验 | 在临时目录中放入精确基线文件，`git apply --check`、实际 apply、反向 check 均通过；应用后两个文件与源码及构建快照逐字节相同 |

校验过的 `matryer/is` module sum 为 `h1:55ehd8zaGABKLXQUe2awZ99BD/PTc2ls+KV/dXphgEQ=`，go.mod sum 为 `h1:8I/i5uYgLzgsgEloJE1U6xx5HkBQpAZvepWuujKwMRU=`。这是补齐原本已有的测试输入，没有修改 module-cache 中的依赖源码，也没有更改 Wails 的 `go.mod` / `go.sum`。编译日志包含既有 GTK 头文件的 `GTimeVal` 弃用警告，构建最终成功退出。

范围限制必须保留：原版实测首先在 GTK widget 断言处终止，**没有执行到 GDK state 调用**；修复后的 nil-widget 路径及既有状态事件测试通过。没有新建真实 GTK 窗口来覆盖“非空 widget / 未 realize 的 GDK window”，没有销毁窗口集成测试，也没有执行 Magpie 的真实 fullscreen/maximise/minimise 窗口管理器转换。没有运行整个 Wails 测试集或声称全部 GUI 场景通过。

## 应用集成和 ABI

本次 Wails 修复是依赖层源码补丁。Magpie 本体固定在 `ef176e21f6c40e5a623709bb0e369647173964de`；主 Agent 使用独立的 `magpie.mod` / `magpie.sum` 和 `replace github.com/wailsapp/wails/v3 => .../repair-20261111/wails-v3` 构建，应用源码及原锁文件保持干净。普通 module cache 中的 Wails 源码没有打补丁。`source-provenance.json` 中的生产补丁 SHA256 与本报告的 18 行生产 diff 一致。

生产修改只增加 Go 分支判断，没有增加 C 函数、native library、头文件依赖、导出接口或 ABI 要求；GTK3 / WebKit2GTK 4.1 的原有依赖关系不变。最终应用二进制的静态核验结果如下，见 `static-binary-verification.json`：

| 项目 | 结果 |
| --- | --- |
| 目标 | ELF64 LoongArch，flags `0x3` |
| loader | `/lib64/ld.so.1` |
| 最高直接 GLIBC 符号需求 | `GLIBC_2.27` |
| RUNPATH | `/usr/lib/clash-verge/runtime/lib` |
| 移除 `LD_LIBRARY_PATH` 后依赖解析 | 全部解析成功 |
| 二进制 SHA256 | `06f6ab247f140fc99f4ef1a831330048cc5ea0d1dafad5780c7a47e2f7042e79` |
| 执行应用 | 未执行 |

这里的 GLIBC 数值描述已检查应用 ELF 的符号版本要求，不能代替所有传递依赖在任意系统上的运行验证。现有宿主依赖解析通过，不代表本报告执行过 live GUI 启动。

## Debian/Ubuntu 分包审查

按根 `AGENTS.md` 第 6 节，先查找真实的发行包对应关系，没有因为 Go 依赖被链接进应用就直接豁免。2026-11-11 官方检索结果如下：

| 官方入口 | 查询范围与结果 |
| --- | --- |
| [Debian binary names](https://packages.debian.org/search?keywords=wails&searchon=names&suite=all&section=all) | 所有已索引 suite、section、architecture，名字包含 `wails`：无结果 |
| [Debian source names](https://packages.debian.org/search?keywords=wails&searchon=sourcenames&suite=all&section=all) | 同范围 source 名字：无结果 |
| [Debian names/descriptions](https://packages.debian.org/search?keywords=wails&searchon=all&suite=all&section=all) | subword matching：无结果 |
| [Debian Sources API](https://sources.debian.org/api/search/wails/) | `exact: null`，`other: []` |
| [Ubuntu binary names](https://packages.ubuntu.com/search?keywords=wails&searchon=names&suite=all&section=all) | 所有已索引 suite、section、architecture，名字包含 `wails`：无结果 |
| [Ubuntu source names](https://packages.ubuntu.com/search?keywords=wails&searchon=sourcenames&suite=all&section=all) | 同范围 source 名字：无结果 |
| [Ubuntu names/descriptions](https://packages.ubuntu.com/search?keywords=wails&searchon=all&suite=all&section=all) | subword matching：无结果 |
| [Debian sid contents](https://packages.debian.org/search?searchon=contents&keywords=wailsapp&mode=filename&suite=sid&arch=any) / [Ubuntu noble contents](https://packages.ubuntu.com/search?searchon=contents&keywords=wailsapp&mode=filename&suite=noble&arch=any) | filenames 含 `wailsapp` 的命名空间交叉核查：均无结果 |

原始官方响应、请求 URL、时间和 SHA256 保存在证据目录 A 的 `wails-packaging-audit/`，其 `index.json`、`supplemental-index.json`、`contents-index.json` 可复核。额外尝试的两项 Launchpad primary archive API 请求超时，已保留失败信息，没有把超时当作“无包”证据。

未找到可对应 Wails 的真实 Debian/Ubuntu 独立 source-development 包，因而没有相应的 `debian/control` 或 `.install` 可直接沿用。本次只把 Wails 作为应用构建依赖，没有单独产出 Wails CLI、可安装 native library、headers 或 SDK；依照第 6 节“没有常见 Debian/Ubuntu 独立发行包对应”的内部中间产物例外，不制造虚构的 Wails `-dev` 包。该结论不豁免其他有真实发行包对应的 Go 库或工具。若以后实际生成独立可复用 Wails CLI/SDK，须重新按相应条款评估。

## 交付、安装和活动会话

依赖层交付是同目录的 [wails-3.0.0-beta.24-gtk3-window-state.patch](./wails-3.0.0-beta.24-gtk3-window-state.patch)，包含生产改动与未跟踪回归测试，不含本机路径或构建环境。

- 完整补丁：2,741 字节；SHA256 `e6711d9422ad6eb4b5be2bb3c352967f656b30ead01cfd25da77513b456a0325`。
- 仅 18 行生产 diff 的 SHA256：`b4652da4d34f8cc8eb4fe22bfe65d253a94336a58056084e7515d63f65df5218`。
- 精确基线的完整 Wails checkout 中，可先运行 `git apply --check /path/to/wails-3.0.0-beta.24-gtk3-window-state.patch`，再 `git apply`；补丁路径以仓库根目录的 `v3/` 开始。

主 Agent 构建并安装的应用产物为 [magpie_v0.1.3078-1loong2_loongarch64.deb](../binaries/magpie/loongarch64/magpie_v0.1.3078-1loong2_loongarch64.deb)，包名 `magpie`、版本 `0.1.3078-1loong2`、架构 `loongarch64`，大小 22,418,572 字节，SHA256 `a4ce6cd81c6f7734b8af515da2d4581683fd307b7401808716e88cda49eb7691`。本报告独立重算规范交付路径中的文件 hash，与 `package-manifest.json` 一致。包内 MD5 全部通过，安装二进制与已核验候选一致，包没有 maintainer scripts。

`host-install-verification.json` 记录 `magpie 0.1.3078-1loong2 loongarch64 install ok installed`。安装时两个既有进程 PID `2874900`、`2880849` 的 starttime、原 executable inode 和原 executable hash 均保持不变；其旧映像仍映射为 `/usr/lib/magpie/magpie (deleted)`。没有执行已安装的 Magpie，也没有重启这些进程。因此修复代码只会在用户以后自行启动的新进程中生效，不能把包安装成功描述为当前活动窗口已运行新代码。

## 证据位置和提交状态

证据目录 A：

```text
/mnt/chinaos-cache/ChinaOSProject-cache/local-install-20261110/magpie-launch-repair-20261111/
```

证据目录 B：

```text
/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/magpie-stable-v0.1.3078/repair-20261111/
```

目录 A 保存基线恢复、上游检索、分包审查和 `wails-agent-handoff.md`。目录 B 保存本报告列出的原版/修复测试、准备失败、校验后 module 下载、格式检查、应用构建、静态 ELF、包校验和宿主安装证据。该目录中实际用于构建的生产文件与新测试均已和本地工作区逐字节比对一致。

源码修改未 stage、commit 或 push。报告及 patch 位于 Wails 仓库外。本报告没有修改 Magpie 的应用适配报告，也没有执行发布操作。


## 2026-11-13 源码提交状态

本轮重新核对最小 diff、已有失败/成功回归、构建输入、补丁及最终 Magpie 包的字节对应关系，没有重复运行成功测试。Wails 源码仍未提交或推送：仓库 AGENTS.md 要求提交前运行 `coderabbit --plain`，当前环境没有该命令，也没有本次 diff 已完成该审查的记录。人工发布授权已收到，但没有将其视为自动完成 CodeRabbit 审查。ChinaOSProject/wails 目标仓库也尚不可访问。报告和已验证二进制归档不表示 Wails 源码提交门禁已经通过。
