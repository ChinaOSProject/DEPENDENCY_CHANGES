# cbindgen 0.29.4 LoongArch64 构建报告

## Firefox 160.0.1 构建与交付（2026-11-25）

- 以同一 crates.io 发行版和 `Cargo.lock` 重建。用 Clang 21 驱动及已有 LoongArch 旧 ABI 兼容 LLD 21 链接，保留展开表，未修改源码。
- Debian 包 `cbindgen`，版本 `0.29.4-1+chinaos1`，架构 `loongarch64`，依赖由 `dpkg-shlibdeps` 计算为 `libc6 (>= 2.28), libgcc1 (>= 1:4.2)`。包内含 `/usr/bin/cbindgen` 及许可证。
- 软件包名称和工具文件布局参考 Debian `rust-cbindgen` 源码包的 `cbindgen` 分包。本包使用 crates.io 的源码归档和旧基座的 `libgcc1`，架构名沿用目标系统的 `loongarch64`。[Debian 打包源码](https://packages.debian.org/source/sid/rust-cbindgen)
- 交付 `../binaries/cbindgen/loongarch64/cbindgen_v0.29.4-1+chinaos1_loongarch64.deb`，SHA256 `1df64a4a79994a47e62fa27b7d9fc30251898d3b7dbaaf1d2646636336b0bd8b`。
- 宿主解包执行返回 `cbindgen 0.29.4`。已与本次五个 Clang 分包和 LLVM 22 SDK 在独立 glibc 2.28 容器共同安装，并通过 `dpkg --verify`、`apt-get check` 与版本检查。Firefox 配置已选中该工具并生成 FFI 头。目标 ELF flags `0x3`，最高 GLIBC 导入 `2.28`。

## 2026-10-28 构建记录

- 仓库：<https://github.com/mozilla/cbindgen>；crates.io 固定版本 `0.29.4`，使用随发行包的 Cargo.lock（`--locked`）。
- 类型：Firefox 157.0.1 构建期 Rust/C++ 绑定生成工具；不作为最终浏览器的运行依赖或独立应用交付。
- 源码修改：无。通过 Cargo 官方注册表下载，全部在 Firefox 专用 Podman 容器内原生编译。
- 工具链：正式 Rust SDK `1.98.1-2`（Firefox 报告列明包来源及 SHA256）、GCC `13.4.0`；`cargo install cbindgen --version 0.29.4 --locked --root /work/tools/cbindgen -j 3`。
- 旧基座配置：独立构建命令的 `RUSTFLAGS` 为 `-C linker=/opt/firefox-tools/gcc13/bin/gcc -C link-arg=-Wl,--dynamic-linker=/lib64/ld.so.1`。GCC 默认的新 loader 路径导致初轮 Cargo build-script 无法执行；显式设置本机旧 loader 后完整编译通过。没有修改 Rust/GCC 源码，也没有覆盖 Firefox 本体的 Cargo linker wrapper。
- 实测：Cargo release 编译与安装成功，用时 1 分 59 秒；运行输出 `cbindgen 0.29.4`；`readelf` 确认 ELF64 LoongArch，flags `0x3`、loader `/lib64/ld.so.1`。
- 集成验证：Firefox `configure-03.log` 的 cbindgen 检查通过，后续已生成多项 Firefox FFI 头文件。Firefox 本体已成功完整链接、生成最终 Debian 包，并通过宿主 CLI/headless/GUI 与干净容器安装运行验证；绑定生成集成完成，详见 [Firefox 最终报告](firefox_v157.0.1_loongarch64.md)。
- 产物与日志：容器 `/work/tools/cbindgen/bin/cbindgen`、`/work/cbindgen-build-02.log`；宿主对应 `/tmp/firefox-build-20261028/`。未安装任何依赖到宿主全局环境。
