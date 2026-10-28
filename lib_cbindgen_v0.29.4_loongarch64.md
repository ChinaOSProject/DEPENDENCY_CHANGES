# cbindgen 0.29.4 LoongArch64 构建报告

- 仓库：<https://github.com/mozilla/cbindgen>；crates.io 固定版本 `0.29.4`，使用随发行包的 Cargo.lock（`--locked`）。
- 类型：Firefox 157.0.1 构建期 Rust/C++ 绑定生成工具；不作为最终浏览器的运行依赖或独立应用交付。
- 源码修改：无。通过 Cargo 官方注册表下载，全部在 Firefox 专用 Podman 容器内原生编译。
- 工具链：正式 Rust SDK `1.98.1-2`（Firefox 报告列明包来源及 SHA256）、GCC `13.4.0`；`cargo install cbindgen --version 0.29.4 --locked --root /work/tools/cbindgen -j 3`。
- 旧基座配置：独立构建命令的 `RUSTFLAGS` 为 `-C linker=/opt/firefox-tools/gcc13/bin/gcc -C link-arg=-Wl,--dynamic-linker=/lib64/ld.so.1`。GCC 默认的新 loader 路径导致初轮 Cargo build-script 无法执行；显式设置本机旧 loader 后完整编译通过。没有修改 Rust/GCC 源码，也没有覆盖 Firefox 本体的 Cargo linker wrapper。
- 实测：Cargo release 编译与安装成功，用时 1 分 59 秒；运行输出 `cbindgen 0.29.4`；`readelf` 确认 ELF64 LoongArch，flags `0x3`、loader `/lib64/ld.so.1`。
- 集成验证：Firefox `configure-03.log` 的 cbindgen 检查通过，后续已生成多项 Firefox FFI 头文件。Firefox 本体已成功完整链接、生成最终 Debian 包，并通过宿主 CLI/headless/GUI 与干净容器安装运行验证；绑定生成集成完成，详见 [Firefox 最终报告](firefox_v157.0.1_loongarch64.md)。
- 产物与日志：容器 `/work/tools/cbindgen/bin/cbindgen`、`/work/cbindgen-build-02.log`；宿主对应 `/tmp/firefox-build-20261028/`。未安装任何依赖到宿主全局环境。
