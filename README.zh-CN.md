[English](README.md) | 简体中文

# Dusk

一个小巧的 macOS 命令行工具：接入外接显示器时压暗 MacBook 内置屏幕，断开后恢复。使用外接屏时仍可保持开盖，方便使用摄像头、散热或满足个人习惯。

Dusk **将内屏镜像到外接屏，并把内屏亮度设为零**，并不从物理上关闭面板供电。它不会主动压暗唯一可用的屏幕。

## 安装

需要 macOS 13 或更新版本。已在运行 macOS 26 的 Apple Silicon MacBook Pro 搭配 Apple Studio Display 上测试。

### Homebrew（推荐）

通过 [kqw8/homebrew-tools](https://github.com/kqw8/homebrew-tools) 安装：

```bash
brew install kqw8/tools/dusk
```

公式从源码构建，因此需要 Xcode 或 Command Line Tools（`xcode-select --install`）。**安装不会启动 Dusk。** 请单独启动，不要使用 `sudo`：

```bash
brew services start kqw8/tools/dusk
brew services list
```

这会启动用户服务并启用登录自启。停止服务、恢复内屏并关闭登录自启：

```bash
brew services stop kqw8/tools/dusk
```

这种安装方式仅使用 `brew services` 管理服务。不要混用 `dusk install`、`start`、`stop`、`uninstall` 或 `login`。从独立代理迁移时，请先用旧二进制执行 `dusk uninstall`，再启动 Homebrew 服务。

### 独立二进制

无需 Homebrew 或 Xcode。安装脚本下载最新的通用二进制（Apple Silicon + Intel），校验 SHA-256 后安装到 `/usr/local/bin`：

```bash
curl -fsSL https://raw.githubusercontent.com/kqw8/Dusk/main/scripts/install.sh | bash
```

二进制未经 Apple 公证。安装脚本在校验通过后移除隔离属性；通过浏览器下载可能触发 Gatekeeper。

仅安装不会启动 Dusk。安装并启动其自带的后台代理：

```bash
dusk install
```

这会立即启动代理，默认不启用登录自启。

### 从源码构建

需要 Xcode 或 Command Line Tools 提供的 Swift 工具链：

```bash
git clone https://github.com/kqw8/Dusk.git
cd Dusk
swift build -c release
.build/release/dusk install
```

最后一条命令启动独立代理，并指向构建出的可执行文件。代理安装期间，请保持该文件路径不变。

## 使用

两种安装方式均可使用：

```bash
dusk status     # 显示支持能力、显示器及保存的状态
dusk --version # 输出版本
dusk --help    # 显示帮助
```

`dusk status` 显示的是 Dusk 自带代理的注册情况，而非 Homebrew 服务状态。Homebrew 的运行和登录自启状态请用 `brew services list` 查看。

以下命令**仅用于独立代理**：

| 命令 | 作用 |
| --- | --- |
| `dusk install` | 安装并启动代理，不启用登录自启 |
| `dusk start` | 启动代理 |
| `dusk stop` | 停止代理并恢复内屏 |
| `dusk login on` | 启用登录自启 |
| `dusk login off` | 关闭登录自启，不停止当前运行的代理 |
| `dusk uninstall` | 恢复内屏、停止代理，移除其注册信息、状态和日志；保留二进制 |

前台调试时，先停止后台服务，再不带参数运行 `dusk`。按 Ctrl-C 退出并恢复内屏。

## 工作原理

- 接入外接屏后，将内屏镜像到外接屏、关闭内屏自动亮度并把亮度设为零。内屏不再是窗口或指针可以进入的独立桌面。
- 断开外接屏或正常退出后恢复内屏。仅剩内屏时，恢复逻辑会确保它可见。
- 手动接管：调高内屏亮度或切回扩展模式即可重新使用内屏。将亮度拖回零可再次压暗；重新连接外接屏也会重新触发。
- 事件驱动，无轮询、无第三方依赖。响应显示器变化、唤醒和亮度通知，以 `launchd` 用户代理运行。

无需辅助功能或屏幕录制权限。故障通过通知、标准错误输出和 `dusk status` 展示的保存状态报告；后台正常运行时不产生日志。独立代理日志位于 `~/Library/Logs/dusk.out.log` 和 `~/Library/Logs/dusk.err.log`。Homebrew 日志位于 `$(brew --prefix)/var/log/`，文件名为 `dusk.log` 和 `dusk.err.log`。

## 限制

- macOS 没有公开 API 可以从物理上关闭单块屏幕。Dusk 使用镜像和压暗，不控制面板供电。
- 亮度和自动亮度控制依赖运行时动态加载的 Apple 私有 `DisplayServices` 框架。macOS 更新可能导致失效；控制能力缺失时会报告，而非静默忽略。将亮度拖到零后立即重新压暗也依赖私有亮度通知。
- 开盖时，面板可能短暂亮起，再被 Dusk 压暗。从扩展切换到镜像可能耗时一两秒，并出现闪屏。
- 如果之前启用了自动亮度，恢复时会重新启用，因此环境光可能影响恢复后的亮度。
- Intel Mac、HDMI/DisplayPort 显示器、多外接屏，以及上述已测配置之外的 macOS 版本尚未测试。

## 卸载

**先停止服务，等待内屏恢复后再删除二进制。**

Homebrew：

```bash
brew services stop kqw8/tools/dusk
brew uninstall kqw8/tools/dusk
```

独立安装：

```bash
dusk stop
dusk uninstall
sudo rm /usr/local/bin/dusk
```

自定义安装或源码构建时，请删除实际的可执行文件，而非 `/usr/local/bin/dusk`。独立卸载命令会清理其代理、状态和日志；Homebrew 卸载后可能留下运行状态及日志。

## 许可证

[MIT](LICENSE) — Copyright (c) 2026 kqw8.
