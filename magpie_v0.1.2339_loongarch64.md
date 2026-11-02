# Magpie v0.1.2339 / loongarch64 适配与交付报告

日期：2026-11-01。当前交付范围为 **CLI + HTTP gateway + Web 浏览器界面**；本报告对应的 Debian 包使用上游原生 `nogui` 标签，**不包含原生 GUI、托盘或 Wails**。用户随后授权的 GUI 尝试不改变本包内容。

## 交付结果

- 源码仓库：<https://github.com/yetone/magpie>
- 锁定 tag：`v0.1.2339`
- 精确提交：`03f609e3fdf7d2b373ecd35fa317c663e410d8e9`
- 软件包：`binaries/magpie/loongarch64/magpie_v0.1.2339_loongarch64.deb`
- Debian Package / Version / Architecture：`magpie` / `0.1.2339-1` / **`loongarch64`**
- 包大小：19,335,352 字节；安装大小字段：62,792 KiB。
- 包 SHA-256：`832f759c82c76229f1b21bf034bb69bd6e00c8d6bb43ee685a2d4cc05b9f5538`
- 程序 SHA-256：`dc90da7bf3e78c455c7a7fd9cb74c940daf73a747639d3080dca6f4c82555d1f`
- 程序大小：63,766,688 字节。

包内 `/usr/bin/magpie` 与已经在真实宿主通过 Web/gateway smoke 的程序逐字节一致。解包后的程序在宿主执行 `--version` 返回 `magpie v0.1.2339`。没有在宿主安装本包，也没有更改用户的真实 Magpie/agent 配置。

## 源码与分支

已遵守上级 `AGENTS.md`、项目 `AGENTS.md`、完整 `LESSONS.md` 和代码标准。`upstream/main → debian10 → loong64` 三个引用均为上述提交，当前分支 `loong64`。**Magpie 源码零修改，依赖源码零修改，工作区干净，因此没有新增源码 commit。** 没有追踪锁定版本之后的发布，没有更改模块缓存或 vendor。

全部脚本、日志、镜像说明、SDK 和工作缓存位于仓库外 `/tmp/magpie-20261031`。只生成本报告和约定的 Magpie 包，没有修改 binaries/报告仓库内其他任务的未跟踪文件。本执行 Agent 没有 push；审核和发布由主 Agent 完成。

## 宿主、构建隔离与工具链

目标宿主是 Kylin V10 SP1，kernel `5.4.18`、glibc `2.28`、LoongArch old-world ABI，系统加载器 `/lib64/ld.so.1`，dpkg 架构为 `loongarch64`。Go 的架构名称为 `loong64`，两者不能混用。

所有编译、测试工具准备和 `.deb` 打包均在项目专用 rootless Podman 容器内完成，没有在宿主安装工具链、构建依赖或改变全局容器配置。专用 VFS storage：

```text
/tmp/magpie-20261031/podman-root
/tmp/magpie-20261031/podman-run
```

调用入口为 `/tmp/magpie-20261031/podman-magpie`。运行容器使用 `--security-opt seccomp=unconfined`，因为本机 Podman 编译时没有 seccomp 支持。旧内核不支持 `podman build` 所用的构建 overlay，因此在专属 `podman run` 容器内准备环境，再使用 `podman commit` 固化专属镜像；没有改变宿主全局设置。

基础镜像为 `cr.loongnix.cn/library/debian:buster-slim`，构建环境 GCC 8.3、glibc 2.28。专属镜像包括：

```text
localhost/magpie-build:v0.1.2339-buster-loongarch64
localhost/magpie-test:v0.1.2339-buster-loongarch64
localhost/magpie-test-final:v0.1.2339-buster-loongarch64
```

`go.mod` 要求 Go `1.26.8`。实际使用可在本机 old-world 上运行的 **Go 1.27.1** 发布工具链，没有修改或重新实现 Go：

- 下载：<https://github.com/loong64-abi1-0/golang/releases/download/go1.27.1/go1.27.1.linux-loong64.tar.gz>
- SHA-256：`e4f8920f59c1da3604fba3ff2d668e7ff607d74b4fd548230149e8c8e513dff8`
- 已与发布 API 的 digest 和发布方独立 `.sha256` 文件核对。
- `GOTOOLCHAIN=local`，避免构建时自动换用工具链。
- Go 1.26.8 的对应 old-world 发布 API 探测返回 404；未因此改动工具链源码。测试记录明确保留 Go 1.27.1 与上游要求版本不同这一边界。

