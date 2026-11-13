# GCC 13.4.0：Clash 私有工具链与 oldworld 输出运行库默认值

日期：2026-11-12。上游 GitHub 仓库：`https://github.com/gcc-mirror/gcc`；官方上游：`https://gcc.gnu.org/`。实际 native 编译器为 GCC/G++ 13.4.0，target `loongarch64-unknown-linux-gnu`。最终三工具包 `clash-gcc-13 / clash-g++-13 / clash-cpp-13` 为 `13.4.0-3loong1`；其余八个既有 runtime/dev/doc 包保持 `13.4.0-1loong1`，版本差异是打包包装器修订，不是重编编译器。

## Debian 依据与私有交付

参考 Debian `gcc-13 13.4.0-1`：`https://snapshot.debian.org/package/gcc-13/13.4.0-1/`。准确源码打包输入与完整 copyright 位于 `references/`，本轮抽取 control/install 以及 `debian/control.m4`、`debian/rules.conf` 并记录 SHA256。准确版本的已生成 control 不构建所有默认 runtime 二进制；runtime 命名及 ABI 分工另由同源 control.m4 模板核对（libgcc-s1、libstdc++CXX_SO、libatomicATOMIC_SO、libitmITM_SO、libsspSSP_SO），不是声称当前 gcc-13 upload 列出的每个 runtime 都已发布。Debian 提供 gcc-13/cpp-13/g++-13、libgcc-13-dev、libstdc++-13-dev 与 runtime 分包；本组用 `clash-` 私有名称，GCC 开发对象、fixed/plugin headers 统一进 `clash-gcc-13-dev`（区别于 Debian 的 libgcc-13-dev/plugin-dev 细分），不创建只含公共标记的 gcc-base 空壳。

工具、启动对象/头文件/链接用开发库位于 `/opt/clash-gcc13`，`clash-libgcc-s1 / clash-libstdc++6 / clash-libatomic1 / clash-libitm1 / clash-libssp0` 在该 prefix 与 `/usr/lib/clash-verge/runtime/lib` 分别携带已审计的运行库副本。没有改写宿主公共 gcc、libstdc++、loader 配置或 alternatives。`clash-gcc-13-doc` 是实际构建生成的 man/info 子集，非完整 GCC 文档。已有 configure 记录只启用 C/C++，关闭 multilib/bootstrap/NLS、libsanitizer/libvtv/libgomp，不宣称交付其他语言或完整 GCC 全功能 SDK。

| 包 | 实际 control Depends |
|---|---|
| `clash-cpp-13` | `clash-gcc-13 (= 13.4.0-3loong1)` |
| `clash-g++-13` | `clash-gcc-13 (= 13.4.0-3loong1), clash-libstdc++-13-dev (= 13.4.0-1loong1)` |
| `clash-gcc-13` | `clash-gcc-13-dev (= 13.4.0-1loong1), clash-libgcc-s1 (= 13.4.0-1loong1), clash-libatomic1 (= 13.4.0-1loong1), clash-libitm1 (= 13.4.0-1loong1), clash-libssp0 (= 13.4.0-1loong1), binutils, libc6 (>= 2.28), clash-libstdc++6 (= 13.4.0-1loong1)` |
| `clash-gcc-13-dev` | `clash-libgcc-s1 (= 13.4.0-1loong1), clash-libatomic1 (= 13.4.0-1loong1), clash-libitm1 (= 13.4.0-1loong1), clash-libssp0 (= 13.4.0-1loong1), libc6-dev, clash-libstdc++6 (= 13.4.0-1loong1)` |
| `clash-gcc-13-doc` | `（无 Depends）` |
| `clash-libatomic1` | `libc6 (>= 2.28)` |
| `clash-libgcc-s1` | `libc6 (>= 2.28)` |
| `clash-libitm1` | `clash-libgcc-s1 (= 13.4.0-1loong1), libc6 (>= 2.28)` |
| `clash-libssp0` | `libc6 (>= 2.28)` |
| `clash-libstdc++-13-dev` | `clash-libstdc++6 (= 13.4.0-1loong1), clash-gcc-13-dev (= 13.4.0-1loong1), libc6-dev` |
| `clash-libstdc++6` | `clash-libgcc-s1 (= 13.4.0-1loong1), libc6 (>= 2.28)` |

