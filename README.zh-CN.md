# Windows Codex 与 OpenClaw 代理安装包

**简体中文** | [English](README.md)

本项目提供两个相互独立的 Windows 迁移包，帮助 Codex GUI 和 OpenClaw Gateway 使用用户自己运行的本地 HTTP 代理，面向希望通过双击安装、按提示操作的 Windows 用户。

> **安装包状态：** 编写本文档时，仓库默认分支尚未包含安装包目录，GitHub Releases 也没有发布安装压缩包。本文是使用说明，不是安装程序。在维护者上传两个迁移包或包含它们的 Release 压缩包前，仅克隆本仓库无法完成安装。

## 项目能做什么

有些 Windows 应用不会自动使用代理软件或 Windows 中配置的本地代理。本项目提供两种独立方式，把本地 HTTP 代理地址交给相应程序：

| 迁移包 | 适用对象 | 代理设置方式 |
|---|---|---|
| **Codex Proxy Migration** | Codex GUI | 将 HTTP_PROXY、HTTPS_PROXY、NO_PROXY 写入当前 Windows 用户级环境变量。需要彻底重启 Codex GUI 才能继承变化。 |
| **OpenClaw Proxy Migration** | OpenClaw Gateway 及其子进程（包括由 OpenClaw 启动的 Codex 进程） | 通过专用启动脚本，仅给 Gateway 进程及其子进程设置代理变量。 |

可以只安装其中一个，也可以两个都安装。两者互不依赖：执行 Codex 的 proxy stop 不会取消 OpenClaw 专用启动脚本给 Gateway 设置的代理。

### 本项目不会做什么

- 不包含或安装代理软件。用户需要自行准备并启动本地 HTTP 代理。
- 不修改 Windows“设置 → 网络和 Internet → 代理”、WinHTTP 或系统级环境变量。
- 不安装、不登录，也不修改 Codex 或 OpenClaw 本体。
- 不修改 Codex/OpenClaw 源码、provider 配置或 WebSocket 传输方式。
- 本地代理可用不代表所有网络或服务问题都一定消失。

## 开始前准备

1. 使用有权限运行相关应用和计划任务的 Windows 账户。
2. 先安装并登录需要使用的程序：
   - 使用 Codex 迁移包前，先安装并登录 Codex GUI。
   - 使用 OpenClaw 迁移包前，先安装并登录 OpenClaw，并确认存在 gateway.cmd。
3. 自行安装代理软件，配置一个**本地 HTTP 代理监听端口**，并在运行安装程序前启动它。地址通常是 127.0.0.1，端口取决于代理软件；不要把示例端口当成实际端口。
4. 有 Release 压缩包后，从仓库的 **Releases** 页面下载并解压到固定位置，例如 C:\Tools\Codex_OpenClaw_Proxy_Installer。不要在 ZIP 内直接运行安装程序；安装后也不要移动或重命名文件夹，因为用户 PATH 或计划任务操作可能引用该路径。
5. 不要把另一台电脑生成的 config.ps1、install_state.json、Backup 或 Logs 复制到新电脑。这些文件包含本机配置、回滚记录或诊断信息。

迁移包不提供代理软件、Codex/OpenClaw 安装程序或登录凭据。每台电脑都应由用户自行登录，不要复制另一台电脑的认证目录或令牌。

## 选择并安装迁移包

### A. Codex GUI 代理迁移包

如果需要让 Codex GUI 使用本地代理，使用此包。

1. 打开解压后的 Codex_Proxy_Migration 文件夹。
2. 双击 install_codex.bat。
3. 按照窗口提示操作：
   - 检测到一个代理时，确认是否使用。
   - 检测到多个候选时，选择正确的地址和端口。
   - 没有检测到时，输入代理软件显示的本地代理主机和端口。
4. 安装程序会检查端口是否正在监听。若不可用，请启动代理软件或核实地址；不要猜端口继续。
5. 安装程序会生成本机专用 config.ps1，把本包的 CodexGUI 文件夹加入**用户级 PATH**，并询问现在是否启用用户级代理变量。
6. 打开一个**新的**命令提示符窗口，输入：

~~~cmd
proxy status
~~~

若安装时选择暂不启用，准备使用时再输入 proxy start。

可使用以下命令：

~~~cmd
proxy status
proxy start
proxy stop
~~~

- proxy start 会检查本地代理，然后设置当前用户的 HTTP_PROXY、HTTPS_PROXY 和 NO_PROXY。
- proxy stop 会删除这三个当前用户级变量。
- proxy status 会显示用户级变量的持久值，并只读显示 Windows 代理状态。

这些是用户级环境变量，不是 Windows 系统代理。它们会持续保留到用户停止代理设置或卸载。已经运行的 Codex GUI 不会自动读取新环境：执行 proxy start **或** proxy stop 后，都要彻底退出 Codex GUI 及其后台进程，再重新启动。

如果 proxy start 自动发现了新端口，用户环境变量可能已经更新，但 config.ps1 中仍保留旧端口。如果 proxy status 显示配置的监听端口不可用，请将显示的 HTTP_PROXY/HTTPS_PROXY 与代理软件地址核对；必要时把本包 config.ps1 中的 ProxyHost、ProxyPort 改为当前实际监听地址。

### B. OpenClaw 代理迁移包

如果需要让 OpenClaw Gateway 及其子进程使用本地代理，使用此包。

