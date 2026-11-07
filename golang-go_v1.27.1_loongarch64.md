# Go v1.27.1 old-world SDK / loongarch64 交付报告

日期：2026-11-04。此报告记录 Magpie v0.1.2339 使用的可复用 Go 1.27.1 old-world SDK。按 Debian Go 分包惯例拆为 compiler、source、documentation 和两个 default metapackage，共 6 个包；**原样重打包发布方工具链，没有修改或从源码重编 Go。**早期单体候选保存在任务备份中，不用于发布。

## 最终交付包组

| Debian 包 | Debian Version | Architecture | 文件 | 字节 | SHA-256 |
| --- | --- | --- | --- | ---: | --- |
| `golang-1.27-go` | `1.27.1-2` | `loongarch64` | `binaries/golang-1.27-go/loongarch64/golang-1.27-go_v1.27.1_loongarch64.deb` | 21,575,168 | `c5d467913d1da8b3721b66b31f3829c404d8a575b0eb98124c5fece9c63bb9b5` |
| `golang-1.27-src` | `1.27.1-2` | `all` | `binaries/golang-1.27-src/all/golang-1.27-src_v1.27.1_all.deb` | 25,163,608 | `9db8282e1c9e039d3963e124edddde1d177a7908f25fdb5c29fa2856f28f388b` |
| `golang-1.27-doc` | `1.27.1-2` | `all` | `binaries/golang-1.27-doc/all/golang-1.27-doc_v1.27.1_all.deb` | 97,740 | `62cf2d9b67e8638128666d0b229668c744873226492d56c6103daf91e02b2490` |
| `golang-1.27` | `1.27.1-2` | `all` | `binaries/golang-1.27/all/golang-1.27_v1.27.1_all.deb` | 1,796 | `a3d7f104405525d0e449a003830222544e597f037ef82ab20318d2c6c37a52a6` |
| `golang-go` | `2:1.27.1-2` | `loongarch64` | `binaries/golang-go/loongarch64/golang-go_v1.27.1_loongarch64.deb` | 1,904 | `3d553d56e764d9fcef1f788ba87e28071ecae8489bd3204d640f5fab4d617721` |
| `golang-src` | `2:1.27.1-2` | `all` | `binaries/golang-src/all/golang-src_v1.27.1_all.deb` | 1,780 | `2629839b413292cb6b5136cb9cf8f0377a085c9e060883af08f4c0dcd5067b9c` |

六个 Go 包都在各自 `/usr/share/doc/<package>/copyright` 中附带完整 upstream BSD `LICENSE`，文本 SHA-256 为 `911f8f5782931320f5b8d1160a76365b83aea6447ee6c04fa6d5591467db9dad`。整组包文件与构建 staging 完全一致；详见 `/tmp/magpie-20261031/logs/tool-group-package-verification.json` 和 `logs/tool-group-install-smoke.log`。

## 交付结果与来源

- 发布方源码仓库：<https://github.com/loong64-abi1-0/golang>
- 发布页：<https://github.com/loong64-abi1-0/golang/releases/tag/go1.27.1>
- 原始归档：<https://github.com/loong64-abi1-0/golang/releases/download/go1.27.1/go1.27.1.linux-loong64.tar.gz>
- 归档大小：70,536,699 字节。
- 归档 SHA-256：`e4f8920f59c1da3604fba3ff2d668e7ff607d74b4fd548230149e8c8e513dff8`，已与发布 API 的 digest 及发布方独立校验文件核对一致。
- 实际 `go version`：**`go version go1.27.1 linux/loong64`**。

主入口 `golang-go` 是 Debian default metapackage，版本 `2:1.27.1-2`，架构 **`loongarch64`**，文件大小 **1,904 字节**，SHA-256 **`3d553d56e764d9fcef1f788ba87e28071ecae8489bd3204d640f5fab4d617721`**。完整 SDK payload 位于 `golang-1.27-go`、`golang-1.27-src` 和 `golang-1.27-doc`。Go 自身将目标架构称为 `loong64`，宿主 dpkg 架构和架构专属交付文件名使用 **`loongarch64`**。

## 隔离与包内容

