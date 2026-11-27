# Firefox 160.0.1-4 LoongArch64 启动页适配报告

## 启动页修复

- 麒麟系统的 `ubuntukylin-default-settings` 可将 Firefox 首页与首次欢迎页都设置为 `file:///usr/share/doc/HTML/index.html`。目标系统缺少该文件时，Firefox 启动报 `fileNotFound`。
- Firefox 在读取发行版默认配置后检查这两个设置。仅当 Linux 上加载了 `ubuntukylin.cfg`、该本地文件缺失，并且对应默认值仍指向该文件时，才将默认首页回退到 `about:home`，将默认欢迎页回退到 `about:welcome`。用户自定义首页、锁定的首选项和系统配置文件均保留。

## 验证

- 在原生 LoongArch、glibc 2.28 的隔离容器中加入实机的两份麒麟配置。新包首次启动显示 Firefox 欢迎页，再次启动显示内置首页。自定义首页和麒麟的固定快捷方式均保留。
- 使用同一个 Firefox 用户配置从 `160.0.1-3` 升级到 `160.0.1-4`。旧版复现 `fileNotFound`，新版直接打开内置首页，配置目录的应用 BuildID 随升级更新。补回可用的本地文档后，Firefox 继续使用发行版指定的本地页面。
- WebAssembly、Canvas、Cookie 与截图冒烟复测通过。新版首次冒烟在保存无头截图后退出时出现一次 `SIGSEGV`，随后用新旧两个安装包复测均未复现。启动页、ABI 和复测日志保存在构建缓存的 `logs/firefox-homepage-{install,abi}.log` 与 `firefox-homepage-smoke-retry.0tXQh9/`。
- 从最终包启动并加载本地 HTTP 页面，内容进程线程继续显示 `Seccomp: 2` 和 `NoNewPrivs: 1`。记录位于 `logs/firefox-homepage-seccomp.log`。

## 构建与交付

- 使用原有 Debian 10 隔离环境和 `/mnt/chinaos-cache` 对象缓存执行 `mach build faster` 与 `mach package`。重新生成的 `omni.ja` 只有 `BrowserGlue.sys.mjs` 内容变化。
- 应用 BuildID 更新为 `20261127043342`，使相同 Firefox 版本的 Debian 修订号升级触发 Gecko 正常的启动缓存失效流程。重新生成 `application.ini` 并重建 `firefox-bin`。Gecko 引擎及其 BuildID 沿用 `160.0.1-3`。
- Debian 包沿用 [`160.0.1-3`](firefox_v160.0.1-3_loongarch64.md) 的分包方式及运行依赖。Firefox 启动程序使用 `/lib64/ld.so.1`，最高 GLIBC 导入符号为 `2.27`。包内其余 23 个 ELF 与上一版一致，整包保持 LoongArch ABI v0 和 GLIBC 2.28。包内 MD5 校验通过。
- 安装包 `firefox_v160.0.1-4_loongarch64.deb` 大小为 81,952,740 字节，SHA-256 为 `ab7e9d566df7f4cddb47aa8f4384c8cc589972908a94b78870cd1a4a7258d030`。
