# ChinaOS NSS 3.132 LoongArch64

Firefox 160.0.1 原生构建生成了 NSS 3.132 运行库、公开头文件、`certutil` 和 `pk12util`。使用 Firefox 固定发布源码中的 `security/nss`，由同一 Clang/LLVM 22 工具链构建。[Firefox 中的 NSS 源码](https://github.com/mozilla-firefox/firefox/tree/FIREFOX_160_0_1_RELEASE/security/nss)

参考 Debian 的 `libnss3`、`libnss3-dev` 和 `libnss3-tools` 分包。运行库和 Firefox 构建的 SQLite 后端处于 `/usr/lib/chinaos-nss`，保留 SONAME，以相对运行路径解析私有 NSPR 和 NSS。头文件位于 `/usr/include/chinaos-nss/nss`，开发者通过 `pkg-config chinaos-nss` 选择这套库。工具发布为 `/usr/bin/chinaos-certutil` 与 `/usr/bin/chinaos-pk12util`。分包覆盖 Firefox 构建实际生成的模块和工具，版本均为 `3.132-1+chinaos1`，架构为 `loongarch64`。[Debian NSS 分包](https://packages.debian.org/source/stable/nss)

| 软件包 | SHA256 |
| --- | --- |
| `chinaos-libnss3` | `ea78aa53e1ee095878afb99954d8f4f710c487ac602b5638f47b68e591a1347e` |
| `chinaos-libnss3-dev` | `4ad727926c5604904a6d77e6e2dea1d5c93396d9e55be7642e11761f00a30868` |
| `chinaos-libnss3-tools` | `ca3c704ab928551c1f6ca6da93f03e354582cf162400fc7e81ee461967fd1939` |

在 glibc 2.28 隔离容器中与系统 `libnss3 2:3.42.1-1+deb10u2.lnd.6` 及匹配的 ChinaOS NSPR 分包共同安装。通过开发包编译链接的探测程序调用 NSS 初始化和关闭，返回 `NSS 3.132`，加载路径指向私有库。工具实际创建测试证书数据库和自签名证书，完成 PKCS#12 导出与读取。ELF 均为 flags `0x3`，最高 GLIBC 导入为 `2.28`。验证日志为 `logs/verify-firefox-components.log`。
