# Scratch Desktop 3.32.0 本机 LoongArch ABI v0 适配报告

本次交付应用版本为 **3.32.0**，Debian 架构为设备实际返回的 **loongarch64**，源码架构分支为 `loong64`。该包内置 Loongnix Electron **31.7.7**，嵌入 Node **20.18.0**、Chromium **126.0.6478.234**。它是在本机 LoongArch 内核上的 Debian buster / glibc 2.28 Podman 容器中构建、安装并验证的 ABI v0 应用包。

源码仓库：[ChinaOSProject/scratch-desktop](https://github.com/ChinaOSProject/scratch-desktop)。实际上游：[scratchfoundation/scratch-desktop](https://github.com/scratchfoundation/scratch-desktop)，已通过远端 HEAD 核实默认分支为 `develop`。项目协作规范中的 `upstream/main` 在这里对应 `upstream/develop`；架构分支继承 `debian10`，不独立长期跟踪上游 `develop`。

基线 `origin/debian10` 为 `4c5e77a8d4dd46da621be03091bac4b7d99cc1fe`，包含 3.32.0 基础提交 `cd3cf845a2f1cc597ae00c0ffd8d210c77cd648c` 和 Linux chrome-sandbox 权限兼容提交。该 3.32.0 基础提交是所核实 `upstream/develop`（`67ab2dc05d1002081bb2f963366723f6b83f3ad6`）的祖先。基线 `package.json` 锁定 Electron 42.0.1、electron-builder 26.8.1、`@scratch/scratch-gui` 13.7.4-svg；该上游 develop 提交的 Electron 为 42.11.9。本次没有整体下调这些依赖版本。

最终修改分为三个独立逻辑提交：

| 分支传播 | 提交 | 必要修改 |
| --- | --- | --- |
| debian10 → loong64 | `3c103f0` | webpack 优先读取 Electron 分发目录的 version 文件；无该文件时使用 execFileSync 回退，允许无图形环境编译。 |
| debian10 → loong64 | `a4de74d2647d88ba877d4f18e28a408136c86add` | GUI 文件上传 HOC 在未选择 fileToUpload 时取消上传状态，但桌面仍负责实际初始文件的 VM 载入。完成回调仅分发官方接口实际返回的 action，避免已转为展示状态时 dispatch(undefined)。CLI 回归同时断言最终 SHOWING_WITHOUT_ID 且无未处理异常。 |
| 仅 loong64 | `2f6c444d05f568898211cfb373e0aaddbdd3bdf8` | 新增 build:loong64 / package-loongarch.js，以原版 asar 和 dpkg-deb 打包；校验 runtime ABI/glibc 与全部素材 MD5，安装 desktop、SVG 图标、MIME 关联和 4755 沙箱。 |

`debian10` 最终 HEAD 为 `a4de74d2647d88ba877d4f18e28a408136c86add`，`loong64` 最终 HEAD 为 `2f6c444d05f568898211cfb373e0aaddbdd3bdf8`。源码工作区干净，未提交临时脚本、容器文件、日志、下载素材或构建产物。主 Agent 已将这两个源码分支推送至 [ChinaOSProject/scratch-desktop](https://github.com/ChinaOSProject/scratch-desktop/tree/loong64)，并通过 SSH 核对远端提交。

Electron 42.0.1 的 [官方发行资产](https://github.com/electron/electron/releases/tag/v42.0.1)不包含 LoongArch 分发。调查的 [darkyzhou/electron-loong64](https://github.com/darkyzhou/electron-loong64)现代构建声明 glibc ≥ 2.38，不能作为本机 glibc 2.28 的兼容运行时。最终只为此架构选用已实际验证的 Electron 31.7.7；其他平台继续沿用 package.json 既有 Electron 配置。

运行时来源：[Loongnix Electron 下载说明](https://docs.loongnix.cn/electron/download/index.html)、[31.7.7 发布说明](https://docs.loongnix.cn/electron/Release_notes/list/Electron-v31.7.7-loongarch.html)、[实际官方 zip](https://ftp.loongnix.cn/electron/LoongArch/v31.7.7/electron-v31.7.7-linux-loong64.zip)、[官方 SHA256 清单](https://ftp.loongnix.cn/electron/LoongArch/v31.7.7/SHASUMS256.txt)。下载在专用容器内完成，151528692 字节，SHA256：

```text
e0c756ca8a66dde3bece6ad902f152f539365ce7442d6353871d7c54d1c0f47b
```

核验对象为最终包中所有 8 个 ELF，均为 ELF64 little-endian、LoongArch machine=258、LP64D `e_flags=0x3` / object ABI v0；ABI 由 e_flags 检查，未仅依据架构名称或 EI_ABIVERSION 判断。electron（包内重命名 scratch-desktop）的最高 GLIBC 符号版本为 2.28，其余 7 个为 2.27。对象包括 chrome-sandbox、chrome_crashpad_handler、libEGL.so、libGLESv2.so、libffmpeg.so、libvk_swiftshader.so、libvulkan.so.1。动态解释器为 `/lib64/ld.so.1`；在 buster 容器取消宿主导入的 LD_LIBRARY_PATH 后，全部 ldd 解析成功，无 missing library。

主 Agent 另外在实际 Kylin 宿主机只读核验同一运行时的 8 个 ELF：所有 ldd 退出 0，无 not found 或符号版本缺失。宿主机版本为 libc6=2.28-10.kylin.35k1.3、libgtk-3-0=3.24.23-1kylin2k18.17update4、libnss3=2:3.98-0kylin0.20.04.2k0.2、libgbm1=20.0.8-0kylin3k26.4。这项宿主机校验未安装或改动系统环境；图形和功能验证仍在下述容器 Xvfb 中完成。

electron-builder 26.8.1 的 [原版架构枚举代码](https://github.com/electron-userland/electron-builder/blob/electron-builder%4026.8.1/packages/builder-util/src/arch.ts)没有 loong64，实际以 Node process.arch=loong64 调用 archFromString 得到 `Unsupported arch loong64`。本体的架构专用 portable fallback 使用未经修改的 `@electron/asar` **3.4.1** 和容器内 dpkg-deb，因此本次无需修改 electron-builder、Electron 或 native npm 依赖源码，也不生成 lib_ 报告。直接引用的 asar 版本已存在于原锁文件，仅新增根 devDependency 声明。

构建安装依赖采用 `npm ci --ignore-scripts --no-audit --no-fund`，保持锁定版本；安装结果与锁文件逐项版本对比无差异。审查发现 esbuild 0.27.7 的 linux-loong64 ELF 为 ABI v1（0x43）；它不是本次 webpack 编译路径所需工具，未执行该 ELF，也未打入应用。lint 使用独立前缀安装的原版 `@unrs/resolver-binding-wasm32-wasi@1.11.1`，通过 `NAPI_RS_FORCE_WASI=1` 和 NODE_PATH 使用官方 WASI fallback，未修改 node_modules 源码。应用 asar 内生产依赖只有 source-map-support 及其 JS 依赖，没有 `.node` 原生模块。

媒体资源实际来源为 Scratch 官方 `https://scratch-assets.scratch.org/internalapi/asset/<md5.ext>/get/`。主 Agent 在同一专用容器用仓库外临时脚本获取清单内 **1328** 项，采用 8 路并发、超时重试、临时文件原子重命名和内容 MD5 核验，逐项记录 URL、大小、MD5 和 SHA256，失败数为 0。打包入口再次校验所有所需素材 MD5；最终安装包内全部 1328 文件再次核对 MD5、SHA256、大小和数量。没有使用 TurboWarp 代码或外架构二进制作为应用运行时。旧 cdn.assets.scratch.mit.edu 域名在本次网络环境连接失败；未为此修改源码默认资源主机。

隔离构建基座为 `cr.loongnix.cn/library/debian:buster-slim`，容器 libc6 为 2.28-10.lnd.38，dpkg 为 1.19.7.lnd.2+nmu1，binutils 为 2.31.1-23.lnd.vec.1。复用宿主现有用户级 Node 24.21.0 / npm 11.19.1，只读挂载至容器，应用运行不依赖该 SDK。项目镜像为 `localhost/scratch-desktop-build:3.32.0-buster-loongarch64`，镜像 ID `b1cc70ab6136d336545dca23b19ad705450f59aa6067d3ab1745a40d904f55a4`；专用容器为 `scratch-desktop-audit-20261003`。apt 安装、npm、编译、asar、dpkg-deb、安装验证均仅在这个项目容器内完成，未向宿主全局安装工具或依赖。

Podman 6.1.2 本机 runtime 未编译 seccomp，项目容器按授权使用 `--security-opt seccomp=unconfined`。Buildah 上下文 overlay userxattr 在 Linux 5.4 上失败，改以 Podman create/exec/commit 生成项目镜像。未修改宿主全局签名 policy，也未删除其他项目或容器。缓存、下载、测试脚本、日志、产物均放在 `/tmp/scratch-desktop-build-20261003`，容器中对应 `/work`。宿主默认 Codex 命令沙箱因 LoongArch seccomp 不支持而 panic，所需命令依审批策略执行。

最终构建在容器副本 `/work/project` 执行，实际可复用入口如下：

```sh
podman exec --workdir /work/project \
  -e CI=1 -e NODE_OPTIONS=--max-old-space-size=8192 \
  -e ELECTRON_OVERRIDE_DIST_PATH=/work/runtime/electron-31.7.7 \
  -e LOONGARCH_BUILD_DIR=/work/artifacts \
  scratch-desktop-audit-20261003 npm run build:loong64
```

renderer 和 main 均由 webpack 5.106.2 编译成功；最终 lint 退出 0，0 错误、37 条基线已有 JSDoc 警告。生产 staging 再次执行 `npm ci --omit=dev --ignore-scripts`，asar 装入编译产物和原版生产 JS 依赖，dpkg-deb 使用 `--root-owner-group -Zxz --build`。

验证通过实际安装后的 `/usr/bin/scratch-desktop` 完成。普通容器用户 scratch-test（uid 1000）使用 Xvfb 1440×900×24 和私有 D-Bus 会话启动应用，实际 main 确认 version=3.32.0、isPackaged=true、resourcesPath=/opt/scratch-desktop/resources、Electron=31.7.7。测试启动命令为：

```sh
runuser -u scratch-test -- env -u LD_LIBRARY_PATH TMPDIR=/tmp \
  xvfb-run -a -s '-screen 0 1440x900x24' dbus-launch --exit-with-session \
  /usr/bin/scratch-desktop --remote-debugging-port=9222 \
  --remote-debugging-address=127.0.0.1 --inspect=9230 \
  --user-data-dir=/work/tests/profile-installed-final
```

未使用 `--no-sandbox`、`--disable-gpu` 或显式软件渲染参数。Xvfb 没有物理 GPU，启动日志有 GPU 初始化失败后回退的信息；实际 WebGL renderer 为 ANGLE / Vulkan / **SwiftShader Device (LLVM 16.0.0)**，图形正常显示。D-Bus system bus 在容器中不存在，日志有相应提示。基线 webpack 配置已有说明的 fetch-worker 绝对 file:///chunks 路径缺失提示不影响已验证的本地媒体存储路径，本次未修改相关依赖。

两组既定核心回归结果：

| 验证 | 结果与方法 |
| --- | --- |
| 默认项目 / renderer | 真实 GUI 载入 Stage 和 Sprite1，2 个默认造型、默认声音和渲染 canvas 存在，保留截图。 |
| 应用网络阻断 / 本地媒体 | CDP Network.emulateNetworkConditions offline=true，禁用缓存，实际 HTTPS fetch 被阻断；通过 VM runtime.storage.load 分别读取 SVG、PNG、WAV，内容 MD5 正确，记录 fs.readFile 实际路径均位于 /opt/scratch-desktop/resources/static/fetched。 |
| Scratch VM 执行 | 向真实 GUI 的 VM 添加绿旗→改变 x 10 的脚本，运行后 Sprite1 x 从 0 到 10。 |
| 真实 GUI 保存 | 测试开始删除此前 scratch-core.sb3，点击 File → Save to your computer，main 的原 will-download / 文件移动流程创建全新 .sb3。仅通过调试器临时替代同步保存对话框的文件选择返回值，未改应用源码或包内容。 |
| .sb3 重载 / 执行 | VM 读取刚保存的 ZIP 项目，块、造型、声音保留，重新绿旗运行 x 从 10 到 20。 |
| 命令行文件初始载入 | 新普通用户进程由 /usr/bin/scratch-desktop /work/tests/scratch-core.sb3 启动，经 main 初始文件 IPC 装载，x=10 且脚本存在，再次运行 x=20；最终 Redux projectState.loadingState=SHOWING_WITHOUT_ID，loadingProject=false，无未处理 renderer 异常。 |

离线三类样本为 `001a2186db228fdd9bfbf3f15800bb63.svg`、`0015433a406a53f00b792424b823268c.png`、`0039635b1d6853face36581784558454.wav`。它们通过存储 API 实际读取包内文件，而非直接用测试脚本替代存储逻辑。

验证边界：这是本机 LoongArch / Linux 5.4.18-168-generic 上的 buster 容器、Xvfb 和软件渲染验证；未直接在 Kylin 的真实图形会话中验证，也未验证物理音频输出、摄像头、麦克风、硬件扩展、Scratch Link 外设或真实 GPU 加速。无效/损坏项目错误对话框、联网服务、其他现代平台的运行未纳入本次测试。现代平台依赖和原构建入口保持原配置。

正式包路径：`/home/laevatein/ChinaOSProject/binaries/scratch-desktop/loongarch64/scratch-desktop_v3.32.0_loongarch64.deb`。

| 字段 | 最终值 |
| --- | --- |
| Package / Version / Architecture | scratch-desktop / 3.32.0 / loongarch64 |
| 大小 | 215293884 字节 |
| SHA256 | b93108ce03cd0545b44db128a99b8068eb8e62ffc718e3b9fbab99e7a62bbd96 |
| 安装入口 | /usr/bin/scratch-desktop → /opt/scratch-desktop/scratch-desktop |
| 沙箱权限 | /opt/scratch-desktop/chrome-sandbox，root:root，4755 |
| 核验 | 1328/1328 素材；8/8 ELF ABI v0、GLIBC ≤ 2.28；无原生 Node addon；dpkg install 成功，dpkg --audit 无输出 |

控制文件声明 libc6 ≥ 2.28 及实际 GTK3、NSS、音频和 X11/DRM/GBM 等运行库依赖，完整 Depends 与归档所有权信息保存在证据中。desktop、原项目 SVG 图标、*.sb3 MIME 关联和项目许可文件随包安装。desktop-file-validate 退出 0，仅提示 Education/Development 两个主分类可能令菜单重复显示。

证据保留在 `/tmp/scratch-desktop-build-20261003`：`logs/build-final.log`、`logs/lint-final.log`、`logs/deb-install.log`、`logs/verify-installed-package.log`、`logs/app-functional-test.log`、`logs/cli-load-test.log`，以及 `audit/media-assets-verified.json`、`audit/installed-package-verification.json`、`audit/app-functional-test.json`、`audit/cli-load-test.json`、`audit/electron31-elf.json`、`audit/deb-info.txt`、`audit/deb-contents.txt` 和截图。测试应用已退出；主 Agent 重启本次指定的 `scratch-desktop-audit-20261003` 容器后，确认只剩 PID 1 的 `sleep infinity`。构建文件、镜像、容器与产物保留供复核。

安装包已通过 Git LFS 上传至 [ChinaOSProject/binaries](https://github.com/ChinaOSProject/binaries/blob/70b8c2123310ca05bd30a972de2196b9a86c1ddd/scratch-desktop/loongarch64/scratch-desktop_v3.32.0_loongarch64.deb)，归档提交为 `70b8c2123310ca05bd30a972de2196b9a86c1ddd`。LFS 上传成功，指针中的 SHA256 和大小与上表实际文件一致；此报告归档到 [ChinaOSProject/DEPENDENCY_CHANGES](https://github.com/ChinaOSProject/DEPENDENCY_CHANGES/blob/main/scratch-desktop_v3.32.0_loongarch64.md)。
