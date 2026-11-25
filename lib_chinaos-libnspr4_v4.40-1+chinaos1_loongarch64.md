# ChinaOS NSPR 4.40 LoongArch64

Firefox 160.0.1 原生构建生成了 NSPR 4.40。使用 Firefox 固定发布源码中的 `nsprpub`，由同一 Clang/LLVM 22 工具链构建。[Firefox 中的 NSPR 源码](https://github.com/mozilla-firefox/firefox/tree/FIREFOX_160_0_1_RELEASE/nsprpub)

按 Debian 的 `libnspr4` 和 `libnspr4-dev` 分包。`chinaos-libnspr4` 将 `libnspr4.so`、`libplc4.so` 和 `libplds4.so` 安装在 `/usr/lib/chinaos-nss`。开发包在 `/usr/include/chinaos-nss/nspr` 提供头文件，使用 `pkg-config chinaos-nspr` 选择私有库和 Firefox 的 `NO_NSPR_10_SUPPORT` 编译定义。两个包均使用 `4.40-1+chinaos1`，架构为 `loongarch64`。[Debian NSPR 分包](https://packages.debian.org/source/stable/nspr)

| 软件包 | SHA256 |
| --- | --- |
| `chinaos-libnspr4` | `d248fa8019cca6744f6b70237670cbc193ccee649bcad93f7abfcebb3895f6a8` |
| `chinaos-libnspr4-dev` | `93eb0c758c03f1698ebd53142857089a96d3da7bdcaa95e0d47db411c9d32c01` |

运行库保留原 SONAME，三个 ELF 的 flags 均为 `0x3`，最高 GLIBC 导入为 `2.27`。在 glibc 2.28 隔离容器中，与发行版 `libnspr4 2:4.20-1.lnd.2` 及 ChinaOS NSS 整组共同安装。使用开发包编译链接的探测程序实际返回 `NSPR 4.40`，包管理检查通过。验证日志为构建缓存中的 `logs/verify-firefox-components.log`。