`go mod download`、`go mod verify` 均成功。`moejs v0.1.0-alpha.11`、`modernc.org/sqlite v1.44.3` 及其 Go 依赖无需源码修改即可编译。

## 构建命令与 ABI

在容器内、源码只读挂载到 `/src`、独立缓存挂载到 `/cache`：

```sh
CGO_ENABLED=0 go build -p 2 -tags nogui -trimpath \
  -ldflags="-s -w -X main.version=v0.1.2339" \
  -o /out/magpie-cli .
```

这是上游 `make cli` 使用的构建方式。`nogui` 保留 Web 后端及嵌入页面，排除原生窗口相关源码。编译进程序的模块列表没有 Wails；最终包也不依赖 GTK 或 WebKitGTK。

`file` 与 `readelf` 结果：ELF64 little endian / LoongArch / LP64，flags `0x3`，**静态链接**，没有 `PT_INTERP` 和动态依赖段。因此本程序没有动态 glibc 符号版本 floor，不依赖宿主 `/lib64/ld.so.1` 装载共享库。已在实际 old-world / kernel 5.4.18 宿主执行并完成请求验证；没有据此推断其在其他更旧内核或所有 LoongArch 系统上的兼容性。

证据：`logs/cli-initial-build.log`、`logs/elf-abi.txt`、包内 `BUILD-INFO`。

## 真实宿主 smoke

`host-smoke.py` 在真实宿主运行编译出的静态程序，使用全新 HOME/XDG 目录、随机 loopback 端口与本地 HTTP fixture。只在 smoke 使用 `MAGPIE_NO_ACCOUNTS=1` 和 `DO_NOT_TRACK=1`；没有在最终单元测试环境保留这些覆盖项，没有读取真实 provider 凭据或干扰已有 gateway。

已成功验证：

1. CLI `provider add` 建立本地 fixture provider，发现一个测试模型。
2. `serve --web=127.0.0.1:PORT` 启动 gateway 和浏览器页面，gateway 根路由报告精确版本 `v0.1.2339`。
3. `healthcheck` 返回 0。
4. `/v1/chat/completions` 真实穿过 gateway 到本地 fixture，返回 HTTP 200 和 `pong from local fixture`；核对了上游模型从 `smoke/fixture-model` 改写为 `fixture-model`，以及测试 provider 的 Authorization。
5. 未登录访问 Web API 返回 401；使用 run key 登录取得 cookie 后，嵌入 HTML 返回 200，长度 44,499 字节。
6. `/api/state`、`/api/providers`、`/api/sessions/overview` 均返回 200 和有效 JSON。
7. 正常用 SIGINT 结束独立测试服务，fixture 线程和服务进程已回收。退出状态 `-2` 来自主动作出的 SIGINT。

本轮续接没有重复已完成的 gateway/Web smoke；包校验确认包内程序与上述成功程序相同，并单独执行了解包程序的宿主 `--version`。

证据：`logs/host-smoke.json`、`logs/host-smoke-run.log`、`logs/host-gateway.log`、`logs/package-host-version.log`。

边界：未使用真实付费 provider/订阅账号；未在真实浏览器中执行页面 JavaScript、点击操作或执行 Playwright 双引擎测试；未进行交互 TUI、长期运行或 native GUI 验证。Web 可用结论限于已验证的页面、鉴权、API 与 gateway 请求路径。

## nogui vet 与全套测试

`CGO_ENABLED=0 go vet -p 2 -tags nogui ./...` 成功。最后一轮完整命令：

```sh
go test -p 2 -timeout 45m -tags nogui ./...
```

运行在无外网、非 root UID 1000 的专属测试容器，临时 HOME、独立 Go 缓存，`CGO_ENABLED=0`。仅同时运行一轮完整 suite，日志完整保留，未覆盖旧日志。

最后一轮从 2026-11-01 15:10:17 到 15:25:13（UTC+8），退出码 **1**：**76 个 package 通过，6 个失败，17 个没有测试**；按日志行首 `--- FAIL:` 精确计数，18 个顶层测试失败。`internal/provider`、`internal/sqlitedb`、`internal/plugin`、`internal/pluginrt`、`internal/sessions`、`internal/tui` 等通过。**不能把本次结果描述为全套测试通过。**

### 已处理的环境问题