三工具中 cpp 和 g++ 精确依赖本轮 `clash-gcc-13 (= 13.4.0-3loong1)`，GCC/g++ 对 runtime/dev 的锁定仍为已有的 `13.4.0-1loong1`。包装器使用 Essential 的 `/bin/bash`，使用系统已声明的 binutils 与 libc6-dev；安装检查还验证实际 assembler/linker 文件的包归属。

## 故障、最小修正与调用者控制

原 v4 工具未处理默认 newworld interpreter；v6 (`13.4.0-2loong1`) 增加有条件的 oldworld loader 默认值后，默认 C 已运行，但默认 C++ 会从基座 GCC8 的 `/lib/loongarch64-linux-gnu/libstdc++.so.6` 加载并因缺 `GLIBCXX_3.4.32` 失败。原始失败保留在 `logs/gcc-wrapper-v6-installcheck-02.log`，历史 v4/v6 三工具均不列入最终发布选择。旧 `gcc-wrapper-v6-review.json` 的草稿状态保留为历史。

v7 仅修改独立的打包 shell wrapper：当真实 driver preview 选择内建 LP64D newworld loader、有效库 sysroot 仅有 oldworld `/lib64/ld.so.1`、且为普通动态可执行链接时，沿用 v6 loader 默认；再仅对 native 根 sysroot `/`、没有调用者 rpath 且没有任何 `LD_RUN_PATH` 设置的调用添加新 dtags `RUNPATH=/opt/clash-gcc13/lib`。运行库路径由 wrapper 相邻 `../lib` 解析，不修改 driver、库、ELF flags、GCC 源码或源缓存。新增选项置于原始 argv 前，调用者后置的 dtags 选项仍能生效。

`-Wl,-rpath/--rpath/-R`（分开或等号/拼接形式）和 `-Xlinker -rpath` 保留调用者运行时路径；`-rpath-link` 只影响链接期，仍允许默认 runtime RUNPATH。`LD_RUN_PATH` 即使显式为空也抑制该默认。driver response 文件按 GCC/libiberty 的引号/空白/反斜杠规则检查，仍把原 argv 交还 GCC；解析失败、深度/数量超限或不透明 linker response 时保守绕过。显式 loader、target/specs、非 LP64D、compile-only、static/static-pie/shared/relocatable 模式保留原行为。非根的 oldworld sysroot 可继续得到 loader 默认，但不注入 native `/opt` runtime；modern/both/无 loader markers 不启用兼容默认。裸 driver `-R` 被该 GCC 拒绝，支持的 linker `-R` 通过 `-Wl` 传递，不把不支持的语法包装成新功能。

这不是跨 sysroot/现代 ABI 的全面 runtime 策略。调用者显式选择 loader/rpath/specs/环境时需自行保证兼容运行库；可执行文件部署到其他机器仍需安装相同前缀的相应 runtime 包。RUNPATH 保留标准 loader 的环境覆盖规则，不设置用户会话 LD_LIBRARY_PATH。

## v7 实际检查与可重现打包

独立脚本为 `tests/gcc-oldworld-loader-wrapper-v7.in`、`prepare-gcc-wrapper-v7.sh`、`gcc-wrapper-v7-dryrun-test.sh`、`gcc-wrapper-v7-installcheck.sh` 和 `audit-gcc-wrapper-v7.py`。它们保留在证据目录，v4/v6 脚本、包与失败日志均未覆盖。`logs/gcc-wrapper-v7-dryrun-03.log` 的所有 loader/runtime policy 用例通过，仅运行 driver `-###`，没有编译链接。前两次测试尝试的日志也保留：首次发现 driver 不接受直接 -R；第二次发现 GCC response preview 使用临时 @file、不能靠 collect2 展开的文本判断注入，随后改为检查最终 exec argv。

`logs/gcc-wrapper-v7-package-audit-01.json` 对比 v4/v7 的控制字段、普通文件 md5、所有 payload 字节/模式：仅 gcc/g++ 两个入口换为 wrapper，原 driver 重命名为 `.real` 且字节相同；所有其他文件不变。cpp 原始压缩 data.tar.xz 完全相同，SHA256 为 `3026425edf2fe7618e17418bf7f2333b14d798857199bd830358d5fe9dd3fe09`。没有重新编译工具链。

