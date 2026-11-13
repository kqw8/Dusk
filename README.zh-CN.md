[English](README.md) | 简体中文

# Dusk

macOS 命令行工具，接入外接显示器时压暗 MacBook 内屏，断开后恢复。

**Dusk 镜像内屏并将亮度设为零，不切断面板供电，也不会主动压暗唯一可用的屏幕。**

## 安装

需要 macOS 13+ 和 Xcode 或 Command Line Tools；Homebrew 从源码构建。

```sh
brew install kqw8/tools/dusk
brew services start kqw8/tools/dusk
```

仅安装不会启动 Dusk。启动用户服务时不要使用 `sudo`；这也会启用登录自启。
**服务管理仅使用 `brew services`，不要混用原生 `dusk install/start/stop/uninstall/login`。** 迁移时先移除旧的独立代理，详见下方指南。

## 常用命令

```sh
dusk status
brew services list
brew services stop kqw8/tools/dusk
```

`dusk status` 显示显示器能力及保存状态，其中的代理注册信息不代表 Homebrew 服务状态。停止服务会恢复内屏并禁用登录自启。

## 限制

无需辅助功能或屏幕录制权限。私有 `DisplayServices` API 可能在 macOS 更新后失效。
已在 macOS 26 的 Apple Silicon MacBook Pro 搭配 Apple Studio Display 上测试，其他配置未经测试。开盖或切换镜像时可能短暂亮屏或闪屏。

## 卸载

先停止服务，等待内屏恢复：

```sh
brew services stop kqw8/tools/dusk
```

再删除二进制：

```sh
brew uninstall kqw8/tools/dusk
```

独立安装、源码安装、迁移、手动接管、原理、限制、调试及残留清理见[使用指南](docs/guide.zh-CN.md)。独立二进制未经 Apple 公证；安装脚本在校验通过后移除隔离属性。

[MIT 许可证](LICENSE)