- 首轮错误地把 smoke 的 `MAGPIE_NO_ACCOUNTS=1` 带入测试，并以 root 运行且缺少 `ps`，导致大量失真的失败；保留 `nogui-suite.log`，不把它当作产品基线。
- 补齐测试容器的 `procps`、`psmisc`，改为非 root 测试，去除生产行为覆盖环境变量。
- Buster Git 2.20 不支持测试使用的 `git init -b main` / `init.defaultBranch`。在专属容器内从未修改源码构建 Git 2.43.0，用于本地 Git fixtures，消除了旧 Git 所致的分支/插件仓库失败。源码 tarball SHA-256 `5446603e73d911781d259e565750dcd277a42836c8e392cac91cf137aa9b76ec`，与 kernel.org 的 `sha256sums.asc` 对照一致。
- 基座 dash 执行 `/bin/sh -n -c 'echo ...'` 仍输出命令结果；在测试容器内把 `/bin/sh` 指向能正确处理 `-n` 的现有 bash，`TestInstallWithoutNodeParses` 最终通过。宿主 shell 未改变。
- Go 1.27 的 `testing.TempDir` 优先使用 `GOTMPDIR`，先前将其固定为 `/tmp`，导致 library 的 TestMain 改变 TMPDIR 后，子进程校验不接受测试创建的 HOME。最终测试将 `GOTMPDIR` 置空，MCP 子进程、hub 等原先的临时目录失败消失。

### 最后一轮仍失败的具体项目

| package | 测试 | 证据与边界 |
| --- | --- | --- |
| agent | `TestInstalledVersion` | 新建 fake pi 的两个版本字符串读为空。 |
| agent | `TestHowInstalled` | fake pnpm 安装方式识别为 nil。 |
| agent | `TestCursorFolderFromTheEditorIsNotTheCLI` | 在 PATH 新建 fake 命令后未识别。 |
| agent | `TestCopilotFolderFromAnIDEPluginIsNotTheCLI` | 同上，fake copilot 未识别。 |
| agent | `TestQoderSettingsFromAnIDEPluginIsNotTheCLI/qoder-cn` | 同上，fake qoderclicn 未识别。 |
| agent | `TestOmpDetectedWithPiDir` | 删除 fake 命令后仍认为 omp 存在。 |
| agent | `TestZedAppPath` | fake CLI 建好后仍返回此前 Preview app 路径。 |
| cli | `TestKeyNote` | 断言期望裸机提示，实际正确进入容器提示分支。 |
| gateway | `TestClaudeClientCacheAbsentUnchanged` | ordinary prompt 的实际输出与断言不符；保留完整 args/told 日志，未判定已修复。 |
| gateway | `TestCodexFittedImagesPastTheBudget` | 后两轮重做 19 张图片，断言期望 20；可能与 Go 编码输出/预算夹具有关，未证实具体原因。 |
| gateway | `TestMarshalRawIsEncodingJSON` | `escapeHTML=false` 时 U+2028/U+2029 字节与 Go 1.27 `encoding/json` 输出不同。仍是有效 JSON，但不能声称逐字节等价。 |
| gateway | `FuzzMarshalRaw/seed#7` | 同一行分隔符编码差异。仅运行 fuzz seeds，未做持续 fuzz。 |
| gateway | `TestAutoVisionDoesNotBuildListsAgain` | searcherModel 51 次 allocation，断言上限 50；AutoVision 长列表增幅断言已通过。 |
| gateway | `TestServeWSLListensBesideLoopback` | 模拟 WSL 网络移除后 IPv6 loopback 仍应答；不是本机 Linux IPv4 smoke 所覆盖的场景。 |
| gui（含 Web 后端） | `TestSettingsSaveKeepsWhatItDoesNotSend` | 上游测试夹具未设置 `SearchAll` 非零值，命中夹具完整性断言。 |
| gui（含 Web 后端） | `TestUpdateNotWritableSaysWhy` | 期望 not-writable，容器内实际优先报告 container；未为迎合断言修改生产源码。 |
| library | `TestGitSignInRefusedSaysSo` | 新编的 fixture Git 未编入 remote-http，无法执行本地 HTTP 鉴权 fixture；见下方定点复测。 |
| proc | `TestNoCommandBypassesProc` | 精确 tag 内 `internal/claudetui/child_unix.go:32` 已直接调用 `exec.Command`，与其源码规范 guard 冲突。未为无关规范测试改动冻结上游。 |

