# pnpm 12.10.0：Clash 前端构建工具

上游仓库为 [pnpm/pnpm](https://github.com/pnpm/pnpm)。Clash Verge Rev v2.5.8 的 `packageManager` 锁定 pnpm 12.10.0。本机未找到该版本的 LoongArch 原生可执行文件；官方同版本 WASM 在本机原生 V8 上因缺少 Wasm SIMD 无法运行，在 LATX 下也未完成版本检查。

本包使用官方 npm 制品 [`@pnpm/exe.linux-x64-musl@12.10.0`](https://registry.npmjs.org/@pnpm%2fexe.linux-x64-musl/12.10.0)，并以专用 Podman 容器中的既有 LATX 执行。npm 元数据给出该包为 Linux x64/musl、未包含 Node.js `engines` 字段；上游 `pnpm@12.10.0` JavaScript 包另声明 `engines.node >=18.*`，但本包实际交付的是 standalone 可执行文件。ELF 检查显示它是 x86-64，未请求 `PT_INTERP`、没有 `DT_NEEDED` 或动态 GLIBC 版本要求；Node.js 运行时随可执行文件提供，因此 Debian 包不声明外部 `nodejs` 依赖。

交付包 `pnpm` 的版本为 `12.10.0-1`，架构如实标为 `amd64`，大小为 15,149,408 字节，下载路径为 [pnpm_v12.10.0_amd64.deb](https://github.com/ChinaOSProject/binaries/blob/main/pnpm/amd64/pnpm_v12.10.0_amd64.deb)。包只安装 `/usr/bin/pnpm` 和上游许可证/第三方声明；未额外提供 `pn`、`pnx` 或 `pnpx` 入口，也未添加虚假的 `Provides`。

Debian sid 的 [`node-corepack` 0.35.0-2 控制文件](https://sources.debian.org/data/main/n/node-corepack/0.35.0-2/debian/control)将该包标为 `Architecture: all`、依赖 `nodejs:any`，且未声明 `Conflicts`、`Breaks` 或 `Replaces`。其[文件清单](https://packages.debian.org/sid/all/node-corepack/filelist)显示它安装 `/usr/bin/corepack`，并在 `/usr/share/nodejs/corepack/shims/` 提供 `pnpm` 与 `pnpx` shim；它没有在安装时拥有 `/usr/bin/pnpm` 或 `/usr/bin/pnpx`。本包与 Debian `node-corepack` 的 dpkg 文件清单没有交集，所以没有增加包级冲突关系。Corepack shim 目录及其启用行为与 standalone 包的文件归属差异已按 Debian 实际布局记录。

官方 npm [`pnpm@12.10.0` 元数据](https://registry.npmjs.org/pnpm/12.10.0)记录 JavaScript 包入口及 `>=18.*` engine 要求；[独立 exe 元数据](https://registry.npmjs.org/@pnpm%2fexe.linux-x64-musl/12.10.0)记录本次实际使用的 x64/musl 可执行制品。其原始 tgz SHA256 为 `220858a3a4d20014b32fdd9722c87b7bd372e8efe48861dcc43ba8b271b733a2`，ELF SHA256 为 `681a7f90ac46a3edb224232823356cf516aff16ce0143474a4d5dc3b099874f7`。最终 Debian 包 SHA256 为 `05aae265a72e258c2e1fdc03ad4d110af6830896ccd68e9870c4ab4df62910a0`。

已用 LATX 运行最终包中同一 ELF，版本输出 `12.10.0`。该 ELF 为 x86-64 制品，不能标为 LoongArch 原生程序；在 LoongArch 上运行此命令需要已有的 LATX。Clash 的项目脚本另外使用了 x64 Node 环境。本次没有把模拟器放进包，也没有让 pnpm 进入 Clash 最终应用包。此前该同一 ELF 已完成 Clash 的 frozen-lockfile 安装、TypeScript/Vite 前端构建、11 个文件共 45 项测试与架构边界检查。

主 Agent 在全新临时 Podman 容器中启用 `amd64` 外来架构并实际安装最终 deb，确认状态为 `install ok installed`、版本与架构正确、没有未满足依赖；安装后的 `/usr/bin/pnpm` SHA256 与上述已运行 ELF 完全一致。这个验证未在宿主启用外来架构或安装软件，记录在 `logs/pnpm-main-installcheck.log`。独立检查确认包内只有命令、许可证和第三方声明三个普通文件，ELF machine 为 62，没有不安全的归档路径。此验证针对最终包的安装元数据；既有前端构建和测试未重复执行。

构建复用了既有 Node 24.21.0 oldworld、GCC 13.4.0、x64 Node 24.21.0/x86 运行根及 LATX；它们不是本次新生成的软件包。相关构建、打包与测试日志保存在本任务 `/tmp` 工作目录的 `logs/package-pnpm-02.log`、`logs/pnpm-install-03.log`、`logs/web-build-01.log`、`logs/frontend-tests-01.log` 中。所有安装、测试和打包均在项目专用 Podman 环境中完成，宿主未安装依赖；主 Agent 负责统一发布。
