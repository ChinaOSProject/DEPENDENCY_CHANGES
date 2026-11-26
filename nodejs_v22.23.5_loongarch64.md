# Node.js v22.23.5 · loongarch64

基座为银河麒麟 V10 SP1，Linux 5.4.18-168-generic，glibc 2.28。包架构采用本机 dpkg 的 `loongarch64`，Node.js `process.arch` 返回 `loong64`。

## 源码与构建

- 从 Node.js 官方发布源下载 `node-v22.23.5.tar.xz`，并按同一发布目录中的 SHA-256 清单校验。源码 SHA-256 为 `6cce7c58bf058700c643944903d60a325e8c9ceede1f5c2c088b435f2ada091a`。https://nodejs.org/dist/v22.23.5/
- 复用用户目录中已有的 GCC 13.4.0、旧世界汇编器适配及 Python 3.10.16。构建启用 small ICU，使用自带 zlib 及 OpenSSL 通用 C 实现，静态链接 GCC 和 C++ 运行库，并指定 ELF 解释器 `/lib64/ld.so.1`。
- 配置参数为 `./configure --prefix=/usr --with-intl=small-icu --openssl-no-asm --partly-static --without-npm --without-corepack --openssl-use-def-ca-store`。交付范围为 Node.js 运行时和开发头文件。

## 打包与验证

- `nodejs` 提供 `/usr/bin/node` 和手册，接替此前 ChinaOS 的 `node` 运行包。`libnode-dev` 提供 `/usr/include/node`，依赖相同版本的 `nodejs`。三个 LTS 版本使用相同安装路径，按版本择一安装。包名及开发文件分包参考 Debian 的 Node.js 22.x、24.x 打包源码。https://sources.debian.org/src/nodejs/
- 本次采用上游可执行文件构建，内部静态归档随可执行文件链接。两份交付包分别安装运行时和头文件。
- ELF 动态依赖为 `libatomic.so.1, libdl.so.2, libm.so.6, libpthread.so.0, libc.so.6, ld.so.1`，最高 GLIBC 符号版本为 `2.28`。运行包依赖为 `libc6 (>= 2.28), libatomic1`。
- 通过选定的上游断言、哈希、文件系统、ICU、流和压缩测试。隔离 dpkg 根目录安装两包后，通过版本和架构检查、JSON 转义及长字符串往返、gzip、worker 和 Node-API 扩展编译加载检查。验证范围为这些测试及 ABI 检查。

## 产物

- `nodejs/loongarch64/nodejs_v22.23.5_loongarch64.deb`，SHA-256 `97f09912fece50d23e50ea478e8b5dbcfe312254bb3256c447e0990933bf32ec`
- `libnode-dev/loongarch64/libnode-dev_v22.23.5_loongarch64.deb`，SHA-256 `b5ea740de9c44aad261143507a4d306be6c82c7aa8e36e341ee6022a4a6393a2`
