# fushi-server — Fushi 无头服务端发布仓

这里是 [Fushi / Hibiki](https://github.com/hajisensai/Fushi) 无头服务端 `fushi_server` 的
**发布仓**：只放 Release 包和这份说明。源码在 monorepo
[`packages/fushi_server`](https://github.com/hajisensai/Fushi/tree/develop/packages/fushi_server)
（与桌面 Fushi 共用同一份互联 host 引擎 `packages/fushi_engine`），issue 也请开到那边。

`fushi_server` 是一个无 GUI 的 CLI 进程，装在 NAS / 家用服务器 / VPS 上，手机和桌面 Fushi
通过「互联」配对后把这些活丢给它：

| 能力 | 说明 |
|---|---|
| 媒体库 host（视频 / 书 / 漫画 / 有声书 / 词典包） | 服务端扫描本机目录入库，客户端浏览、拉流、同步进度 |
| 漫画整卷 OCR | 客户端上传卷，服务端跑 ONNX 检测+识别，回传 mokuro |
| 字幕识别（ASR） | 客户端上传音轨或指定 host 视频，服务端转录成 SRT + token 时间轴 |
| 代下载 | 内置 libtorrent 引擎或外接 qBittorrent，落盘后自动入库 |
| 内容订阅 | 订阅在 host 上创建、周期检查（Nyaa / apibay / Knaben / Torznab）并投进下载管线 |
| WebUI / admin API | `http(s)://<host>:38780/`：状态、配对 PIN、库根、上传、任务、下载、模型、设置、日志 |

## 下载

到 [Releases](https://github.com/hajisensai/fushi-server/releases) 取：

| 资产 | 平台 |
|---|---|
| `fushi_server-<version>-<seq>-linux-x64.tar.gz` | Linux x86-64，glibc ≥ 2.35（Debian 12 / Ubuntu 22.04 及以后、主流 NAS） |
| `fushi_server-<version>-<seq>-windows-x64.zip` | Windows x86-64 |

- `v<version>-beta.<seq>`（prerelease）= 测试版；`v<version>` = 正式版（Latest）。
- 包里已含 sqlite3、内置 torrent 引擎（Linux 静态链 libtorrent，零运行库依赖）、
  onnxruntime 1.22.0 CPU 版。目标机**不需要**装 libtorrent / openssl。
- 运行期只需 `ffmpeg` / `ffprobe`（视频封面抽帧、ASR 音轨解码、下载后转封装）；不在 PATH
  时在配置里写 `ffmpeg:` / `ffprobe:` 路径。

## 快速开始

```bash
mkdir -p /opt/fushi_server && tar -C /opt -xzf fushi_server-*-linux-x64.tar.gz
cd /opt/fushi_server
bin/fushi_server init                # 生成 fushi_server.yaml（含随机 admin_token）
$EDITOR fushi_server.yaml            # 填 libraries[]
bin/fushi_server serve --scan        # 起服务 + 首次扫描
```

启动后终端打印互联端口、TLS 指纹、WebUI 地址。浏览器开 `https://<host>:38780/`（自签证书，
浏览器会警告一次），输入 `admin_token` 登录。手机/桌面 Fushi：设置 → 互联 → 添加设备 →
输入 `<host>:38765`，按服务端显示的 6 位 PIN 完成配对。

配置项、systemd 单元、CLI 子命令、admin API、分块上传协议、CUDA 加速——见包内 `README.md`
（与源码仓的 [`packages/fushi_server/README.md`](https://github.com/hajisensai/Fushi/blob/develop/packages/fushi_server/README.md) 同一份）。

## 这个仓怎么发包

[`.github/workflows/release.yml`](.github/workflows/release.yml) 是一条薄 `workflow_dispatch`
入口，它调用 hibiki 的可复用 workflow
[`release-server.yml`](https://github.com/hajisensai/Fushi/blob/develop/.github/workflows/release-server.yml)
（`workflow_call`）：在 Linux（ubuntu-22.04）和 Windows 上各构建一份、真进程冒烟（起
`serve`、断言内置 torrent 引擎与 onnxruntime 都真被加载）、然后用本仓的 `GITHUB_TOKEN`
发 Release。零 PAT、零 secret，源码只在 monorepo 一份。

```bash
gh workflow run release.yml -R hajisensai/fushi-server -f channel=beta   -f source_ref=develop
gh workflow run release.yml -R hajisensai/fushi-server -f channel=formal -f source_ref=develop
```

版本号取 hibiki 的 `packages/fushi_server/pubspec.yaml`，与 app 版本独立；`<seq>` 是 hibiki
的发布序号（与 app 同一条单调序列）。

## 许可

GPL-3.0，见 [LICENSE](LICENSE)；与源码仓一致。
