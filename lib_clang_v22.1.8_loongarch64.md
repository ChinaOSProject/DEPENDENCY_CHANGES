# Clang 22.1.8 原生 LoongArch64 ABI v0 依赖构建

## 来源与目的

- 依赖仓库：https://github.com/llvm/llvm-project
- 实际源码：Loongnix `llvm-project_22.1.8-1.src.tar.gz`
- 下载地址：https://ftp.loongnix.cn/toolchain/llvm/llvm22/llvm-project_22.1.8-1.src.tar.gz
- 版本：Clang 22.1.8；Loongnix 源码包修订 `22.1.8-1`。
- 源码归档 SHA-256：`9acdcc3531fe7641bc9455a644508b03d3facecbfb4a92ebfde7f01df1e5b278`，本次重新计算一致。
- 原因：Firefox 原生 LoongArch 构建需要 LSX。现有 GCC 13 不接受 `-mlsx`；另一个 Clang 21 工具虽然能在本机运行，但产生 ABI v1 对象，不能与目标系统 ABI v0 的库及正式 LLD 22 混链。

本次没有修改 Clang 或 LLVM 源码。只从上述完整归档的源码树复制 `clang/`、`cmake/`，并对复制结果执行递归比较，两目录均与原源码一致。未修改 ELF 标志位，未忽略链接器 ABI 检查，未丢弃异常展开信息。

## 隔离构建与复用

所有配置、编译、链接、安装和运行验证均在 Firefox 专用 Podman 容器 `firefox-build-20261028` 中完成。基础镜像为 `localhost/firefox-build:157.0.1-buster-loongarch64-online`，镜像 ID `8bcc5b1f391af65b4d0bc56bba2a3bc8e397594be0f39e0ef43435cd83cc4c07`。没有向宿主系统安装编译器、依赖库或修改全局环境。

- 本机：Loongson 3A5000 / LoongArch64，目标系统 glibc 2.28、ELF ABI v0、动态加载器 `/lib64/ld.so.1`。
- CMake：容器已有的原生 `/opt/cmake-3.27.9/bin/cmake`。
- 构建编译器：隔离挂载的 `/opt/firefox-tools/gcc13/bin/gcc`、`g++`，版本 13.4.0。
- 复用正式 LLVM 22.1.8 SDK：`/work/tools/llvm22-v0/usr/lib/llvm-22`，包括静态 LLVM 库、头文件、CMake package 和 `llvm-tblgen`。
- SDK 的 `llvm-config --has-rtti` 为 `NO`，Clang 使用相同设置；没有重新编译 LLVM 后端，也没有借用 ABI v1 后端。
- 源码：`/work/clang22-source/{clang,cmake}`。
- 构建目录：`/work/clang22-v0-20261028/build`。
- 安装前缀：`/work/clang22-v0-20261028/install`。
- 宿主对应工作目录：`/tmp/firefox-build-20261028/clang22-v0-20261028`。

工具执行作用域需要 `LD_LIBRARY_PATH=/opt/firefox-tools/gcc13/lib`。系统原有 libstdc++ 版本不足以执行用 GCC 13 构建的工具；该变量只作用于相应容器构建进程。

关键配置：

```sh
cmake -S /work/clang22-source/clang \
  -B /work/clang22-v0-20261028/build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/work/clang22-v0-20261028/install \
  -DCMAKE_C_COMPILER=/opt/firefox-tools/gcc13/bin/gcc \
  -DCMAKE_CXX_COMPILER=/opt/firefox-tools/gcc13/bin/g++ \
  -DCMAKE_C_FLAGS_RELEASE='-O2 -DNDEBUG' \
  -DCMAKE_CXX_FLAGS_RELEASE='-O2 -DNDEBUG' \
  -DCMAKE_EXE_LINKER_FLAGS='-Wl,--dynamic-linker=/lib64/ld.so.1' \
  -DLLVM_DIR=/work/tools/llvm22-v0/usr/lib/llvm-22/lib/cmake/llvm \
  -DLLVM_CONFIG=/work/tools/llvm22-v0/usr/lib/llvm-22/bin/llvm-config \
  -DLLVM_TABLEGEN_EXE=/work/tools/llvm22-v0/usr/lib/llvm-22/bin/llvm-tblgen \
  -DLLVM_ENABLE_RTTI=OFF \
  -DLLVM_INCLUDE_TESTS=OFF -DCLANG_INCLUDE_TESTS=OFF \
  -DCLANG_ENABLE_STATIC_ANALYZER=OFF -DCLANG_ENABLE_ARCMT=OFF \
  -DCLANG_ENABLE_OBJC_REWRITER=OFF -DCLANG_ENABLE_CIR=OFF \
  -DLLVM_LINK_LLVM_DYLIB=OFF -DCLANG_LINK_CLANG_DYLIB=OFF \
  -DCLANG_BUILD_TOOLS=ON -DCLANG_BUILD_EXAMPLES=OFF \
  -DLLVM_USE_SYMLINKS=ON \
  -DLLVM_PARALLEL_LINK_JOBS=1
ninja -C /work/clang22-v0-20261028/build -j3 \
  clang libclang clang-resource-headers
ninja -C /work/clang22-v0-20261028/build -j3 \
  install-clang install-libclang install-clang-resource-headers \
  install-libclang-headers
```

