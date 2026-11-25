# FFmpeg 7.1.4 · LoongArch64

源码来自 FFmpeg 官方 GitHub 镜像 <https://github.com/FFmpeg/FFmpeg/tree/n7.1.4>，归档 SHA-256 为 `e045ac47d8eec15397ad22e0a8712e8537fb3d601956217ccc74f5ba60e17654`。

## 适配

麒麟 V10 SP1 提供 FFmpeg 4.2.2。Blender 4.5.17 需要 FFmpeg 7 的头文件及运行 ABI。使用面向 glibc 2.28 的 GCC 14.3，禁用内联汇编和自动探测，启用共享库、静态库、GPL 和已选编解码器。复用现有的 LoongArch 编解码器归档和 AOM 3.13.1。FFmpeg 源码保持上游标签内容。

## 分包

参考 Debian [`ffmpeg` 7.1 打包源码](https://sources.debian.org/src/ffmpeg/7%3A7.1.5-0%2Bdeb13u1/debian/control/)。`chinaos-ffmpeg` 提供 `chinaos-ffmpeg` 和 `chinaos-ffprobe` 命令。运行库保留上游 SONAME，采用 `chinaos-libavcodec61` 等名字。各运行库和 `-dev` 包采用对应的 ABI 分包和原有 Debian epoch `7:`。私有库及 `pkg-config` 文件位于 `/usr/lib/chinaos-ffmpeg` 下，开发头文件位于 `/usr/include/chinaos-ffmpeg`。应用可显式设置 `PKG_CONFIG_PATH=/usr/lib/chinaos-ffmpeg/pkgconfig` 使用这些开发包。

`chinaos-libavcodec-dev` 的静态归档使用 FFmpeg 构建时所用的编解码器归档，包括 x264 ABI 164 和 x265 ABI 215。麒麟自带开发包中的对应 ABI 为 155 和 165。合并归档保留了静态使用能力，避免安装或替换系统编码器包。AOM 保持独立运行及开发分包，随包附带内嵌编解码器的许可证。

## 验证

使用提取出的开发包各编译并运行一个动态链接及静态链接程序，确认 H.264、H.265、AV1、VP9 和 Opus 编码器可用。使用 `chinaos-ffmpeg` 编码 H.264，`chinaos-ffprobe` 识别出 128×96 画面。包内 ELF 的最高 GLIBC 需求为 `GLIBC_2.28`，动态依赖均可解析。23 个前缀依赖包一同模拟安装时，APT 计划新增 23 个包，升级与删除均为 0。
