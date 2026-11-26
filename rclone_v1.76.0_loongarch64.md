# rclone 1.76.0 LoongArch 适配报告

- 源码为 [rclone/rclone `v1.76.0`](https://github.com/rclone/rclone/commit/e982ff36668f5130d94b4f70409babb3d964df42)。复用本地克隆，在该提交的分离工作树中编译，源码与 `go.mod`、`go.sum` 均无改动。
- 目标是 Kylin V10 SP1，内核 `5.4.18-168-generic`，glibc `2.28`。Debian 包采用本机 `dpkg` 架构名 `loongarch64`，Go 目标名为 `linux/loong64`。
- 原生构建复用了 [已验证的 Go 1.27.1 old-world SDK](https://github.com/loong64-abi1-0/golang/releases/tag/go1.27.1)。构建命令为 `CGO_ENABLED=0 GOOS=linux GOARCH=loong64 go build -p 4 -mod=readonly -trimpath -buildvcs=true -ldflags '-s -w -X github.com/rclone/rclone/fs.Version=v1.76.0' -o rclone .`。构建未生成需单独交付的库或工具链。
- ELF 为 LoongArch ELF64、LP64、flags `0x3`。静态链接，无动态解释器、`DT_NEEDED` 项或 GLIBC 符号版本。保留纯 Go `mount` 和 `mount2`，构建未启用依赖 CGO 的 `cmount`。

## Debian 包

打包参考 [Debian rclone 1.75.2-2 源码](https://deb.debian.org/debian/pool/main/r/rclone/rclone_1.75.2-2.debian.tar.xz)的 `debian/control`、`rclone.install`、`rclone.links`、`rclone.manpages` 和 `rclone.docs`。包名和版本为 `rclone`、`1.76.0`。包含 `/usr/bin/rclone`、`/usr/sbin/mount.rclone`、man page、Bash 补全、手册和版权许可。补全由本次构建的二进制生成。`Suggests: ca-certificates, fuse3` 分别对应公网 HTTPS 和 FUSE 功能。Debian 参考允许 `fuse | fuse3`，本包选择 `fuse3`，因为本版默认挂载后端调用 `fusermount3`。Debian 源码分包针对 Go 开发者，本次交付范围为运行包。

## 验证

- `go mod verify` 通过。Go SDK 和链接进二进制的 237 个 Go 模块的许可或声明随包交付。
- 本机通过 `backend/local`、`fs`、`fs/operations`、`fs/sync`、`vfs`、`cmd/mount`、`cmd/mount2`、`lib/rest` 八个测试包。测试使用本地数据，未使用云账号运行后端集成测试。
- 对原始二进制及安装后的二进制分别测试了版本、复制、校验、同步更新与删除，并通过 `mount` 和 `mount2` 读取真实 FUSE 挂载中的文本和二进制文件。
- 在独立 dpkg 根目录完成安装、校验和卸载。九个普通文件的包内 MD5 校验通过，`dpkg --audit` 无异常。麒麟的 `dpkg` 在非 root 安装期间输出共享内存权限提示，安装和卸载均以 0 退出。
- 随包提供的上游 man page 可用本机 `man` 渲染。系统自带 `groff` 对部分宽表格和特殊字符输出排版提示。
- 构建信息、ELF 检查和验证日志位于 `/mnt/chinaos-cache/ChinaOSProject-cache/rclone-v1.76.0/logs/`。

## 交付

- 文件 `rclone/loongarch64/rclone_v1.76.0_loongarch64.deb`
- 大小 24450268 字节
- SHA-256 `587070bceb73ac786f0203a4a3d78ea12518b08e9744c70835511a312eb4b16a`

已从 Git LFS 重新下载并核对 SHA-256。[已发布的 `.deb`](https://github.com/ChinaOSProject/binaries/blob/2b996095c03e21123088c2c3ba91af5ec15c6570/rclone/loongarch64/rclone_v1.76.0_loongarch64.deb)。