SDK 最初解包在 Magpie 专属容器的 `/opt/magpie-go`。整个下载准备、编译使用、复制 staging、封包、包安装及编译验证均在专属 rootless Podman 环境完成。基座为 `cr.loongnix.cn/library/debian:buster-slim`，专用镜像包括 `localhost/magpie-build:v0.1.2339-buster-loongarch64` 和 `localhost/magpie-tools:v0.1.2339-buster-loongarch64`。

专用 storage 为 `/tmp/magpie-20261031/podman-root` 与 `podman-run`，调用入口 `/tmp/magpie-20261031/podman-magpie`。没有在宿主安装 Go、编译器或构建依赖，也没有在宿主执行编译。

old-world 发布归档中的 Go 可执行文件和 SDK 源文件未修改、未 patch、未重编；这里只按 Debian 路径分包和 relocation，源文件按原字节复制。compiler 包安装到 **`/usr/lib/go-1.27`**，SDK source 和 include 位于 `/usr/share/go-1.27`，compiler 包以符号链接访问它们；`/usr/bin/go`、`/usr/bin/gofmt` 为 default package 提供的相对链接。compiler 包 Depends 精确匹配 `golang-1.27-src (= 1.27.1-2)`，并 Recommends `gcc, g++, libc6-dev, pkg-config`。`golang-go` Depends `golang-1.27-go (= 1.27.1-2), golang-src (>= 2:1.27.1-2)`；`golang-src` 依赖同版本 `golang-1.27-src`。Cgo 和 external linking 需要匹配目标 ABI 的 C 编译器与 libc 开发文件；Go 包含 Cgo 工具但不包含 C 编译器。六个 Go 包各自携带完整上游 BSD license，不依赖 optional doc 包提供许可文本。

Debian 1.27 packaging rules、install/link manifests 和 `golang-defaults` control 是分包参考，记录在 `/tmp/magpie-20261031/packaging-audit-20261102.md` 与 `logs/reference-go127-{control,rules,links}.txt`、`logs/reference-go-defaults-control.txt`。对应参考使用 Debian 的 `loong64` 架构名；此目标基座的 dpkg 架构是 `loongarch64`。参考的源码与 compiler 包按架构生成内容拆分，本包也将六个生成 Go 文件归入 compiler 包，并让 source/compiler 版本精确匹配。default `golang-go` / `golang-src` 使用 epoch `2:` 和上游默认入口关系。

| 包 | 归属文件与用途 |
| --- | --- |
| `golang-1.27-go` | `/usr/lib/go-1.27/bin`、`/usr/lib/go-1.27/pkg/tool` 和访问 source 的链接；另拥有 `/usr/share/go-1.27/src/` 下六个生成文件：`cmd/go/internal/cfg/zdefaultcc.go`、`internal/runtime/sys/zversion.go`、`time/tzdata/zzipdata.go`、`cmd/cgo/zdefaultcc.go`、`cmd/internal/objabi/zbootstrap.go`、`internal/buildcfg/zbootstrap.go` |
| `golang-1.27-src` | `/usr/share/go-1.27/{api,lib,misc,src,test,pkg/include}`，以上六个生成文件由 compiler 包持有 |
| `golang-1.27-doc` | `/usr/share/doc/golang-1.27-doc/html` 与 Go 文档 |
| `golang-1.27` | 安装 compiler、source、doc 的版本 metapackage |
| `golang-go` | 默认 `/usr/bin/go`、`/usr/bin/gofmt` 和 `/usr/lib/go` 链接 |
| `golang-src` | 默认 `/usr/share/go` 链接到 Go 1.27 source |

每个包都单独提供 `/usr/share/doc/<package>/copyright`，内容是完整 upstream BSD `LICENSE`（SHA-256 `911f8f5782931320f5b8d1160a76365b83aea6447ee6c04fa6d5591467db9dad`）。

## ABI 与安装位置验证

对 SDK 的 `bin` 和 `pkg/tool/linux_loong64` 中全部 **10 个可运行 ELF 工具**逐个检查：LoongArch ELF64 / LP64、flags **`0x3`**，均为静态链接，无 `PT_INTERP` 和动态依赖段。因此这些工具本身没有动态 GLIBC 符号版本 floor。标准库源码目录中的其他架构测试对象或 `.syso` 资料不属于本机可运行工具，按原始 SDK 保留。

