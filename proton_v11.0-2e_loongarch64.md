# Proton 11.0-2e：原生 LoongArch64 构建障碍记录

检查日期：2026-10-28。状态：上游不接受原生 LoongArch 构建目标；未完成 Proton 编译，未生成或交付 `.deb`。用户已明确要求坚持原生龙芯版本，优先完成 Firefox，并记录 Proton 的移植障碍。

## 固定源码与目标

- 官方仓库：<https://github.com/ValveSoftware/Proton>。
- 检查标签：`proton-11.0-2e`，提交 `bc896217ffcd67f892f952b010f898fe7bd0f0a1`；通过官方 `git ls-remote --tags` 与本地浅克隆核对。
- 本地源码：`/home/laevatein/ChinaOSProject/proton`。仅克隆顶层仓库，未初始化子模块，未下载 Steam Runtime SDK。
- 该标签固定的 Wine 子模块提交：`e60892cc2398b0a83c332a2078c7b85e47bdb42b`，仓库 <https://github.com/ValveSoftware/wine>。
- 本机：Loongson-3A5000，`uname -m` 为 `loongarch64`；麒麟 V10 SP1，glibc 2.28，旧 ABI，原生 ELF flags `0x3`、解释器 `/lib64/ld.so.1`。

## 实际复现

配置检查在原生 LoongArch64 的 rootless Podman 容器中执行，未修改宿主系统或上游源码。检查镜像为 `localhost/proton-native-check:11.0-2e-buster-loongarch64`，从已有 Debian buster 旧 ABI 开发镜像复用。容器禁止网络，源码以只读方式挂载。

在源码挂载为 `/src` 的容器内，分别执行：

```sh
make --no-print-directory -f /src/Makefile.in SRCDIR=/src TARGET_ARCH=loongarch64 get-steamrt-image
make --no-print-directory -f /src/Makefile.in SRCDIR=/src TARGET_ARCH=loong64 get-steamrt-image
```

两条命令的退出码均为 `2`，原始输出分别为：

```text
/src/Makefile.in:34: *** Unknown TARGET_ARCH=loongarch64.  Stop.
/src/Makefile.in:34: *** Unknown TARGET_ARCH=loong64.  Stop.
```

此失败发生在 Makefile 的目标架构检查阶段，尚未选择或运行 SDK，更未开始编译 Wine。日志和临时检查脚本保存在 `/tmp/proton-native-check-20261028/`；上面的命令和固定源码提交可独立复现该结果。

## 具体缺口

