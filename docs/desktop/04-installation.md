# 安装指南

桌面端基于 **Electron**，提供 macOS / Windows / Linux 安装包。`v0.4.3` 起的正式 macOS Release 使用 Developer ID 签名和 notarization；更早版本或临时开发包仍可能需要手动放行。

## 下载

前往 [GitHub Releases](https://github.com/host1syan/claude-cn/releases) 下载对应平台的安装包：

| 平台 | 文件 |
|------|------|
| macOS (Apple Silicon / M 系列) | `Claude-Code-Cn-<版本>-mac-arm64.dmg` |
| macOS (Intel) | `Claude-Code-Cn-<版本>-mac-x64.dmg` |
| Windows (x64) | `Claude-Code-Cn-<版本>-win-x64.exe` |
| Windows (ARM64) | `Claude-Code-Cn-<版本>-win-arm64.exe` |
| Linux (x64) | `Claude-Code-Cn-<版本>-linux-x86_64.AppImage` 或 `...-linux-amd64.deb` |
| Linux (ARM64) | `Claude-Code-Cn-<版本>-linux-arm64.AppImage` 或 `...-linux-arm64.deb` |

> 不确定 Mac 架构？点击左上角  → 关于本机，芯片为「Apple M…」选 arm64，「Intel」选 x64。

## macOS 安装

双击 DMG 把应用拖入 `Applications`。`v0.4.3` 起的正式 Release 正常情况下只会出现 macOS 的标准下载来源确认，不需要执行 `xattr`。

如果安装的是旧版或 unsigned 临时包，首次打开可能提示**"已损坏"**或**"无法验证开发者"**，再在终端执行：

```bash
xattr -cr /Applications/Claude\ Code\ Cn.app
```

也可以在「系统设置 → 隐私与安全性」里点"仍要打开"。

## Windows 安装

双击 `.exe` 安装。首次运行如果 SmartScreen 弹出 **"Windows 已保护你的电脑"**，点击 **「更多信息」** → **「仍要运行」**。

## Linux 安装

AppImage：

```bash
chmod +x Claude-Code-Cn-<版本>-linux-x86_64.AppImage
./Claude-Code-Cn-<版本>-linux-x86_64.AppImage
```

> 提示缺少 FUSE：Ubuntu 22.04 及更早 `sudo apt install libfuse2`，24.04+ `sudo apt install libfuse2t64`。

deb：

```bash
sudo apt install ./Claude-Code-Cn-<版本>-linux-amd64.deb
```

## Web UI 模式

想要不安装桌面客户端、直接用浏览器使用，或在无界面 Linux 服务器上部署时，推荐使用 [Docker 部署](../../DOCKER_README.md)：运行容器后浏览器访问 `http://localhost:3456` 即可，无需在本机安装任何环境。

如果需要在局域网或其他设备上访问，可结合 [H5 访问](./06-h5-access.md) 使用令牌认证。

## 常见问题

**Q: 这个版本会自动更新吗？**

`v0.4.3` 起的正式 Release 会通过 GitHub Releases 检查更新，并下载对应平台的更新包。覆盖安装或应用内更新不会删除本地配置和会话数据（`~/.claude`）。

正式公开的 macOS Release 需要签名和公证；draft/unsigned 临时包仍可能需要手动放行。Windows 签名不是发布阻塞项，未签名安装包仍可更新，但可能出现 SmartScreen 提示。