包安装后 `go env GOROOT` 实际为 **`/usr/lib/go-1.27`**，未残留构建 staging 的 `/opt/magpie-go`。真实 Kylin V10 SP1 / kernel 5.4.18 / glibc 2.28 / old-world 宿主执行暂存 SDK 的 `go version` 成功；容器使用本 SDK 编译的 hello 在真实宿主运行输出 **`old-world Go SDK OK`**。宿主没有安装本包，亦未据本检查宣称对所有内核或 new-world 系统兼容。

## 已完成验证与边界

1. 在专属工具验证容器中对 Git/Go 八包执行 apt 本地整组重装和升级，8 包最终均为 installed，`dpkg --audit` 干净。
2. 安装后的 `/usr/bin/go version` 精确为 `go1.27.1 linux/loong64`，`go env GOROOT` 为 `/usr/lib/go-1.27`，default `/usr/lib/go` symlink 正确。
3. 在容器编译并运行独立 hello，输出 `old-world Go SDK OK`；gofmt 格式化检查通过。
4. Git `--version`、HTTP/HTTPS remote helper 与 man page 入口可用；不支持的 SVN/CVS/Perl launcher 不存在。
5. 容器解包对 `golang-1.27-src` 的 **15,603** 个文件、compiler 的 **19** 个普通文件和 **6** 个 symlink、doc 的 **21** 个文件，以及三个轻量包逐项校验 md5、staging 文件集合、内容、类型与模式。六份版权文件与发布归档中的 LICENSE 完全一致。

Magpie 使用本 SDK 完成 `go mod download`、`go mod verify`、`CGO_ENABLED=0 go vet -p 2 -tags nogui ./...` 与最终 CLI/Web 程序编译；静态程序已通过真实宿主 gateway/Web smoke。Magpie `go.mod` 要求 Go 1.26.8，本任务实际使用 1.27.1；最后完整 nogui suite 为 76 包通过、6 包失败、17 包无测试、18 个顶层失败，**不是全套通过**。其中 JSON 行分隔符、图片预算和 allocation 断言等与 Go 版本相关的可能边界详见 Magpie 正式报告，不作为 Go 工具链已通过全部兼容测试的依据。

没有执行 Go 的完整标准库/compiler test suite、race、全平台交叉编译或 Cgo/external-linking 功能验证。归档来自上述 old-world 发布方，不冒充 go.dev 官方二进制。使用 `GOTOOLCHAIN=local` 可避免构建时自动下载另一版本的工具链。

封包在容器使用 `dpkg-deb --root-owner-group -Zxz -z6`，文件属主 root/root，`SOURCE_DATE_EPOCH=1793427542`。Go 编译器、标准库和模块缓存均未由本任务修改。

## 证据与关联交付

全部辅助资料位于仓库外 `/tmp/magpie-20261031`：

- 原始归档：`downloads/go1.27.1.linux-loong64.tar.gz`；发布 API 留档：`downloads/go-oldworld-releases.json`。
- 封包脚本：`package-tools.sh`；封包日志：`logs/package-tools.log`。
- ELF：`logs/go-sdk-abi.txt`、`logs/go-all-elf-abi.json`。
- 安装、编译和 gofmt：`tools-smoke.py`、`logs/tools-smoke.json`、`logs/tools-verify-retry.log`。
- 真实宿主：`logs/tools-host-smoke.log`；预编译验证程序：`out/toolchain-hello`。
- 包完整性：`verify-split-tool-group.py`、`logs/tool-group-package-verification.json`、`logs/tool-group-install-smoke.log`、`logs/tool-group-build-manifest.json`。

本次另一项需独立交付的可复用软件 Git 2.43.0 和匹配的 `git-man` 见 `git_v2.43.0_loongarch64.md`。apt 安装的既有 GCC/binutils/curl/gettext 等没有在本任务从源码重建；Magpie 的 Go module `-dev` source packages 在后续独立交付记录中说明。

Magpie 本体交付为 `magpie_v0.1.2339_loongarch64.deb`（CLI + Web，无原生 GUI），其正式报告保持主 Agent 已审核版本。源码及依赖源码零修改，没有新增源码 commit。本执行 Agent 没有 push，包与报告交主 Agent 统一审核发布。
