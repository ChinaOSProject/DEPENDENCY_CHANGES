# Git v2.43.0 / loongarch64 构建与交付报告

日期：2026-11-04。此报告对应 Magpie v0.1.2339 构建期间生成的可复用 Git，按 ChinaOSProject `AGENTS.md` 第 6 节交付为 Git 核心包和匹配的 man page 包。下列 2026-11-04 包组是最终版本；早期未分包候选已留在任务备份中，不用于发布。

## 最终交付包组

| Debian 包 | Debian Version | Architecture | 文件 | 字节 | SHA-256 |
| --- | --- | --- | --- | ---: | --- |
| `git` | `1:2.43.0-2` | `loongarch64` | `binaries/git/loongarch64/git_v2.43.0_loongarch64.deb` | 4,688,640 | `90cc1e4de09df21c062c9b7fdcddda7a01be95a33955cfc1cd13177ae9d1a8bd` |
| `git-man` | `1:2.43.0-2` | `all` | `binaries/git-man/all/git-man_v2.43.0_all.deb` | 1,144,940 | `e31490565fe3fe6d211b54c4977e264227bd2a2825024646fe181ba9effb46e0` |

`git` Depends 精确匹配 `git-man (= 1:2.43.0-2)`；`git-man` 含 Git 2.43.0 手册页和上游 COPYING。核心包保留已验证的原生 ELF，移除九个不受支持的 Perl/Python launcher，保留 Git 自身 COPYING。新包完整性报告为 `/tmp/magpie-20261031/logs/tool-group-package-verification.json`，整组安装核验为 `logs/tool-group-install-smoke.log`。

## Debian/Ubuntu 参考差异与文件归属

参考 Ubuntu Noble 同版本 Git 2.43.0 的 `git` / `git-man` 分包（打包源码记录见 `/tmp/magpie-20261031/packaging-audit-20261102.md` 与 `logs/reference-git-ubuntu.dsc`）。核心仍使用已验证的 `/usr/libexec/git-core`，而 Debian/Ubuntu 通常使用 `/usr/lib/git-core`；没有为路径差异重编 ELF。手册单独属于 `git-man`。上游因未启用 Perl/Python/Tcl 而生成的不可用 launcher 直接移除，不创建空的 `git-svn`、`git-email` 等包。

从 `git` 中移除的九个桩文件为：

```text
/usr/libexec/git-core/git-cvsserver
/usr/libexec/git-core/git-svn
/usr/libexec/git-core/git-send-email
/usr/libexec/git-core/git-p4
/usr/libexec/git-core/git-cvsimport
/usr/libexec/git-core/git-instaweb
/usr/libexec/git-core/git-cvsexportcommit
/usr/libexec/git-core/git-archimport
/usr/bin/git-cvsserver
```

文件归属：`git` 持有 Git 程序、remote helpers、daemon/scalar、模板、locale 和 core 文档；`git-man` 持有 `/usr/share/man/man1`、`man5`、`man7` 下的 man pages 及其 copyright。全部 10 个原生 ELF 仍归 `git`，其字节与此前验证的 HTTP(S) ELF 相同。

## 交付结果与来源

- 源码仓库：<https://github.com/git/git>
- 固定源码版本：`2.43.0`，实际运行 `git --version` 输出 `git version 2.43.0`。
- 源码归档：<https://www.kernel.org/pub/software/scm/git/git-2.43.0.tar.xz>
- 源码 SHA-256：`5446603e73d911781d259e565750dcd277a42836c8e392cac91cf137aa9b76ec`，已与 kernel.org 的 `sha256sums.asc` 对照一致。
- 交付路径：`binaries/git/loongarch64/git_v2.43.0_loongarch64.deb`
- 最终 Git core Debian Package / Version / Architecture：`git` / `1:2.43.0-2` / **`loongarch64`**。
- 核心包大小：**4,688,640 字节**；SHA-256：**`90cc1e4de09df21c062c9b7fdcddda7a01be95a33955cfc1cd13177ae9d1a8bd`**。
- man page 包 Debian Package / Version / Architecture：`git-man` / `1:2.43.0-2` / **`all`**；大小：**1,144,940 字节**；SHA-256：**`e31490565fe3fe6d211b54c4977e264227bd2a2825024646fe181ba9effb46e0`**。

