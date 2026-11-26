# Firefox 160.0.1-3 LoongArch64 seccomp 适配报告

## 源码与构建

- 在 Firefox 160.0.1 的 Linux LoongArch64 构建中默认启用 `MOZ_SANDBOX`。沿用 LoongArch ABI v0、Mozilla 官方品牌和参照 Scratch 3 打包的应用菜单入口。基础适配和 Debian 依赖依据见 [160.0.1-2 适配报告](firefox_v160.0.1-2_loongarch64.md)。
- 补齐 LoongArch64 的 seccomp 审计架构标识、系统调用汇编、SIGSYS 寄存器映射和旧系统头文件中的常量。使用 libc 的 `sigprocmask` 匹配目标 Linux 5.4.18 的 16 字节信号掩码。
- `statx` 路径查询使用 Firefox 文件代理的路径权限。文件描述符查询将描述符传给代理，代理查询该描述符指向的对象。无代理分支检测内核是否支持空指针路径，旧内核继续使用 `fstat` 回退。Firefox 保留默认内容沙箱等级 6。
- 使用现有 Debian 10 LoongArch 构建环境、Clang 22.1.8、ABI v0 Rust 1.98.1 和 `/mnt/chinaos-cache` 对象缓存。复用的 14 个 NSPR、NSS、SQLite、NSS 工具和 wasm2c 配套二进制与上一版哈希一致。

## 验证

- 在原生 `5.4.18-168-generic` 内核上验证系统调用汇编、TSYNC、SIGSYS 六个参数和错误返回，以及代理描述符传递。内容策略探针通过路径与描述符查询、拒绝未经授权的路径、阻止进程创建和旧内核 `fstat` 回退测试。现代 LoongArch 内核尚未进行原生验收。
- 最终包的 24 个 ELF 均通过 LoongArch ABI v0 检查，ELF flags 为 `0x3`，动态可执行文件使用 `/lib64/ld.so.1`。最高 GLIBC 导入符号版本为 `2.28`。包内 MD5 校验通过。
- 从最终包启动浏览器并加载本地 HTTP 页面。WebAssembly、Canvas 和 Cookie 测试通过。读取 `/proc` 确认被测网页内容进程的 17 个线程均为 `Seccomp: 2`、`NoNewPrivs: 1`，浏览器主进程为 `Seccomp: 0`。上一版对照包的内容进程显示 `Seccomp: 0`。
- 在 glibc 2.28 隔离容器中完成从 `160.0.1-2` 升级。普通用户通过 `gtk-launch firefox` 打开测试页面并显示 X11 窗口，窗口类与桌面条目一致。卸载后检查应用菜单入口、图标和 MIME 记录均已清理。
- 本地构建缓存的 `logs/firefox-seccomp-{build,native-probe,policy-runtime,baseline,deb,smoke,runtime,desktop,components}.log` 保存了构建和验收记录。

## 交付

- 安装包为 `firefox_v160.0.1-3_loongarch64.deb`，大小为 81,955,532 字节。
- SHA-256 为 `4fc7431cdfeb6a01cce07884e9b3f7e8940d32cfd913cb8c1e56bb672338b187`。
- 官方品牌二进制在本地交付。公开分发待确认 Mozilla 的书面许可。[Mozilla 软件分发政策](https://www.mozilla.org/en-US/foundation/trademarks/distribution-policy/)
