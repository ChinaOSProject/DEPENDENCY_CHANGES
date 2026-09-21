# 依赖适配说明

## npm 项目

- GitHub: `https://github.com/npm/cli`
- Version: `11.19.1`
- Type: Node.js command line package manager。

## Debian 10 运行依赖

- Dependency: `node`
- Version: `20.20.2`
- Type: 预编译 Node.js runtime deb。
- Reason: npm `11.19.1` 的 `engines.node` 要求为 `^20.17.0 || >=22.9.0`；目标系统基座为 Debian 10 / glibc 2.28。
- Root Cause: Debian 10 系统提供的 Node.js 版本无法满足 npm `11.19.1` 的运行要求。
- Changes: npm deb 声明 `node (>= 20.20.2)`，并将 npm 安装到 `/usr/lib/node_modules/npm`，提供 `/usr/bin/npm` 和 `/usr/bin/npx`。当前 amd64 验证使用上级 `binaries/node/amd64/node_v20.20.2_amd64.deb`。
- Compatibility Impact: npm deb 为架构无关包；Node.js 二进制文件仍需按目标架构分别提供，当前 amd64 Node.js 二进制文件的最高 glibc 符号版本为 `GLIBC_2.28`。npm JavaScript 依赖随 npm tarball bundled。
- Upstream Status: 未修改上游源码；本次为 Debian 10 runtime 依赖与 deb 打包配置。
- Validation: 使用 Node.js `20.20.2` 执行 `npm pack`；deb 元数据声明 `Architecture: all` 与 `node (>= 20.20.2)`；隔离安装验证将在包生成后执行。AArch64 与 LoongArch64 runtime 尚未在当前环境执行验证。
- Project-side Changes: 无源码修改；生成 `npm_v11.19.1_all.deb`。