最终 Depends 覆盖全部 10 个独立 ELF（包括 `git-imap-send` 直接使用的 OpenSSL），并依赖版本完全匹配的 `git-man`。HTTP/HTTPS smoke 所验证的 ELF 内容没有改变；九个不支持的 Perl/Python launcher 已从核心包删除。

## 构建隔离与源码

基座为 `cr.loongnix.cn/library/debian:buster-slim`，GCC 8.3、glibc 2.28、LoongArch old-world。编译、安装构建依赖、strip、封包、解包和包安装验证均在 Magpie 专用 rootless Podman 环境中完成，未在宿主安装 Git、SDK 或任何构建依赖，未修改宿主或 Podman 全局配置。

专用 VFS storage 和调用入口：

```text
/tmp/magpie-20261031/podman-root
/tmp/magpie-20261031/podman-run
/tmp/magpie-20261031/podman-magpie
```

相关专属镜像为 `localhost/magpie-test:v0.1.2339-buster-loongarch64` 和 `localhost/magpie-tools:v0.1.2339-buster-loongarch64`。源码归档解包到 `/tmp/magpie-20261031/tools/git-2.43.0`，在容器中为 `/work/tools/git-2.43.0`。

**Git 源码零修改，未 patch 上游或模块缓存。** 基座 Git 2.20 不支持 Magpie fixture 使用的 `git init -b main`，因此构建此版本。补齐容器已有软件源中的 `libcurl4-openssl-dev`、`libssl-dev` 和 gettext 后，使用 Git 原生构建设置启用 HTTP(S)：

```sh
make -j2 prefix=/usr NO_TCLTK=YesPlease \
  NO_PERL=YesPlease NO_PYTHON=YesPlease all
make prefix=/usr NO_TCLTK=YesPlease \
  NO_PERL=YesPlease NO_PYTHON=YesPlease \
  DESTDIR=/work/git-package-root install
```

最终 Git helper 目录为 **`/usr/libexec/git-core`**，安装后的 `git --exec-path` 已核对一致。包含 `git-remote-http`、`git-remote-https`、FTP/FTPS 别名、`git-http-backend`、daemon、scalar、模板和 locale；保留上游安装的核心命令与 hardlinks。Tk 前端和 Perl/Python 扩展功能未提供；man pages 由独立的 `git-man` 包提供，未据核心 Git 可用宣称这些可选前端可用。

包内附上游 COPYING、2.43.0 release notes 和 README.Debian。没有 maintainer scripts，不自动启动服务。

## 运行依赖与 ABI

最终 control 的 Depends：

```text
libc6 (>= 2.28), libcurl4 (>= 7.56.1), libexpat1 (>= 2.0.1),
libssl1.1 (>= 1.1.0), zlib1g (>= 1:1.2.2), git-man (= 1:2.43.0-2)
```

Recommends：`ca-certificates, patch, less, openssh-client`。HTTPS 信任根和 SSH transport 客户端由系统包提供；本包不内置 SSH 客户端。

对包内全部 **10 个独立 ELF 物理文件**检查：LoongArch ELF64 / LP64、flags `0x3`，动态加载器均为 **`/lib64/ld.so.1`**，各文件最高 GLIBC 符号版本均为 **`GLIBC_2.28`**。`dpkg-shlibdeps` 对全部 ELF 生成最终依赖声明；其 librt 未使用符号提示没有作为错误隐藏，也未为消除提示改动源码。

真实 Kylin V10 SP1 / kernel 5.4.18 / glibc 2.28 宿主执行暂存程序 `git --version` 成功。宿主 `ldd` 检查核心 Git 和 HTTP helper，库全部解析，没有 `not found`。没有在宿主安装本包；没有据此声称适用于 new-world ABI 或更旧 glibc。

## 验证结果与边界

已在专属验证容器实际 `dpkg -i` 安装工具包，并验证：

