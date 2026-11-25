# Blender 4.5.17 LTS · LoongArch64

目标设备为 Loongson 3A5000、银河麒麟桌面 V10 SP1、glibc 2.28 和 AMD Oland `1002:6611`，以 `amdgpu` 驱动运行。GCN 1 是显卡架构，Blender 本体编译目标是 LoongArch64。Blender 5.0 的[官方兼容性说明](https://developer.blender.org/docs/release_notes/5.0/#compatibility)将 AMD 显卡的最低要求提高到 GCN 4。4.5 LTS 是此设备的目标分支。构建时最新维护版为 [2026 年 11 月 24 日发布的 4.5.17](https://download.blender.org/source/)。

## 源码与构建

源码来自 Blender 的 [4.5.17 官方发布归档](https://download.blender.org/source/blender-4.5.17.tar.xz)，对应 [上游仓库](https://github.com/blender/blender/tree/v4.5.17)。校验了上游的 MD5 `8f104b7518f463c9444ef1193d52cdb4`，归档 SHA-256 为 `05ce0568b7a9d0d21b9a67ed877f2f8f3a6fa8fa7d97cd64ca462069e5872722`。

使用 GCC 14.3、CMake 和 Ninja 在龙架构宿主机上构建，复用现有 LoongArch64 依赖缓存。逐文件比对确认编译源码与官方归档内容一致。Grease Pencil 的构建配置采用麒麟的 `libtiff.so.5` 及对应的 `libtiff-dev` 头文件。链接器显式保留 `librt`，从中解析旧版 glibc 的 `shm_open`。构建启用 X11/OpenGL、Cycles CPU、OpenSubdiv、OpenVDB、FFmpeg 和 Python 3.13。Wayland、Vulkan、Cycles GPU 设备后端、Cycles OSL、Embree、Open Image Denoise、USD 及 OpenXR 在此构建中关闭。

## 分包及桌面集成

依据 Debian [`blender` 打包源码](https://sources.debian.org/src/blender/4.3.2%2Bdfsg-2/debian/control/)，交付 `blender` 和同版本的架构无关 `blender-data`。程序私有目录包含与构建 ABI 一致的 C++ 运行时及复用的应用库。FFmpeg 运行库由 `chinaos-` 包提供，并由相对 RUNPATH 载入。麒麟自带 Mesa 20.0.8 在 Oland 上的 SSBO 数不足，启动器为符合条件的显卡加载 `chinaos-libgl1-mesa-dri` 提供的私有 RadeonSI 驱动。

参照 `scratch3_3.31.1_loongarch64.deb`，安装系统应用菜单项并刷新桌面 MIME 缓存。菜单项提供「创建桌面快捷方式」，也可执行 `blender --create-desktop-shortcut`。该操作使用 `xdg-user-dir DESKTOP` 解析当前用户的桌面目录，并保留已存在的同名快捷方式。

旧版 `blender_v5.2.2_loongarch64.deb` 已通过二进制仓库提交 `77cad70fe7e14e7774cfc98413cd880dbf4baf65` 下架。

## 验收

- 从 `.deb` 提取后，通过 Blender 启动器在实际 X11 桌面打开工厂场景，执行视口绘制并输出 1920×994 截图。Blender 报告 `AMD OLAND` 和 `OpenGL 4.6 Mesa 20.2.6`。
- Python 测试验证了细分曲面修改器，并用 Cycles CPU 生成 128×128 PNG。
- 桌面测试在 `桌面` 目录生成可执行的 `Blender.desktop`，重复调用保留已有修改。
- 同时模拟安装 25 个交付包，APT 计划新增 25 个包，升级和删除均为 0。包内 158 个 ELF 文件为 LoongArch，最高 GLIBC 需求为 `GLIBC_2.28`。Blender 使用 `/lib64/ld.so.1`，通过包内相对路径加载私有 GCC 运行时和 `chinaos-` FFmpeg 库。

二进制产物见 [ChinaOSProject/binaries 提交 `8024dfd`](https://github.com/ChinaOSProject/binaries/commit/8024dfd54d849d589da893f5a105dda700322861)。