agent 这一组 fake 路径缓存失败有明确环境线索：本机容器的 `/tmp` 和 tmpfs 中，创建目录后立即创建文件的目录 mtime 可完全相同，等待 20ms 后再写才改变；源码的查找缓存依赖目录 mtime。这是与旧内核时间戳分辨率一致的现象，**尚未逐项证明每个失败都由它造成**，也没有据此宣布真实 agent 发现功能全部正常。

`TestGitSignInRefusedSaysSo` 随后在相同隔离、相同 Go 环境中，仅将 PATH 改回带 HTTP transport 的基座 Git，定点复测通过（0.071s）。这消除了该测试的 transport 环境阻塞，但没有把最后一轮完整 suite 的退出码改写为成功。

结果文件：`logs/nogui-suite-final.log`、`logs/nogui-suite-final.exit`、`logs/test-summary.json`、`logs/nogui-final-checks.log`。前一轮完整日志 `logs/nogui-suite-user.log` 也保留。未无限重试剩余测试，没有跳过失败项伪造全绿；已明确把本包交主 Agent 审核。

未执行 Windows/macOS cross build/vet、race（本次 LoongArch 工具链未进行该验证）、Playwright 或真实原生 GUI 测试；本次没有源码行为修改。

## 包内容、使用与打包验证

包内安装 `/usr/bin/magpie`，附带 README.Debian、上游 MIT LICENSE/copyright、THIRD_PARTY_NOTICES、44 个实际编译模块的可用顶层许可证/notice 文件、Go LICENSE、BUILD-INFO。Depends 为 `ca-certificates`，Recommends 为 `git, procps`。没有额外安装 GUI 库、工具链、后台服务或系统配置，也没有 maintainer script。

按用户新增要求复核包名、依赖和分包：参考 [Debian bookworm 的 Go CLI 应用 restic](https://packages.debian.org/bookworm/restic) 的独立应用包形式；Magpie 本包没有公共共享库、SONAME、头文件或独立 SDK 内容，因此保持单个 `magpie` 包，不人为拆出 `-dev` / `-common`。运行依赖以本包实际内容为准：`ca-certificates` 提供 HTTPS 所需的系统证书信任链；可选 Git 集成调用系统 Git（`internal/library/skillgit.go`、`internal/plugin/gitrev.go`），进程管理调用 `ps` / `pgrep`（`internal/proc/list_other.go`、`internal/agent/running_other.go`），因此推荐 `git, procps`。编译所需 Go SDK、GTK、WebKit 均不是这个静态 `nogui` 包的运行依赖。本次复核没有改变已审核的包或 SHA-256；可复用工具和外部库的独立交付仍按各自报告核对。

可按上游命令启动：

```sh
magpie --version
magpie --help
magpie serve --web=127.0.0.1:3430
# 或 magpie web --no-open
```

从终端打印的带 run key 链接进入 Web 界面。默认 gateway 为 `127.0.0.1:3425`。包安装后不自动启动服务。

`package.sh` 在专属 Podman 容器中使用 `dpkg-deb --root-owner-group -Zxz -z6` 打包。`SOURCE_DATE_EPOCH=1793427542` 来自锁定提交时间；文件属主为 root/root。另一个容器解包并对全部 `DEBIAN/md5sums` 检查成功，确认 Architecture、Version 和 Depends，且 `cmp` 确认包内程序与 smoke 程序相同。宿主仅运行解包程序的 `--version`，没有全局安装。

打包脚本、README 源稿、许可证收集脚本位于 `/tmp/magpie-20261031`，打包和核验日志为 `logs/package.log`、`logs/package-verify.log`。

## 功能限制与后续

- 上游 Bun 自动下载目标表只支持 amd64/arm64，没有 linux/loong64；本包不提供外部 Bun。需要该外部运行时的插件/订阅集成不属于已验证功能，不能因内置 moejs 编译成功就宣称全部插件可用。内置 moejs 与原生 gateway 请求路径已随构建/测试覆盖。
- 上游自动更新对本平台的 release asset 尚未验证，建议通过匹配的发行包升级。
- 真实 provider、交互浏览器和长期服务尚待用户场景验证；本包可以用于已经验证的 CLI/Web 和本地 gateway 路径，并保留上述完整测试边界供主 Agent 决定发布说明。
- 用户最新允许在确认 CLI+Web 成果后尝试 GUI；该尝试不在本包内。任何需要修改 Wails/Go 等依赖源码的工作仍须由主 Agent 按独立依赖流程安排，本执行 Agent 不修改依赖模块缓存。