`logs/gcc-wrapper-v7-installcheck-01.log` 与续检 `-02.log` 合并记录在既有 `clash-gcc-wrapper-v6-installcheck` 容器中仅升级三工具，其余 36 个已接受包版本不动；`apt-get check` 和三个工具的 `dpkg --verify` 通过。共享 flock 下默认 C/C++ 编译、链接和直接运行成功；默认输出为 `/lib64/ld.so.1`、flags `0x3`、RUNPATH `/opt/clash-gcc13/lib`，ldd 将 libstdc++ 解析到该私有前缀。实际还检查 rpath/rpath-link、response rpath、LD_RUN_PATH/空值、dtags 覆盖、显式 loader、shared 和 relocatable 边界。导入 GLIBC/GLIBCXX/CXXABI 与容器及私有 runtime 的提供版本进行比较，具体数值为 C/C++ GLIBC 2.27（容器 2.28）、所需/提供 GLIBCXX 3.4.32、所需 CXXABI 1.3（提供 1.3.14），文件见安装日志和 `gcc-v7-installed-verification.json`。首次安装检查在所有编译/运行成功后，因显式空 LD_RUN_PATH 产生合法 `RUNPATH []` 被过严的“完全无 tag”断言拒绝；原失败日志及测试脚本保留。只修正测试断言为“不得注入私有路径”，`logs/gcc-wrapper-v7-installcheck-02.log` 在共享 flock 下继续核对已生成输出、真实 provider 和安装完整性，未重新编译或改动包。最终结论须结合两份日志，不能称 attempt01 整段通过。static/替代 sysroot/LP64F 等边界为 preview 验证，未声称实际交叉/静态程序完整运行。

## 最终产物

本来源的既有 ABI 审计覆盖 57 个 ELF，最高导入版本为 GLIBC 2.27、GLIBCXX 3.4.21、CXXABI 1.3.9；这是文件实际导入版本，control 可能为统一基座要求保留更保守的 libc6 >=2.28。按来源归类结果由原始逐文件审计派生，记录在 `archive-review-20261112/family-abi-summary-from-accepted-evidence.json`。

