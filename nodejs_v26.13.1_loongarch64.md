# Node.js v26.13.1 · loongarch64

基座为银河麒麟 V10 SP1，Linux 5.4.18-168-generic，glibc 2.28。包架构采用本机 dpkg 的 `loongarch64`，Node.js `process.arch` 返回 `loong64`。

## 源码与构建

- 从 Node.js 官方发布源下载 `node-v26.13.1.tar.xz`，并按同一发布目录中的 SHA-256 清单校验。源码 SHA-256 为 `a7f6204ff63b6a2cd274d48903285abb86ed53a99070e532d4c2fd787457d9e6`。https://nodejs.org/dist/v26.13.1/
- 复用用户目录中已有的 GCC 13.4.0、旧世界汇编器适配及 Python 3.10.16。构建启用 small ICU，使用自带 zlib 及 OpenSSL 通用 C 实现，静态链接 GCC 和 C++ 运行库，并指定 ELF 解释器 `/lib64/ld.so.1`。
- 配置参数为 `./configure --prefix=/usr --with-intl=small-icu --openssl-no-asm --partly-static --without-npm --openssl-use-def-ca-store --v8-enable-temporal-support`。交付范围为 Node.js 运行时和开发头文件。

## 打包与验证

- `nodejs` 提供 `/usr/bin/node` 和手册，接替此前 ChinaOS 的 `node` 运行包。`libnode-dev` 提供 `/usr/include/node`，依赖相同版本的 `nodejs`。三个 LTS 版本使用相同安装路径，按版本择一安装。包名及开发文件分包参考 Debian 的 Node.js 22.x、24.x 打包源码。https://sources.debian.org/src/nodejs/
- 本次采用上游可执行文件构建，内部静态归档随可执行文件链接。两份交付包分别安装运行时和头文件。
- ELF 动态依赖为 `libatomic.so.1, libdl.so.2, libm.so.6, libpthread.so.0, libc.so.6, ld.so.1`，最高 GLIBC 符号版本为 `2.28`。运行包依赖为 `libc6 (>= 2.28), libatomic1`。
- 通过选定的上游断言、哈希、文件系统、ICU、流和压缩测试。隔离 dpkg 根目录安装两包后，通过版本和架构检查、JSON 转义及长字符串往返、gzip、worker 和 Node-API 扩展编译加载检查。验证范围为这些测试及 ABI 检查。

## 产物

- `nodejs/loongarch64/nodejs_v26.13.1_loongarch64.deb`，SHA-256 `8bc73e5e929495b191cf090dcfba1ee4d79d74de5e2b7d41b10a54bf2212b669`
- `libnode-dev/loongarch64/libnode-dev_v26.13.1_loongarch64.deb`，SHA-256 `0efcb35c200024543d9ab4a4f502479efecb7605b68bd6a68700fcf81ba3d638`

## Temporal

复用 Rust 和 Cargo 1.92.0，通过源码自带的 crate 离线静态构建 Temporal，并通过日期运算测试。

## V8 兼容改动

当 Highway 选择 SCALAR 时，V8 使用已有的标量字符串转义实现。SIMD 目标继续使用原有实现。标量回退沿用 ChinaOS 上一批 Node.js 移植。

字符串哈希的 Latin-1 检查在 SCALAR 目标上使用已有的逐字符检查。

```diff
--- a/deps/v8/src/json/json-stringifier.cc
+++ b/deps/v8/src/json/json-stringifier.cc
@@ -2315,10 +2315,12 @@
                                   size_t start, size_t uncopied_src_index,
                                   const DisallowGarbageCollection& no_gc);
 
+#if HWY_STATIC_TARGET != HWY_SCALAR
   template <typename SrcChar>
     requires(sizeof(SrcChar) == sizeof(uint8_t))
   V8_INLINE bool AppendStringSIMD(const SrcChar* chars, size_t length,
                                   const DisallowGarbageCollection& no_gc);
+#endif
 
   template <typename SrcChar>
     requires(sizeof(SrcChar) == sizeof(base::uc16))
@@ -3333,11 +3335,15 @@
 bool FastJsonStringifier<Char>::AppendString(
     const SrcChar* chars, size_t length,
     const DisallowGarbageCollection& no_gc) {
+#if HWY_STATIC_TARGET == HWY_SCALAR
+  return AppendStringScalar(chars, length, 0, 0, no_gc);
+#else
   constexpr int kUseSimdLengthThreshold = 32;
   if (length >= kUseSimdLengthThreshold) {
     return AppendStringSIMD(chars, length, no_gc);
   }
   return AppendStringSWAR(chars, length, 0, 0, no_gc);
+#endif
 }
 
 template <typename Char>
@@ -3386,6 +3392,7 @@
   return AppendStringScalar(chars, length, i, uncopied_src_index, no_gc);
 }
 
+#if HWY_STATIC_TARGET != HWY_SCALAR
 template <typename Char>
 template <typename SrcChar>
   requires(sizeof(SrcChar) == sizeof(uint8_t))
@@ -3439,6 +3446,7 @@
                           no_gc) ||
          needs_escaping;
 }
+#endif
 
 template <typename Char>
 template <typename SrcChar>
--- a/deps/v8/src/strings/string-hasher.cc
+++ b/deps/v8/src/strings/string-hasher.cc
@@ -76,13 +76,14 @@
 }
 
 bool IsOnly8BitSIMD(const uint16_t* chars, unsigned len) {
+  const uint16_t* end = chars + len;
+#if HWY_STATIC_TARGET != HWY_SCALAR
   namespace hw = hwy::HWY_NAMESPACE;
   hw::FixedTag<uint16_t, 8> tag;
   const size_t stride = hw::Lanes(tag);
   const auto high_byte_mask = hw::Set(tag, static_cast<uint16_t>(0xFF00));
   const auto zero = hw::Zero(tag);
 
-  const uint16_t* end = chars + len;
   while (chars + stride <= end) {
     const auto data = hw::LoadU(tag, chars);
     const auto high_bytes = hw::And(data, high_byte_mask);
@@ -92,6 +93,7 @@
     }
     chars += stride;
   }
+#endif
   // Handle remaining characters.
   while (chars < end) {
     if (*chars > 0xFF) {
```
