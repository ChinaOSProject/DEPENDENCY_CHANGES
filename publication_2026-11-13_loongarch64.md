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
| [Magpie stable 0.1.3078](https://github.com/ChinaOSProject/magpie/tree/loong64) | `ef176e21f6c40e5a623709bb0e369647173964de` | `ef176e21f6c40e5a623709bb0e369647173964de` |
| [Clash Verge Rev 2.5.10](https://github.com/ChinaOSProject/clash-verge-rev/tree/loong64) | `c83f7ad529c7c9cb3d2d54a0955ef8c1b22a411b` | `c83f7ad529c7c9cb3d2d54a0955ef8c1b22a411b` |
| [service-ipc 2.7.7](https://github.com/ChinaOSProject/clash-verge-service-ipc/tree/loong64) | `eaaae99bb24c77155dbea95b3964f6d5be42b0f1` | `eaaae99bb24c77155dbea95b3964f6d5be42b0f1` |
| [seccompiler 0.5.0](https://github.com/ChinaOSProject/seccompiler/tree/loong64) | `c3cf77d65815037931ae5bc2fca010713defdc8c` | `8bb8d11bbd729d6b1a64f2aa1ca466cc802f043b` |
| [Wry 0.57.1](https://github.com/ChinaOSProject/wry/tree/loong64) | `c8bb6e63fd696160609d7795bbb32566e8ed6086` | `c8bb6e63fd696160609d7795bbb32566e8ed6086` |
| [Rust float16 0.1.5](https://github.com/ChinaOSProject/rust-float16/tree/loong64) | `9028edc088ab4419536a5aa0b30cc227fcb5eb03` | `9028edc088ab4419536a5aa0b30cc227fcb5eb03` |

以上分支均已通过实际远端 refs 精确核验。用户创建 fork 后新增的分支均通过普通原子推送发布，未改动 fork 默认分支。Magpie、Clash 应用源码未修改；应用分支归档精确构建基线，依赖源码修改由各独立仓库承载。Herdr 0.9.5 的正式包与历史完整测试记录已在此前归档，本次未重建或重复上传该包。

Wry 的 `c8bb6e63fd696160609d7795bbb32566e8ed6086` 已推送至用户 fork。2026-11-14 已确认用户新建的 ChinaOSProject/rust-float16 的 parent/source 均为正确的 **Alexhuszagh/float16**，并将已审阅的 `9028edc088ab4419536a5aa0b30cc227fcb5eb03` 原子推送至 debian10 和 loong64；远端引用精确核验通过。默认 main 保持 `6066be1922fb528a23aacb4da5880160378c758c`，本地 main 保留原构建基线；源码、补丁及实际源码包均未改动，未重跑测试。原先误配的 Go fork ChinaOSProject/float16 保持原样。本次没有向任何上游仓库推送。

LLVM 与 RustyV8 的版本线分别归档如下，不把不同构建输入或版本的修改混合：

| 仓库及版本 | Debian 分支 | LoongArch 分支 | 两分支精确提交 |
|---|---|---|---|
| [LLVM 21.1.8 已验收适配](https://github.com/ChinaOSProject/llvm-project/tree/loong64-llvm21) | `debian10-llvm21` | `loong64-llvm21` | `de0645087733b0320e759ba7886430ae2a95c3cd` |
| [LLVM23 的 V8 构建输入](https://github.com/ChinaOSProject/llvm-project/tree/loong64) | `debian10` | `loong64` | `20b6ec66967ac2a8f932863c1abf251e5b17a843` |
| [RustLLVM 23.1.1 输入](https://github.com/ChinaOSProject/llvm-project/tree/loong64-rustllvm23) | `debian10-rustllvm23` | `loong64-rustllvm23` | `1b9c0d5ff9bbe7634aead059efe6b11a7eeba145` |
| [RustyV8 152.2.0 输入](https://github.com/ChinaOSProject/rusty_v8/tree/loong64) | `debian10` | `loong64` | `350c5a859a9b3c84ab8701b3ad51e3b7699b30c0` |

LLVM21 的三个逻辑提交依次为 loader 检测 `070c87e0c0b1920eccd6b55737e5046c8bbdc5d7`、SOP/ABI/EH 兼容 `9ccd465481d8a934fabca6472aaf427dbb5b8474`、no-R CIE 修复 `de0645087733b0320e759ba7886430ae2a95c3cd`。最终14文件逐项匹配既有已验收源码；补丁内容未改，完整归档补丁的哈希保持 `1f10982e79f454adeaf21b007b1a6c3c52efed0fa91c2c6c6269e316f33e4ef6`。提交前仅从现存、哈希匹配的未修改文件恢复部分克隆所缺的原始 Git 对象。LLVM23 和 RustyV8 没有新增本地源码补丁，版本分支归档未修改的精确输入，外部构建配置未搬入源码仓库。此轮共向7个新 fork 发布18个分支，均已核对远端引用和默认分支未变。

## 保留项与验证边界

先前不存在或不可访问的源码 fork 现均已可访问，并已核对 parent/source。已完成推送的精确分支见上表及 LLVM/RustyV8 版本记录；float16 的目标 fork 问题已于 2026-11-14 解决。Wails 和 Rust 的源码审查要求与验证失败仍独立保留，用户创建 fork 不代表这些门禁已经通过。

2026-11-14 已补充 Wails 只读静态审查，未发现阻断性代码问题，且与原测试输入、补丁一致；覆盖边界保留。已核实 `taliesin-ai` 身份，并在专用缓存取得签名及哈希校验通过的官方 CodeRabbit 0.9.5。其 x64 包经现有 LATX 实际执行 `--plain` 返回 Bun 解析错误、退出码 7，未完成审查，故 Wails 源码仍未提交或推送。详见 Wails 正式报告的同日补充。

Rust 1.99.1 的只读审查已完成，源码、SDK 仍保留：现有 ABI 回归失败、选项未覆盖全部 ELF 生产路径，候选开发归档中仍有 `0x43` 代码对象，不能以最终链接文件的 `0x3` 代替验收；现有归档回归还漏查 `lib.rmeta-link`。详细审查为本地外部证据，不作为 Rust PR、issue 或贡献文档。Rust 对 LLM 原创贡献文本及 soundness-sensitive 实现的限制仍适用，后续须遵循该项目的人类处理流程和 `#llm-mentoring` 路径。

Codex 0.166.0 的真实匹配 helper 虽已构建成功，但尚缺最终对象/通知、成对包和运行验收，因此未把旧版 compatibility helper 或新未验收 helper 纳入本次发布。

ordinary16 的消费者夹具失败，Next35 仅完成组装，Next17 仅完成准备；Zstd 及其他工具草案也未进入本清单。已撤回的 GLib 单体、旧 GN/Ninja、旧编译器候选和 Git NO_CURL 包均排除。源码开发包只确认各报告所列 feature/consumer 范围，不表示已完成全部 548 个已观察 Rust crate 版本的依赖交付。

Magpie 修复包保留既有失败/成功回归及菜单路径验证记录。**本轮没有执行、启动、停止、重启或向 Magpie 发信号**；活动会话仍按原约束保留，修复后真实窗口状态转换的验收未做。软件仓库为项目指定 Git LFS 二进制仓库，此处不宣称另一个 APT 索引或签名发布已经完成。
