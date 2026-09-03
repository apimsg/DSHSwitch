# DSHSwitch

> Windows 上的一键开关，用于管理 DeepSeek `dsh`（@deepseek-ai/dsh）的 Web 服务进程。

一个托盘常驻的小工具：点一下大开关即可启动/停止 DSH Web 服务，并自动打开网页，同时提供命令行接口（CLI）供脚本/自动化使用。

## 功能特性

- **一键开关**：绿色 = 运行中，灰色 = 已停止，点击即可切换。
- **托盘常驻**：关闭窗口缩到托盘，服务不受影响；双击图标恢复主界面。
- **自动探测**：自动定位 `node.exe` 与 `@deepseek-ai/dsh` 的入口（`lib/bin.js`），无需手动配置。
- **防误杀**：停止时只在确认是本软件启动的进程时才终止，端口被外部进程占用时默认跳过，避免误伤。
- **命令行模式**：`--cli status|start|stop|restart|doctor` 供调试/自动化/冒烟测试。
- **自包含单文件**：发布为单文件 exe，目标机器无需安装 .NET 运行时。
- **界面 UI**：手绘开关、状态文字、地址栏，支持高 DPI 自动缩放。

## 环境要求

- 已安装 **Node.js**（`node.exe` 需在 `PATH` 中）。
- 已安装 **@deepseek-ai/dsh**（通过 `npx` 或全局安装）。
- 构建本工具需要 **.NET 8 SDK**（Windows）。

## 安装

### 从源码构建

在项目目录运行：

```cmd
dotnet publish DSHSwitch.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -o dist
```

构建产物在 `dist\DSHSwitch.exe`，可直接复制到任意 Windows x64 机器运行。

### 直接使用

双击 `DSHSwitch.exe` 启动，点大开关即可启动/停止 DSH Web 服务。

## 使用方法

### 图形界面（GUI）

1. 启动程序，默认从主界面显示。
2. 点击**大开关**打开绿色 = 启动服务。
3. 启动成功后会自动打开网页（可在 `settings.json` 里关闭）。
4. 点窗口右上角 ✕ 或“最小化到托盘”会收进托盘，服务不受影响。

### 命令行（CLI）

```cmd
DSHSwitch.exe --cli status
DSHSwitch.exe --cli start
DSHSwitch.exe --cli stop
DSHSwitch.exe --cli restart
DSHSwitch.exe --cli doctor
```

- `status`：查询服务状态（JSON）。
- `doctor`：打印 `node`、`dsh` 路径、端口、工作目录、数据目录等诊断信息。
- 结果同时写入数据目录的 `last-cli-result.json`。

## 配置文件

配置位于数据目录下的 `settings.json`：

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `port` | DSH Web 服务端口 | `3080` |
| `dshHome` | DSH_HOME；留空用环境变量或 `~/.dsh` | `""` |
| `dshCmd` | 留 `auto` 自动定位，或写死 dsh 路径 | `"auto"` |
| `nodePath` | 留空自动找 node | `""` |
| `cwd` | 服务工作目录；留空用当前目录 | `""` |
| `args` | 附加启动参数 | `[]` |
| `openBrowserOnStart` | 启动成功后是否由本程序打开网页 | `false` |
| `startTimeoutSec` | 等待服务就绪的最长秒数 | `180` |
| `forceStopByPort` | 是否允许强停“非本软件”端口进程 | `false` |

> 数据目录：`%LOCALAPPDATA%\DSHSwitch`（可用环境变量 `DSH_SWITCH_DATA` 覆盖）。

## 目录结构

```
DSHSwitch/
├─ Program.cs          # 入口；单实例；--cli 分流
├─ MainForm.cs         # 主窗体：开关 + 状态 + 托盘
├─ ToggleButton.cs     # 手绘开关控件
├─ ServerController.cs # 启动/停止/状态探测（防误杀）
├─ DshResolver.cs      # 定位 node 与 dsh 入口
├─ Settings.cs         # 配置与路径常量
├─ Cli.cs              # 命令行模式
├─ DSHSwitch.csproj
└─ build-exe.bat       # 一键发布脚本
```

## 常见问题

- **启动失败/找不到 node**：确认 `node` 在 `PATH`，或在 `settings.json` 里设 `nodePath`。
- **找不到 dsh**：确认已安装 `@deepseek-ai/dsh`，或在 `settings.json` 里设 `dshCmd` 绝对路径。
- **停止时提示端口被占用**：该端口进程不是本软件启动的。确需强停，把 `forceStopByPort` 设为 `true`。
- **停止后重启有延迟**：进程 kill 后端口通常在几十毫秒到 1 秒内释放，程序已把确认等待限制在 1 秒内。

## License

MIT