1. [Proton 固定提交的 Makefile.in](https://github.com/ValveSoftware/Proton/blob/bc896217ffcd67f892f952b010f898fe7bd0f0a1/Makefile.in) 的 `TARGET_ARCH` 分支只接受 `x86_64` 和 `arm64`。容器内外均有架构检查，不能靠变更一个目标名称获得原生支持。
2. [该标签使用的 Wine configure.ac](https://github.com/ValveSoftware/wine/blob/e60892cc2398b0a83c332a2078c7b85e47bdb42b/configure.ac) 的 `HOST_ARCH` 识别只列出 `aarch64`、`arm`、`i386` 和 `x86_64`；PE 交叉编译目标白名单为 `i386`、`x86_64`、`arm`、`aarch64`、`arm64ec`，没有 LoongArch。
3. [Wine ntdll 的源文件列表](https://github.com/ValveSoftware/wine/blob/e60892cc2398b0a83c332a2078c7b85e47bdb42b/dlls/ntdll/Makefile.in) 没有 LoongArch 的信号与异常处理后端。PE/Unix 调用约定、线程上下文和异常展开等仍需实际移植；仅把 Makefile 的架构白名单扩展到 LoongArch 无法补齐这些实现。
4. Windows x86/x86_64 游戏的指令执行还需要与上述原生运行时衔接。系统已有 LATX 不代表上游 Proton 已具备原生 LoongArch 构建支持。按照用户本次选择，未将 LATX 加 x86_64 Proton 组合替代为原生交付。

## 校验与交付边界

下载并核对了上述固定 Wine 提交的两个完整源文件，SHA256 为：

```text
35478d21bce698fde9153694163022c2256b603075b20b05d82b61cf29e92248  configure.ac
b1ef9ef4281555adaccac8172e4357224f35da5fad789a2de6a516675f17a2e9  dlls/ntdll/Makefile.in
```

此记录证明所检查的官方 Proton 标签及其 Wine 依赖没有现成的原生 LoongArch 构建路径，不对所有社区移植作穷尽性结论。本次没有修改源码伪造架构支持，也没有将 x86_64 二进制、启动脚本或仅创建好的容器标记为原生 Proton 成品。后续若继续开发，需要将其作为独立的架构移植任务，并验证原生异常/线程处理、PE 互操作以及实际 Windows 程序运行。

## 麒麟 Wine 8 软件包复用核查

2026-10-28 按用户提供的系统仓库线索检查。候选为 `wine8=8.0.58`，实际包名 `wine8_8.0.58_all.deb`，53,296,790 字节，`Architecture: all`、`Multi-Arch: foreign`。通过当前系统 APT 下载，SHA256 `963924cc62dbbcf2954550cce7873143cb889f366434fa5502382e445242c9d5` 与仓库元数据一致。APT 将该 all 包列在 loongarch64 Packages 索引下，不能据此认定其内部机器码是 LoongArch。

解包后逐一检查普通文件的 ELF/PE 头，82 个 ELF 中 44 个为 EM_X86_64、38 个为 EM_386，没有 EM_LOONGARCH；1464 个 PE 为 702 个 AMD64 和 762 个 i386。仅有的压缩文件是文档 changelog，没有另藏原生引擎归档。

| 文件（相对 `/opt/kylin-wine/wine8`） | 实际架构 | PT_INTERP |
| --- | --- | --- |
| `bin/wine` | i386 ELF32 | `/run/wine/lib/ld-linux.so.2` |
| `bin/wine64` | x86_64 ELF64 | `/run/wine/lib64/ld-linux-x86-64.so.2` |
| `bin/wineserver` | x86_64 ELF64 | `/run/wine/lib64/ld-linux-x86-64.so.2` |
| `lib/wine/x86_64-unix/ntdll.so` | x86_64 ELF64 | 共享库无解释器 |

`kylin-wine-internal` 设置包内 PATH/WINEDLLPATH、加载 `x86_64.env` 后 `exec wine`；该环境脚本明确使用 `/run/wine` 的 i386/x86_64 库目录。没有执行安装脚本或启动任何 Wine 前缀。

宿主已安装 `wine-runtime=22.04-22k1`（all），提供只读挂载在 `/run/wine` 的 Ubuntu 22.04 x86 运行库镜像。对可读普通文件的扫描检出 1580 个 x86_64 ELF、940 个 i386 ELF、0 个 LoongArch ELF；权限不可读的 shadow/private/cache 等路径在证据中单独记录。镜像内 libc 为 x86 两架构的 2.35，不是宿主原生 glibc 2.28 的可替换库；镜像中没有安装 Wine/DXVK/vkd3d 包。

宿主实际具有 `lat=1.6.4-2`（loongarch64）；`latx-i386.real` 与 `latx-x86_64.real` 是 LoongArch ELF64、flags `0x3`、解释器 `/lib64/ld.so.1`、最高 GLIBC 需求 2.28。当前 i386/x86_64 binfmt_misc 规则指向对应 LATX wrapper，wrapper 再配置 `/run/wine` 的 x86 库路径。这解释了 x86 Wine 可以借助本机转译设施启动的设计，但本次未执行转译器或验证游戏兼容性。

当前本地 APT 索引中，Wine 8 要求 `inst-translation (>= 3.0.4)`，该依赖候选仅为 `3.0.0`，`apt-get --simulate install wine8=8.0.58` 实际退出 100，报告未满足依赖。此为当前索引的单独安装障碍，不作为所有远端仓库版本的结论；未更改宿主源或安装软件。

复用结论：该包的 Wine 主程序和 Unix 库均为 x86，不能直接作为原生 LoongArch Wine/Proton 后端。字体、配置和启动脚本可作架构无关资源或打包参考；已有 LATX 与 runtime 是单独外架构执行路径的材料，不补齐原生 Wine 的信号/异常/调用约定实现。包内没有可直接取用的 LoongArch 后端源码。后续若找到真正的原生社区实现，应另行核对具体源码和功能，不以本次二进制包检查否定所有社区移植。

原始包、`wine8-payload-inventory.json`、`apt-evidence.json`、`installed-runtime-summary.json`、`installed-runtime-evidence.txt`、`installed-runtime-elf-scan.json` 位于 `/tmp/wine8-reuse-check-20261028/`。此次仅做下载、静态解包、只读系统检查和模拟安装。