仅选择 Firefox 需要的编译器、libclang 及资源头目标，不构建额外 Clang 工具、测试套件、静态分析器或 compiler-rt。Clang 工具自身沿用 LLVM SDK 的无 RTTI、无异常构建选项；这不禁用它为被编译的应用生成 C++ RTTI 或异常展开代码。

## 验证

构建于 2026-10-28（Asia/Shanghai）完成。主构建从 `2026-10-27 19:19:34 UTC` 到 `20:09:18 UTC`，1206 个步骤全部成功；随后安装和验证均以退出码 0 完成。验证脚本为工作目录中的 `verify.sh`，详细输出在 `verify.log`。

1. **后端预检**：正式 SDK 的 `llc` 将最小 LoongArch IR 生成为 `flags 0x3` 对象；使用旧加载器链接后在本机运行成功。
2. **工具版本与资源头**：安装的 `clang`、`clang++` 均报告 `22.1.8`、默认目标 `loongarch64-unknown-linux-gnu`；`-print-resource-dir` 指向安装前缀内的 `lib/clang/22`，其中 LSX 和 Wasm SIMD 头文件均存在。
3. **C11 线程与原子操作**：新 Clang 编译并通过正式 LLD 22 链接的程序在本机输出 `c11-thread-atomic 42`，退出码 0。
4. **C++17、LSX、线程、异常展开及 RTTI**：程序通过 `std::thread` 执行来自运行时输入的 LSX 向量相加，跨函数抛出并捕获异常，确认栈对象析构恰好执行一次；同时测试 `dynamic_cast` 的匹配与不匹配两条路径。输入 21、22 分别输出 `cxx17-lsx-unwind 42 1 1` 和 `cxx17-lsx-unwind 44 1 0`，均退出 0。
5. **真实指令与展开表**：LLVM 22 反汇编明确包含两条 `vld` 和一条 `vadd.w`；最终程序的 `.eh_frame` 保有 FDE，异常析构测试实际通过。没有通过禁用 LSX、异常、RTTI 或展开表绕过问题。
6. **libclang C API**：使用安装的 `clang-c/Index.h`、新 `libclang.so` 编译并运行解析程序；对未保存 C 源码建立翻译单元并遍历函数声明，输出 `clang version 22.1.8; parsed-functions=1`，没有错误诊断，退出 0。
7. **Wasm 资源头**：新 Clang 使用 `wasm_simd128.h`、`--target=wasm32-unknown-unknown -msimd128 -ffreestanding` 成功生成合法 Wasm 对象。该检查没有增加或替换 Firefox 原有的 WASI sysroot。
8. **ELF 与 ABI floor**：直接读取 ELF 头并用 `readelf` 交叉核对，Clang、libclang、C/C++ 探测对象与三个探测程序的 `e_flags` 全为真实的 `0x3`；所有本机可执行程序的解释器均为 `/lib64/ld.so.1`。Clang 与 libclang 引用的最高 GLIBC 符号版本均为 `GLIBC_2.27`，低于目标系统 2.28；在作用域内使用 GCC 13 运行库时，`ldd` 没有缺失依赖。

本机程序统一显式使用：

```sh
--gcc-toolchain=/opt/firefox-tools/gcc13
--ld-path=/work/tools/llvm22-v0/usr/lib/llvm-22/bin/ld.lld
-L/opt/firefox-tools/gcc13/lib
-Wl,--dynamic-linker=/lib64/ld.so.1
```

线程程序另加 `-pthread`；C++ LSX 程序加 `-mlsx`。没有混用其他 Clang 版本的后端或本机对象。

最终安装约 373 MiB，`clang`、`clang++` 等别名使用符号链接。关键产物 SHA-256：

- `bin/clang-22`：`ccb7d67e23abb906d8d17388caa69090e384cf9fb31ccf0dc6dca04834b147e4`
- `lib/libclang.so.22.1.8`：`36a031e4110df99cd8320ae9c08df1d0e8dc118fdefdde3ad8cfa1901f555b1b`

## 日志与交付边界

配置、构建和验证日志保存在上述独立工作目录内：`configure.log`、`build.log`、`build-start.txt`、`build-end.txt`、`source-compare.log`、`install.log`、`verify.log`、`verify.sh` 及 `probes/`。反汇编与展开表单独保存在 `probes/cxx17-lsx-unwind.disassembly.txt`、`probes/cxx17-lsx-unwind.frames.txt`。

这是用于 Firefox 构建的编译器/SDK 依赖，按项目规则不额外创建无用途的 `.deb` 应用包。Firefox 的新构建必须使用独立对象目录，不能混入此前 GCC 或 ABI v1 Clang 的对象。最终 Firefox 软件包的系统 ABI、运行库私有携带和浏览器功能验证由 Firefox 构建报告记录。
