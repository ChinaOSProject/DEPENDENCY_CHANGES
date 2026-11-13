# Rolldown 1.2.9 LoongArch64 原生绑定审查报告

审查日期：2026-11-12（Asia/Shanghai）。本报告覆盖已成功构建的 Node 原生绑定、运行验证和独立分包判定，没有修改上游源码或重编译。

## 上游和产物

- 官方仓库：https://github.com/rolldown/rolldown
- 版本：`1.2.9`。
- 源码提交：`5b4746e442989d770c606ce08d2737e6aafbd25d`。
- 构建输出：`target/rolldown/release/librolldown_binding.so`，21,124,232 字节。
- SHA256：`89823577793d722507eb4eb3ef949914f290f1b9d1a64c44cd3f0331be3b8cf1`。
- 主 Agent 已放入原有 npm loader 路径 `rolldown/dist/shared/rolldown-binding.linux-loong64-gnu.node`；本次检查确认 hash 一致，没有再替换该文件。

原生输出为 LoongArch ELF，flags `0x3`，最高 GLIBC 符号版本 `2.28`。移除 `LD_LIBRARY_PATH` 后 native NEEDED 全部解析成功，没有 LLVM 运行时依赖。

在既有 Podman 容器内，使用原有 task-local Node 和实际 `rolldown` JS API 对一个独立 fixture 进行 bundle，随后加载生成模块，验证其导出 `answer === 42`。该代表性检查没有运行应用套件，也没有重新构建 Rust 或重装 npm。

## Debian/Ubuntu 对应关系及例外

[Debian 官方全 suite 名字检索](https://packages.debian.org/search?keywords=rolldown&searchon=names&suite=all&section=all) 和 [Ubuntu 官方全 suite 名字检索](https://packages.ubuntu.com/search?keywords=rolldown&searchon=names&suite=all&section=all) 均返回无结果，包括没有可核对的 Node 包、Rust source-dev 包或 CLI 包。原始响应和 SHA256 保存于下文证据目录的 `correspondence-searches.json`、`debian-rolldown.html` 和 `ubuntu-rolldown.html`。

当前交付用途是应用前端构建中的绑定中间产物，没有单独生成已约定的系统 Node SDK 或 CLI。一个裸 `.node` 文件不能代替完整 npm 包：实际 `rolldown` 1.2.9 JS 包还依赖 `@oxc-project/types =0.150.0` 与 `@rolldown/pluginutils ^1.0.0` 等 loader/API 内容，不能据此虚构一个缺失 JS 依赖的系统包。

依据根 `AGENTS.md` 第 6 节“没有常见 Debian/Ubuntu 独立发行包对应”的内部中间产物例外，本次不额外制造 Rolldown `.deb`。结论来自官方分包对应关系和实际用途，不以静态链接或“不是应用包”作为豁免。若以后交付独立 Rolldown CLI 或系统 Node API，须按完整可用内容另行完成分包、依赖和许可验证。

上游 `LICENSE`、npm 包原许可和完整源码保留在专属输入目录，没有以一个空包代替许可/源码归档。此处不声称独立系统安装已通过。

## 证据和状态

证据目录：

```text
/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/tool-package-review-20261111/
```

`rolldown-native-abi.json` 记录 ELF、NEEDED、符号版本和无 `LD_LIBRARY_PATH` 的依赖解析；`native-node-smoke.mjs` / `native-node-smoke.json` 记录实际 JS API fixture 和 loader 使用的 binding hash。主 Agent 原构建记录为上级目录 `logs/rolldown-native-build-main-20261111.log`。

没有宿主安装、共享 cache/source 修改、commit、push 或 Magpie 执行。其他已完成的 pnpm、esbuild 和 Wails 交付没有被修改。
