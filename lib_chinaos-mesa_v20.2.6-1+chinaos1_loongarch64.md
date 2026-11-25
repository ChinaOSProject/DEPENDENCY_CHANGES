# Mesa 20.2.6 · LoongArch64

Mesa 的上游仓库位于 <https://gitlab.freedesktop.org/mesa/mesa>。本次使用的 `mesa-20.2.6` 源码归档来自 GitHub 镜像 <https://github.com/chaotic-cx/mesa-mirror/tree/mesa-20.2.6>，归档 SHA-256 为 `a8c1d6923787e4b210d0ef423ea9b0cf3d62809fdc7710562179b58a0171bf2b`。

## 适配

银河麒麟 V10 SP1 自带 Mesa `20.0.8-0kylin3k26.4`。AMD Oland `1002:6611` 的 OpenGL 驱动在顶点、片元及计算着色阶段分别提供 8 个 SSBO 绑定。Blender 4.5.17 每阶段需要至少 12 个。以系统的 LLVM 8、libdrm 和 X11 为基座构建 Mesa 20.2.6 RadeonSI。Meson 配置选择 X11、GLVND/GLX、共享 LLVM 和 RadeonSI，链接 GLX 时显式纳入 `xcb-xfixes`。Mesa 源码保持上游标签内容。

## 分包

参考 Debian [`mesa` 打包源码](https://sources.debian.org/src/mesa/20.3.5-1/debian/control/)。分别交付 `chinaos-libgl1-mesa-dri`、`chinaos-libglapi-mesa`、`chinaos-libglx-mesa0` 和 `chinaos-mesa-common-dev`。运行库位于 `/usr/lib/blender/mesa`，RadeonSI DRI 模块位于该目录的 `dri` 子目录。开发头文件位于 `/usr/include/blender-mesa`，`blender-mesa-dri.pc` 提供独立名称。包名称和文件路径允许与系统 Mesa 一起安装。Blender 启动器在检测到旧 Mesa 20.0 和相应 GCN 1 PCI ID 时为该进程设置 DRI 搜索路径。

## 实机验证

在 Loongson 3A5000、麒麟 V10 SP1 和实际 X11 会话中，从 `chinaos-libgl1-mesa-dri` 提取的驱动报告 `AMD OLAND`、`Accelerated: yes` 和 `OpenGL 4.6 Mesa 20.2.6`。顶点、片元和计算阶段各提供 16 个 SSBO 绑定。`radeonsi_dri.so` 的最高 GLIBC 需求为 `GLIBC_2.28`。
