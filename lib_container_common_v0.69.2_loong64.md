Dependency: go.podman.io/common
Version: v0.69.2
Type: 第三方 Go library 源码适配
Reason: 为 Linux LoongArch64 提供 seccomp 架构映射和默认 profile 支持。
Root Cause: seccomp package 未定义 LoongArch64 架构常量，缺少 Go、OCI runtime-spec、libseccomp 与本机架构映射，默认 profile 也未列出该架构。
Changes: 增加 LoongArch64 架构常量和转换映射，补充本机架构识别、默认 Go profile 与 seccomp.json，并增加映射测试。
Compatibility Impact: 为 LoongArch64 增加 seccomp 支持；其他架构的映射和默认规则保持原状。
Upstream Status: commit eafe72e8f154a31639c1a9d146a925c6d47ab380 已推送至 ChinaOSProject/container-libs 的 common-loongarch64 分支；尚未提交 upstream PR。
Validation: 在 LoongArch64、Kylin V10 SP1、Go 1.26.0 环境运行 go test -mod=vendor -tags seccomp ./pkg/seccomp，通过；gofmt 检查通过；seccomp.json 的 jq JSON 检查通过。
Project-side Changes: Podman 仓库无修改。
Repository:
https://github.com/ChinaOSProject/container-libs
