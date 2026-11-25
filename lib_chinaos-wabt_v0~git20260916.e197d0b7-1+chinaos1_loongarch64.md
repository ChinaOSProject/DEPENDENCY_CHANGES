# ChinaOS wasm2c LoongArch64

Firefox 160.0.1 构建生成的 wasm2c 来自 WABT `e197d0b7bd3fe426f5963ae540f4426a1d49e89f`。版本输出为 `Firefox-in-tree-version`，包版本使用 Firefox vendoring 元数据中的源码日期和提交 `0~git20260916.e197d0b7-1+chinaos1`。[WABT 上游提交](https://github.com/WebAssembly/wabt/commit/e197d0b7bd3fe426f5963ae540f4426a1d49e89f)

Debian 的 `wabt` 包包含整套 WebAssembly 工具和 wasm2c runtime。Firefox 构建只生成 wasm2c，因此 `chinaos-wabt` 提供 `/usr/bin/chinaos-wasm2c`，在 `/usr/share/chinaos-wabt` 提供匹配的 runtime 源码、头文件与 SIMDe。工具及所需 GCC 13 C++ 运行库放在 `/usr/lib/chinaos-wabt`，通过相对运行路径加载。[Debian WABT 分包](https://packages.debian.org/source/stable/wabt)

`chinaos-wabt_v0~git20260916.e197d0b7-1+chinaos1_loongarch64.deb` 的 SHA256 为 `9a9a368c2042b0e8b93d4c077d7121c553e830bcbb28b17b6eb31168a6fb5922`。

工具 ELF flags 为 `0x3`，解释器为 `/lib64/ld.so.1`，最高 GLIBC 导入为 `2.27`。在 glibc 2.28 容器安装后，成功将 SIMD Wasm 模块翻译为 C。使用包内 runtime 与 SIMDe 编译的程序读出预期值 42，越界读取触发 `WASM_RT_TRAP_OOB`。验证日志为 `logs/verify-firefox-components.log`。