| 包 | 完整版本 | 架构 | 字节 | SHA256 |
|---|---|---|---:|---|
| [clash-cpp-13](../binaries/clash-cpp-13/loongarch64/clash-cpp-13_v13.4.0-3loong1_loongarch64.deb) | 13.4.0-3loong1 | loongarch64 | 2073592 | `90660d407d03288c0a053f8153c301503f98f06823ba572c2145c8916271126b` |
| [clash-g++-13](../binaries/clash-g++-13/loongarch64/clash-g++-13_v13.4.0-3loong1_loongarch64.deb) | 13.4.0-3loong1 | loongarch64 | 83627696 | `0024fec9ff7697555a6e2fa780ddc74df9004ef5a70c428703aaf551d7cef0f3` |
| [clash-gcc-13](../binaries/clash-gcc-13/loongarch64/clash-gcc-13_v13.4.0-3loong1_loongarch64.deb) | 13.4.0-3loong1 | loongarch64 | 218940892 | `5c160f9333e56edc49aa8503729f111aff19b3a75ccbaf172984b7d152cefa3b` |
| [clash-gcc-13-dev](../binaries/clash-gcc-13-dev/loongarch64/clash-gcc-13-dev_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 3116218 | `4d0e997319fba55236cb21c5a7246464cd5b1828c7b894ec10b0366cdd15fbef` |
| [clash-gcc-13-doc](../binaries/clash-gcc-13-doc/all/clash-gcc-13-doc_v13.4.0-1loong1_all.deb) | 13.4.0-1loong1 | all | 377496 | `3120f750a33527cb3980235b5feeb5c803db60628cf8947ae010ac53fcb8658f` |
| [clash-libatomic1](../binaries/clash-libatomic1/loongarch64/clash-libatomic1_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 63040 | `584725bebfc38abfe289e6a009f168ba3b47ed15cd94a3544ea7b8f75ae16bfb` |
| [clash-libgcc-s1](../binaries/clash-libgcc-s1/loongarch64/clash-libgcc-s1_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 417724 | `cfaa0db7941319e71eaac285c760c821287d658caace21b0d655a60b9f161662` |
| [clash-libitm1](../binaries/clash-libitm1/loongarch64/clash-libitm1_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 269572 | `da790cfbc9338de96790dc9e80bd3adc79d50dea2fad803479a4cc43761d5b1e` |
| [clash-libssp0](../binaries/clash-libssp0/loongarch64/clash-libssp0_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 50912 | `0dd26d236d72fa79258d84af13d87fd2e95ddcf3cfa44e108ffc3b95610b8779` |
| [clash-libstdc++-13-dev](../binaries/clash-libstdc++-13-dev/loongarch64/clash-libstdc++-13-dev_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 13637186 | `3d4570d420bfd45385ead85bbe817fb537929201452daf43f11a1004de9bac39` |
| [clash-libstdc++6](../binaries/clash-libstdc++6/loongarch64/clash-libstdc++6_v13.4.0-1loong1_loongarch64.deb) | 13.4.0-1loong1 | loongarch64 | 10172422 | `f7349371e9b0772394d318550adc0bcf40f035b5e16a375b2cc55d781b81b44d` |

## 共享归档与验证依据

完整交付索引为 [clash-native-family_v2.5.10_artifacts.json](clash-native-family_v2.5.10_artifacts.json)，可读索引为 [同名 Markdown](clash-native-family_v2.5.10_artifacts.md)。每个 `.deb` 使用完整 Debian Version 命名，旁附 `.deb.sha256`；Architecture:all 的数据/文档包放在 `all/`。目标旧世界发行版的实际 dpkg 架构名为 `loongarch64`，不同于当前 Debian sid 的 `loong64`。

主证据目录为 `/mnt/chinaos-cache/ChinaOSProject-cache/clash-verge-rev/v2.5.10/native-family`。本轮沿用已通过检查并由主 Agent 归档的 36 个 v4 包，没有重建、重拷或重跑整组/GUI 测试；仅 GCC 三个工具由独立 v7 候选替换。既有 `native-family-final-manifest-v2510.json` 对应历史 v2，不应作为最终哈希表。最终选择以本共享索引、`archive-review-20261112/archived-artifacts.json` 和 GCC v7 归档记录为准。

`logs/native-family-v4-archive-and-payload-audit.json` 检查了全部 39 个历史 v4 归档、文件 md5 与版权；`logs/native-family-v4-postinstall-integrity.json` 记录专用 Buster 容器实际安装、39 包版本匹配、2742 个普通文件 md5、`dpkg --verify` 和 `apt-get check` 通过。`logs/dt-needed-package-closure-v2510-v4-01.json` 对 133 个 ELF、723 条 DT_NEEDED 依赖边及实际包归属检查为 PASS：使用记录明确指定的私有运行库环境，不能解释成任意裸启动环境均自动解析私有库。

既有 `logs/family-v2-abi-audit.json` 的 133 个原生 ELF 均为 LoongArch、flags `0x3`；50 个程序使用 `/lib64/ld.so.1`，其余 83 个没有 PT_INTERP；整组最高导入 GLIBC 为 2.28、GLIBCXX 为 3.4.32。v4 的内容变化/不变范围另见其 manifest。所有包使用私有 `clash-*` 名称，无用于取代系统包的 `Provides/Conflicts/Breaks/Replaces`，没有覆盖宿主公共库、修改全局 ld.so 配置或全局环境。私有 SDK 的消费者应显式设置 `PKG_CONFIG_PATH=/usr/lib/clash-verge/runtime/lib/pkgconfig` 并选择相应加载路径。

本轮只在现有容器组装 GCC 包和进行有针对性的安装验证，所有实际编译/链接/运行均受宿主共享 flock 保护。未运行各上游完整测试集，未新增 GUI 验收，也未启动/停止/执行 Magpie。应用最终 GUI 验收与推送发布由主 Agent 负责；本执行 Agent 未 commit 或 push。