1. `git --version` 为精确 `2.43.0`，helper 安装位置正确，HTTP/HTTPS helper 和 HTTP backend 存在且可执行。
2. `git init -b main`、add、commit、bare clone、生成 HTTP 可读仓库成功。
3. 本地 HTTP fixture 的 `ls-remote` 与 clone 成功，克隆后文件内容逐字节核对正确。
4. 本地 HTTPS fixture 的 `ls-remote` 与 clone 成功；临时自签证书通过显式 `http.sslCAInfo` 信任，证书包含并校验 `127.0.0.1` 的 IP SAN，**没有使用 `sslVerify=false`**。
5. 新 Git 安装后，Magpie `TestGitSignInRefusedSaysSo` 定点复测通过（0.073s），消除了最初缺失 remote-http 的测试环境阻塞。

2026-11-02 追加真实宿主 transport smoke：容器准备独立 bare 仓库与临时 TLS 证书，宿主直接运行同一 staging Git 和 helpers，使用宿主 libcurl/TLS 共享库。HTTP、HTTPS 的 `ls-remote`、clone 和文件内容核对均通过；HTTPS 显式启用证书验证并使用 fixture CA/IP SAN。测试使用临时 HOME、loopback 随机端口和显式 staging `GIT_EXEC_PATH`，没有安装或替换宿主 Git，也没有访问真实凭据。

HTTPS 外网探测 `git -c http.version=HTTP/1.1 ls-remote https://github.com/git/git.git refs/tags/v2.43.0` 曾在 90 秒超时，保留原日志并回收该测试进程。因此 HTTPS 成功结论来自已校验证书的本地 TLS fixture，未把外网超时写成成功。

最终封包后在专属容器解包，对 Git core 的 **229 个文件**和 `git-man` 的 **196 个文件**逐项检查 md5，并与 staging 的文件集合、内容、类型和模式核对一致。8 包整组通过 apt 本地重装/升级；Git 与 Go 均处于已安装状态，`dpkg --audit` 干净。最终包使用 `dpkg-deb --root-owner-group -Zxz -z6`，文件属主为 root/root，`SOURCE_DATE_EPOCH=1793427542`。

未运行 Git 上游完整 test suite；未验证真实 SSH/FTP 远端、Git server 部署、Tk/Perl/Python 扩展或长期运行。Magpie 最后一轮完整 suite 仍有已在其报告列明的失败，不因本定点测试通过而改写为全绿。

## 证据与关联工具清单

辅助脚本和日志均位于仓库外 `/tmp/magpie-20261031`：

- 构建与安装：`logs/git-http-deps.log`、`logs/git-http-build.log`、`logs/git-http-layout.log`。
- 封包：`package-tools.sh`、`finalize-tool-metadata.py`、`logs/package-tools.log`、`logs/tool-metadata-finalization-final.log`。
- ABI：`logs/git-all-elf-abi.json`、`logs/git-elf-abi.txt`。
- 运行：`tools-smoke.py`、`logs/tools-smoke.json`、`logs/tools-verify-retry.log`、`logs/tools-host-smoke.log`；外网超时证据为 `logs/tools-verify.log`。
- 真实宿主 HTTP/HTTPS：`prepare-git-host-fixture.py`、`git-host-transport-smoke.py`、`logs/git-host-transport-smoke.json`、`logs/git-host-transport-smoke.log`。
- 最终包完整性：`verify-split-tool-group.py`、`logs/tool-group-package-verification.json`、`logs/tool-group-install-smoke.log`。

本次 Magpie 流程需单独交付的可复用软件为 **Git 2.43.0** 和 **Go 1.27.1 old-world SDK**；后者见 `golang-go_v1.27.1_loongarch64.md`。GCC/binutils/curl/gettext 等使用既有仓库软件包，没有在本次流程从源码生成新的独立工具链。moejs/sqlite 等为编入 Magpie 的库，没有本次自制的独立可运行软件。Git 上游 test fixture helper 只为源码测试服务，不另造无实际用途的软件包。初版缺 HTTP 的 Git 不单独发布。

Magpie 源码及依赖源码仍为零修改，没有新增源码 commit。本执行 Agent 只生成约定产物及报告，未触碰其他任务文件，未 push；由主 Agent 统一审核发布。
