# TSRS 0.20.0-ts7.1.0-beta LoongArch64 工具交付报告

打包日期：2026-11-12（Asia/Shanghai）。本报告交付已成功编译的原生 TSRS 工具，没有重新编译、修改上游源码、安装宿主构建依赖或执行 Magpie。

## 上游版本和已有构建

- 官方仓库：https://github.com/oxc-project/tsrs
- 源码提交：`d197eeea709616d61930c2bd5eac048471e404b8`。
- TSRS workspace 版本：`0.20.0`；其 TypeScript 基线为 `7.1.0-beta`，提交 `3e9236361fa97a9e6c685f4833ac457ae2d8c50a`。
- 已有原生构建采用上游支持的 `noembed,tsrs_core/plain-ptrs` features，没有源码补丁。
- 实际 `--version` 输出：`Version 7.1.0-beta (tsrs 0.20.0-ts7.1.0-beta, microsoft/TypeScript@3e9236361fa9)`。
- 原生二进制 SHA256：`5b5d77e2f53a594eb6ad7d69f2f0c323904056cc9a92fb5044c4ca5831d5e345`。

## Debian/Ubuntu 参考和分包

官方 Debian/Ubuntu 对 `tsrs` 的全 suite 名字检索只返回不相关的 `postsrsd` 邮件服务，不能借用其名称或依赖。TSRS 作为本次实际生成的可复用工具，仍按根 `AGENTS.md` 第 6 节单独打包。

布局参考 [Debian node-typescript 6.0.3-2 control](https://sources.debian.org/data/main/n/node-typescript/6.0.3-2/debian/control) 和 [实际文件列表](https://packages.debian.org/sid/all/node-typescript/filelist)：发行包在一个包中提供命令入口和 `lib*.d.ts` 标准库数据，没有为这批声明文件另造空的 `-common` 包。本次相应交付一个实际架构的 `tsrs` 包：

| 项目 | 内容 |
| --- | --- |
| 包名 / 架构 | `tsrs` / `loongarch64` |
| Debian 版本 | `0.20.0+ts7.1.0~beta-1chinaos1` |
| 上游完整版本 | `0.20.0-ts7.1.0-beta`；Debian 版本用 `+ts` 标记 TypeScript 基线、`~beta` 保持 prerelease 排序 |
| 原生命令 | `/usr/lib/tsrs/tsrs` |
| 命令入口 | `/usr/bin/tsrs -> ../lib/tsrs/tsrs` |
| 声明数据 | `/usr/share/tsrs/lib*.d.ts`，116 个文件，共 3,870,196 字节 |
| 邻接兼容链接 | `/usr/lib/tsrs/lib*.d.ts -> ../../share/tsrs/lib*.d.ts` |

上游 `noembed` 实现先求 executable realpath，再要求同目录有 `lib.d.ts`。上述相对链接保留其原始查询行为，提取到独立目录时也能工作；没有修改上游路径逻辑或加入依赖 shell 的启动包装器。声明文件全部逐字节来自同一源码提交。

## 原生 ABI 和 Depends

在既有 `codex-v0.166.0-v8-directory-probe-20261107` Podman 容器中，`dpkg-shlibdeps` 实际导出：

```text
libc6 (>= 2.28), libgcc1 (>= 1:4.2)
```

程序为 LoongArch ELF，flags `0x3`，loader `/lib64/ld.so.1`，最高 GLIBC 符号版本为 `2.28`。其 NEEDED 为 `libgcc_s.so.1`、`libpthread.so.0`、`libm.so.6`、`libdl.so.2`、`libc.so.6`，移除 `LD_LIBRARY_PATH` 后全部解析成功。LLVM、Rust 编译器和 Node.js 不在这个原生工具的运行时依赖中，没有机械复制构建依赖为 Depends。

## 内容、许可和验证

包内保留上游完整 `LICENSE`、`NOTICE`、`licenses/Go-LICENSE`、README 和声明文件原始 copyright 头。另保守收录已缓存 Cargo.lock 依赖中的 265 个 license/notice 文件及各 crate 原始 Cargo.toml，保存于 `/usr/share/doc/tsrs/third-party-licenses/`。这批通知覆盖已编译依赖，也包含部分未选中的开发/平台条目；没有给未使用 crate 增加运行时 Depends。两项没有通知文件的 lock 条目 `cpufeatures 0.3.1`、`r-efi 6.0.0` 均没有该目标的构建 fingerprint，未作为本次二进制选中依赖。

所有包组装、`dpkg-deb`、提取和工具执行都在上述既有容器内完成，只在本任务独立目录写入暂存内容。没有替换容器共享 Rust、Node、pnpm、esbuild 或应用源码。

- 包提取后的全部 MD5 校验通过；二进制与原有构建输出相同，116 个声明文件逐字节一致。
- 从提取后的 `/usr/bin/tsrs` 运行 `--version` 成功。
- 严格 ES2022 + DOM 普通 type-check fixture 使用 `Array`、`Promise`、`document`，退出 0，证实邻接声明查找可用。
- 错误赋值 fixture 返回 TS2322、退出 2，证实执行了真实类型检查。
- 在移除 `LD_LIBRARY_PATH` 的环境中依赖解析通过。
- 没有安装到宿主或容器共享 dpkg root，没有重跑应用完整构建/测试套件。

## 交付

[tsrs_v0.20.0+ts7.1.0~beta-1chinaos1_loongarch64.deb](../binaries/tsrs/loongarch64/tsrs_v0.20.0+ts7.1.0~beta-1chinaos1_loongarch64.deb)，6,511,140 字节，SHA256：

```text
71bec09c9ff5fe99a95ca646bc2df01b8b76cf71f633c27c7ef4f72326c52284
```

同目录已写入 `.deb.sha256`。完整组装脚本、原始官方参考、输入通知清单、包 manifest、MD5 结果、ABI 和提取后 smoke 位于：

```text
/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/tool-package-review-20261111/
```

主要文件为 `assemble-tsrs.py`、`tsrs-package-manifest.json`、`tsrs-native-abi.json`、`tsrs-shlibdeps.log`、`tsrs-extracted-smoke.log`、`tsrs-package-md5.log`。没有 commit 或 push；此处完成的是工具包的提取与运行验证，不宣称宿主安装或所有 TSRS conformance 测试通过。
