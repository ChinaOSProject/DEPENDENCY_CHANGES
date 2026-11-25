# AOM 3.13.1 · LoongArch64

上游源码位于 <https://aomedia.googlesource.com/aom/>，对应标签也可在 [GitHub 源码镜像](https://github.com/arthenica/libaom/tree/v3.13.1)查阅。本次使用 Blender 依赖缓存中的 [AOM 3.13.1 发布归档](https://storage.googleapis.com/aom-releases/libaom-3.13.1.tar.gz)，SHA-256 为 `19e45a5a7192d690565229983dad900e76b513a02306c12053fb9a262cbeca7d`。

为 FFmpeg 7.1.4 复用已有的 LoongArch64 归档。参考 Debian [`aom` 分包](https://sources.debian.org/src/aom/3.12.1-1%2Bdeb13u1/debian/control/)，交付 SONAME `libaom.so.3` 的 `chinaos-libaom3` 运行库和提供匹配头文件、链接入口、静态归档及 `aom.pc` 的 `chinaos-libaom-dev`。两包使用 FFmpeg 私有目录，允许与麒麟自带 AOM 并存。版权及专利许可随包交付。

提取后的 FFmpeg 开发包已用共享库和静态归档分别完成链接和运行，两个测试程序均能找到 `libaom-av1` 编码器。
