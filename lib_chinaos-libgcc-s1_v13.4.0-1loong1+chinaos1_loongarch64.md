# ChinaOS GCC 13 运行库包名迁移

上游仓库为 [gcc-mirror/gcc](https://github.com/gcc-mirror/gcc)。本次使用已有的 GCC 13.4.0 LoongArch 构建产物，包名从 `clash-libgcc-s1` 改为 `chinaos-libgcc-s1`。`libgcc-s1` 沿用 Debian 的运行库分包名称，`chinaos-` 标记该包是安装在项目私有目录中的发行包。基座发行版的 `libgcc1` 继续由系统仓库提供。原 GCC 源码与分包依据见 [GCC 13.4.0 适配报告](lib_gcc_v13.4.0_loongarch64.md)。

## 产物

| 字段 | 值 |
| --- | --- |
| 包名 | `chinaos-libgcc-s1` |
| 版本 | `13.4.0-1loong1+chinaos1` |
| 架构 | `loongarch64` |
| 产物 | `chinaos-libgcc-s1_v13.4.0-1loong1+chinaos1_loongarch64.deb` |
| 大小 | 418236 字节 |
| SHA-256 | `6bd0ea4189c09b020e32a9d654f565927c6813a20744247e24fb505eb78b5cfd` |

`/opt/clash-gcc13/lib/libgcc_s.so.1` 和 `/usr/lib/clash-verge/runtime/lib/libgcc_s.so.1` 在两个包之间逐字节相同，SHA-256 均为 `f489be66c0fde1124d4b0a1bc976e15e9f519d1a23e5fa01906f6b58c3eb4029`。它们保留原路径，供已发布程序继续加载。许可证随包安装在 `/usr/share/doc/chinaos-libgcc-s1/copyright`。重新生成的 `shlibs` 让后续针对该库的 Debian 包声明 `chinaos-libgcc-s1 (= 13.4.0-1loong1+chinaos1)`。

新包依赖 `libc6 (>= 2.28)`。它通过 `Provides: clash-libgcc-s1 (= 13.4.0-1loong1)` 供应已发布 Wine、LLVM 等包需要的旧名字及精确版本。`Conflicts` 和 `Replaces` 仅作用于持有同一组私有库文件的 `clash-libgcc-s1`。

## 安装和迁移

APT 索引更新后，已安装旧包的系统可显式切换一次。

```sh
sudo apt update
sudo apt install chinaos-libgcc-s1
```

离线可通过 `apt install ./chinaos-libgcc-s1_v13.4.0-1loong1+chinaos1_loongarch64.deb` 安装。需要按既有 SHA-256 清单复现旧交付时，可从 `ChinaOSProject/binaries` 提交 `6593d11b17f90a1b1331e6f4a81a91949f1bc27f` 读取旧 LFS 包。

## 验证

在两个独立、断网的 LoongArch Debian 10 基座容器里，用已发布的年度版启动器和 Wine 依赖组分别验证了全新安装与旧包迁移。全新安装组由新包直接满足 Wine 对旧名的精确依赖。迁移组由 APT 在一个事务中移除旧包并安装新包，Wine 保持安装。两组最终 `apt-get check`、`dpkg --audit` 和新包文件校验均通过，两份私有库的包文件归属已切换至新名称，系统 `libgcc1` 的 SHA-256 保持一致。