1. 打开解压后的 OpenClaw_Proxy_Migration 文件夹。
2. 双击 install_openclaw.bat。
3. 按窗口提示操作：
   - 确认检测到的代理，或从多个候选中选择一个。
   - 没有检测到代理时，输入代理软件显示的主机和端口。
   - 确认 Gateway 端口和 gateway.cmd 路径。默认路径位于当前用户的 .openclaw 目录下；只有安装位置不同时才需要修改。
4. 安装程序会检查代理端口和 gateway.cmd 是否可用，然后生成本机专用 config.ps1。
5. 如果找到名为 **OpenClaw Gateway** 的现有计划任务，程序会把它的**操作（Action）**改为调用本包 PowerShell 启动脚本，并备份原任务 XML。如果没有找到任务，程序不会凭空创建任务。
6. 安装程序询问是否立即启动 Gateway 时，可以选择是；否则可通过任务计划程序启动现有 **OpenClaw Gateway** 任务。如果安装程序没有更新任务，也可以从解压目录手动运行 OpenClaw\start_openclaw_gateway.ps1。

启动脚本会先检查代理，再为 Gateway 进程及其子进程设置 HTTP_PROXY、HTTPS_PROXY、NO_PROXY。它不会把这些变量写入 Windows 用户级或系统级环境。计划任务仍在使用该启动脚本期间，请保持迁移包文件夹路径不变。

基础检查可运行包内的 OpenClaw\test_openclaw_proxy.ps1。Gateway 启动日志写入 Logs 文件夹。本地端口检测只能确认该地址上有程序监听，并不能单独证明远端服务一定可访问。

## 代理端口发生变化时

代理端口由代理软件决定。端口变化后，先启动代理软件并确认它显示的本地 HTTP 地址。

- **Codex：**运行 proxy start。配置端口不可用时，命令会尝试发现替代端口。如果无法唯一确定或没有发现代理，请将 Codex_Proxy_Migration\config.ps1 中的 ProxyHost 和 ProxyPort 改为实际地址，然后重新运行 proxy start。之后彻底重启 Codex GUI。
- **OpenClaw：**Gateway 启动脚本会尝试从可用环境变量/系统代理信息中识别唯一代理。如果不能可靠确定，会停止而不是猜测。将 OpenClaw_Proxy_Migration\config.ps1 中的 ProxyHost、ProxyPort 改为代理软件显示的地址，再重新启动 Gateway。
- 不要为了这两个迁移包使用 setx、修改系统级变量或更改 Windows 系统代理。

## 卸载

卸载程序运行完成前，请保留迁移包文件夹。两个包有各自的卸载程序，可以独立卸载。

### 卸载 Codex 迁移包

1. 如果安装后曾手动运行 proxy start，请注意：只有安装程序当时启用了代理，卸载程序才会恢复保存的原始用户环境。如果后来才手动启用，请先运行 proxy stop；它会删除这三个变量，不会恢复安装前可能存在的自定义值。如需保留原值，请先自行记录并在卸载后恢复。
2. 双击 uninstall_codex.bat。
3. 按提示输入 YES 确认。
4. 卸载后彻底退出并重新启动 Codex GUI。

如果安装程序当时启用了代理，卸载程序会根据安装记录恢复保存的原始用户环境，并只删除本包添加的用户 PATH 项。如果安装时选择暂不启用、后来又手动运行了 proxy start，请先运行 proxy stop；此情况下卸载程序不会自动恢复更早的环境。它不会卸载 Codex GUI 或删除认证文件。

### 卸载 OpenClaw 迁移包

1. 双击 uninstall_openclaw.bat。
2. 按提示输入 YES。
3. 如果询问是否删除本包生成的日志，选择删除或保留。

卸载程序会尝试恢复备份的 **OpenClaw Gateway** 任务操作和之前的包配置。它会保留 Backup 文件夹，不会卸载 OpenClaw 或删除认证信息。如果缺少有效安装记录或回滚备份，脚本可能拒绝修改，以免猜测性回滚。

## 常见问题

- **提示代理不可用：**启动代理软件，并核实本地 HTTP 代理主机和端口。除非代理软件明确提供 HTTP 代理，否则不要填写仅供 SOCKS 使用的端口。
- **没有检测到代理：**输入代理软件显示的精确地址。常见端口会因软件和配置不同而变化。
- **Codex GUI 运行后仍然重连：**检查 proxy status，彻底退出所有 Codex GUI/后台进程，再重新打开。迁移包无法修复无关的服务器、账户或网络故障。
- **OpenClaw 无法启动：**确认配置路径下存在 gateway.cmd、代理正在运行，且 **OpenClaw Gateway** 任务指向当前迁移包目录。查看 Logs\openclaw-gateway-proxy.log。
- **端口变化后仍未生效：**按上文修改对应迁移包的 config.ps1。不要修改另一个迁移包；两套代理控制互相独立。
- **卸载不能恢复计划任务：**保留 Backup 文件夹。回滚完成前不要删除迁移包。

## 安装包内容与安全

正式发布的压缩包应包含两个文件夹：Codex_Proxy_Migration 和 OpenClaw_Proxy_Migration。每个包都包含双击安装/卸载入口、脚本和示例配置。安装程序会生成本机专用状态，例如 config.ps1、install_state.json；运行期间还可能生成任务 XML 备份和日志。

未经检查和脱敏，不要公开分享生成的 config.ps1、install_state.json、计划任务 XML 备份或诊断日志。它们不是登录凭据，但可能暴露本机路径和配置信息。

## 许可与使用责任

使用每次发布时附带的许可证和 Release Notes。运行脚本前，请确认使用的代理软件可信，并自行审阅脚本。本项目不收集凭据，也不保证任何服务始终可访问。
