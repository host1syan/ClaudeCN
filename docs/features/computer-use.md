# Computer Use 功能指南

> 授权后让 Claude 直接控制你的电脑——截屏、移动鼠标、点击按钮、输入文字、管理应用窗口。

---

## 功能简介

Computer Use 让 AI 模型能够**直接控制你的电脑**。它通过 Python 底层操作层实现所有系统交互——macOS 使用 `pyautogui` + `mss`，Windows 使用 `pyautogui` + `mss` + `win32gui` + `psutil`，让 Claude 可以执行真实桌面操作。

支持的操作（共 24 个工具）：

| 类别 | 工具 |
|------|------|
| 截屏 | `screenshot`、`zoom` |
| 鼠标 | 左/中/右击、双击、拖拽、移动、滚动等 |
| 键盘 | 输入文字、按键、按住组合键 |
| 应用 | 打开应用、切换显示器 |

---

## 支持的平台

| 平台 | 支持 |
|------|------|
| macOS（Apple Silicon / Intel） | ✅ 完整支持（推荐） |
| Windows x64 | ✅ 完整支持 |
| Linux | ⚠️ 部分受限 |

## 开始使用

### 1. 在桌面端启用

首次使用 Computer Use 时，系统会弹出**应用授权对话框**，选择允许 Claude 操作的应用，之后模型即可截图、点击、输入。

> Python 依赖会在首次使用 Computer Use 时自动安装到本地运行目录，无需手动操作。

### 2. 授予 macOS 权限

在 macOS 上首次使用需要授予两个系统权限：

#### Accessibility（辅助功能）

在「系统设置 → 隐私与安全性 → 辅助功能」中，将你的终端 / 桌面端应用添加到允许列表。

```
open "x-apple.systempreferences:com.apple.preference.security?Privacy_Accessibility"
```

#### Screen Recording（屏幕录制）

在「系统设置 → 隐私与安全性 → 屏幕录制」中，同样将应用加入允许列表。

### 3. 使用

在对话中用自然语言请求即可：

```
> 帮我打开网易云音乐，搜索一首歌
> 截个屏看看当前桌面
> 帮我在 VS Code 里打开终端
```

模型会先调用 `request_access` 请求权限，你在对话框中确认允许哪些应用后才开始操作。

### 禁用 Computer Use

只需使用普通 Coding Agent、不希望暴露电脑控制工具时，可在桌面端 **Settings > Computer Use** 关闭开关（配置写入 `~/.claude/claude-cn/computer-use-config.json`）。关闭后，新会话不再注入电脑控制工具。

---

## 安全机制

| 机制 | 说明 |
|------|------|
| **应用白名单** | 每次会话需明确授权允许操作的应用 |
| **并发保护** | 同一时间只有一个 Claude 会话可使用 Computer Use |
| **剪贴板保护** | 剪贴板输入文本时自动保存和恢复原始内容 |
| **操作确认** | 敏感操作（如系统快捷键）需要额外授权 |

---

## 环境变量（可选）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `CLAUDE_COMPUTER_USE_ENABLED` | `1` | 设为 `0` 可禁用 Computer Use |
| `CLAUDE_COMPUTER_USE_COORDINATE_MODE` | `pixels` | 坐标模式：`pixels` 或 `normalized_0_100` |
| `CLAUDE_COMPUTER_USE_CLIPBOARD_PASTE` | `1` | 是否启用剪贴板粘贴输入 |
| `CLAUDE_COMPUTER_USE_MOUSE_ANIMATION` | `1` | 是否启用鼠标动画 |
| `CLAUDE_COMPUTER_USE_HIDE_BEFORE_ACTION` | `0` | 操作前是否隐藏其他窗口 |
| `CLAUDE_COMPUTER_USE_DEBUG` | `0` | 调试模式 |

---

## 已知限制

- Linux 仅理论可行，需替换平台专有 API，当前未完整适配。
- 坐标由模型视觉能力分析截图实现，复杂或动态界面偶有识别偏差。
- 跨应用操作受操作系统辅助功能权限约束。