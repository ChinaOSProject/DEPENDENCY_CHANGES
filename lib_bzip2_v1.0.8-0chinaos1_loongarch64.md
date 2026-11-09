# bzip2 1.0.8 / LoongArch ABI 1.0

Codex v0.166.0 首轮最终链接中，bundled bzip2 的 C 代码调用 Rust `bz_internal_error`，产生 14 条超出 ±128 MiB 的 SOP B26 重定位。改用上游 `bzip2-sys` 原有的 pkg-config 动态库路径，避免该跨静态代码段调用；没有修改 bzip2、Rust crate、Codex 源码或 Codex Cargo.lock。

## 来源与版本

- 关联主项目 GitHub：https://github.com/openai/codex；适配仓库：https://github.com/ChinaOSProject/codex。
- bzip2 上游：https://sourceware.org/bzip2/。本次使用 Debian 原样收录的 upstream orig tar，未创建或声称存在独立 Git 源码补丁。
- [Debian 1.0.8 orig](https://deb.debian.org/debian/pool/main/b/bzip2/bzip2_1.0.8.orig.tar.gz)，810029 字节，SHA256 `ab5a03176ee106d3f0fa90e381da478ddae405918153cca248e682cd0c4a2269`。与下载的 Debian DSC 中 orig SHA256 一致；未独立验证 DSC 的 PGP 签名。
- [Debian 1.0.8-6 打包源码](https://deb.debian.org/debian/pool/main/b/bzip2/bzip2_1.0.8-6.debian.tar.bz2)，26991 字节，SHA256 `648ed0dac9a041ba6951e0cca521628aa39947cefee78139f7b934a5dc502095`。参考 control/rules、命令和文档分包，并原样采用其中的 bzexe 脚本及 man page。未应用该源码包的补丁集。
- 包版本 `1.0.8-0chinaos1` 标识本次打包，不代表 Debian `1.0.8-6` 修订。56 个原始上游文件的构建前后内容一致。

## Debian 分包与依赖

| 包 | 架构 | 内容 / 依赖 |
|---|---|---|
| libbz2-1.0 | loongarch64 | SONAME libbz2.so.1.0；依赖 libc6 >= 2.28，提供 ldconfig trigger |
| libbz2-dev | loongarch64 | 头文件、静态库、开发链接和 pkg-config/bzip2.pc；精确依赖同版本运行库及 libc6-dev 或 libc-dev；推荐 bzip2-doc |
| bzip2 | loongarch64 | 动态链接主命令、recover、grep/diff/more/bzexe 和 man pages；精确依赖同版本运行库及 libc6 >= 2.28 |
| bzip2-doc | all | 上游已生成 HTML/PDF/PS、CSS；无工具链或运行库依赖 |

遵循 Buster 基座的 `/bin` 命令和 `/lib/loongarch64-linux-gnu` 运行库布局。开发文件位于 `/usr/include`、`/usr/lib/loongarch64-linux-gnu`。未生成 Texinfo，也未创建无内容的额外分包。四包均包含完整 bzip2 许可证；命令包额外包含 bzexe 的 GPL-2 全文和版权。所有普通载荷文件均在 md5sums 中，归档为 root:root。

## 构建与验证

构建、打包、安装和运行测试全部在现有 `codex-v0.166.0-v8-directory-probe-20261107` 项目 Podman 容器内完成。宿主持 `/tmp/chinaos-heavy-build-20261105.lock`，最多 2 jobs，GCC 8.3 + SDK LLD；不定义 BZ_NO_STDIO，不修改全局宿主环境。全部源文件、目标和日志位于专属 ext4 缓存。

- 上游 make 测试通过；三个独立 ELF 均 machine 258 / flags 0x3，程序解释器 `/lib64/ld.so.1`，最高 GLIBC_2.27；静态库全部 7 个 ELF 成员为 flags 0x3。
- 四包整组安装成功，dpkg 状态/版本、apt-get check、40 个已安装普通文件和所有符号链接核验通过。原有 bzip2 与 libbz2 的精确版本依赖随整组升级得到满足。
- 实际共享/静态 C buffer API 与完整 stdio API 压缩解压通过，非法输入返回错误。主命令与 bunzip2 roundtrip、bzip2recover 恢复、Debian bzexe 包装后运行通过；坏数据 `bzip2 -t` 退出码 2。
- 独立 Rust 工程使用 bzip2 0.6.1 / bzip2-sys 0.1.13+1.0.8，六个依赖的版本及 checksum 与 Codex 锁定值一致。原版 build.rs 通过 pkg-config 输出动态 bz2 链接，实际 Rust wrapper roundtrip 和坏输入通过，DT_NEEDED 含 libbz2.so.1.0；测试 ELF flags 0x3、最高 GLIBC_2.28。
- 首次在完整 Codex 工作区内定点选择 bzip2 的探测，在编译前因离线缺少 alsa 0.11.0 失败，证据保留；改用独立测试工程，不更改主项目锁或补抓无关依赖。该小测试不等同于完整 Codex 产品运行验证。

## 产物

| 文件 | 字节 | SHA256 |
|---|---:|---|
| `binaries/libbz2-1.0/loongarch64/libbz2-1.0_v1.0.8-0chinaos1_loongarch64.deb` | 45044 | `cc40db565ee61039ebd710c7c5844ccda39ddfdf40fa141f85b9eb11fd0dccbb` |
| `binaries/libbz2-dev/loongarch64/libbz2-dev_v1.0.8-0chinaos1_loongarch64.deb` | 54900 | `abbe0d64877fbf66fad5780443a02d1fccefcb76b29d8cef3d909c72b63d95fb` |
| `binaries/bzip2/loongarch64/bzip2_v1.0.8-0chinaos1_loongarch64.deb` | 50916 | `ba857dd62a7d893e0cf85e85223a3aa313421b77d8db6e56d624beb2edf5cc18` |
| `binaries/bzip2-doc/all/bzip2-doc_v1.0.8-0chinaos1_all.deb` | 392276 | `b59353d8e574bf17e4003ec735a588622cae5370c7a6e038c24793929da10e09` |

## 本机证据

缓存根：`/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107`。

- 构建/整组安装/API：`logs/main-bzip2-build-package-20261108.log`、`cargo-home/bzip2-system-20261107/main-build-verification-20261108.json`。
- 独立 Rust FFI：`logs/main-bzip2-rust-dynamic-probe-20261107.json`、`logs/main-bzip2-dynamic-probe-20261107.log`。
- 主审直接归档校验：`/mnt/chinaos-cache/ChinaOSProject-cache/main-review-20261107/new-tools-main-payload-audit-20261108.json`。
