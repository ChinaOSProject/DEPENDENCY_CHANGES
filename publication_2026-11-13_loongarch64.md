# 2026-11-13 LoongArch 软件交付归档

已将 **220 个已验收包**上传到 [ChinaOSProject/binaries](https://github.com/ChinaOSProject/binaries/tree/2933e168c9d1ac0324c1d0505f1385c9a39f00a1)，提交 `2933e168c9d1ac0324c1d0505f1385c9a39f00a1`。总计 707,332,240 字节；58 个包的 control 架构为 loongarch64，162 个为 all。已通过认证 Git LFS batch 下载查询逐项核对全部对象 ID、大小及可下载状态；上传 hook 成功，没有重复下载整批 707 MB。

逐包 Package、完整 Version、Architecture、字节数、SHA256、校验文件和固定提交的下载链接见 [机器可读清单](publication_2026-11-13_loongarch64.json)。现有历史测试/安装证据保持原样，本轮未重复成功套件。

| 交付组 | 包数 |
|---|---:|
| Clash 私有 GLib/libsoup/GStreamer/WebKit/GCC family | 39 |
| Clash Verge 2.5.10-1loong2、两通道 Mihomo、公共数据 | 4 |
| Magpie stable 0.1.3078-1loong2 修复包 | 1 |
| pnpm / esbuild / TSRS | 3 |
| LightningCSS 源码开发依赖 | 110 |
| float16 及精确源码依赖 | 5 |
| GN / Ninja | 2 |
| LLVM 运行库、修复 LLD21、限范围 LLVM23 开发包 | 4 |
| 六项可复用 C SDK | 6 |
| psm / cc / shlex 源码开发包 | 3 |
| 首组叶依赖源码开发包 | 10 |
| Exact33 版本源码开发包 | 33 |

## 源码分支

| 仓库 | debian10 | loong64 |
|---|---|---|
| [Codex](https://github.com/ChinaOSProject/codex/tree/loong64) | `5577d8c9895d5d8d046e4633ba0e1c2739c527df` | `4ff4a9d9aa75a4c748facdc9a286a3d10519e1fe` |
| [Herdr](https://github.com/ChinaOSProject/herdr/tree/loong64) | `5b99272554457d1d9e29d1dce17fb2abb365f645` | `ef1f4f159923fd9271391d5fc483b6c2bf806220` |

两仓分支均已通过实际远端 refs 精确核验。Herdr 0.9.5 的正式包与历史完整测试记录已在此前归档，本次未重建或重复上传该包。

Wry 的 `c8bb6e63fd696160609d7795bbb32566e8ed6086` 和 float16 的 `9028edc088ab4419536a5aa0b30cc227fcb5eb03` 已各自在 debian10 提交并快进到 loong64，main 保留上游基线。对应补丁、报告及实际二进制/源码包一致。其源码远端尚未创建或授权访问；本次没有推送到上游。

## 保留项与验证边界

ChinaOSProject 下的 magpie、clash-verge-rev、clash-verge-service-ipc、seccompiler、wails、wry、float16、llvm-project、rusty_v8 源码仓库均返回 API 404 / SSH Repository not found。当前 SSH 凭据可推送已有仓库，但没有用于创建仓库的 GitHub API 登录。已完成的源码提交和未提交 diff 保留在各自工作树，不能将补丁归档说成这些源码仓库已推送。

Wails 源码仍未提交，缺少其 AGENTS.md 强制要求的 `coderabbit --plain` 审查。Rust 1.99.1 SDK 的 ABI 回归及源码审阅门禁尚未通过。Codex 0.166.0 的真实匹配 helper 虽已构建成功，但尚缺最终对象/通知、成对包和运行验收，因此未把旧版 compatibility helper 或新未验收 helper 纳入本次发布。

ordinary16 的消费者夹具失败，Next35 仅完成组装，Next17 仅完成准备；Zstd 及其他工具草案也未进入本清单。已撤回的 GLib 单体、旧 GN/Ninja、旧编译器候选和 Git NO_CURL 包均排除。源码开发包只确认各报告所列 feature/consumer 范围，不表示已完成全部 548 个已观察 Rust crate 版本的依赖交付。

Magpie 修复包保留既有失败/成功回归及菜单路径验证记录。**本轮没有执行、启动、停止、重启或向 Magpie 发信号**；活动会话仍按原约束保留，修复后真实窗口状态转换的验收未做。软件仓库为项目指定 Git LFS 二进制仓库，此处不宣称另一个 APT 索引或签名发布已经完成。
