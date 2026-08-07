# 桌面端文档

> 图形化的 AI Code Editor，支持多会话、多标签、IM 接入的完整桌面体验。

![桌面端界面](../images/desktop_ui/01_full_ui.png)

---

## 文档目录

### [快速上手](./01-quick-start.md)

面向用户的桌面端使用指南：界面布局、对话操作、多标签、权限控制、项目管理、模型配置、IM 适配器、定时任务。

### [功能详解](./03-features.md)

深入每个功能模块：聊天引擎、代码展示、工具调用、Agent Teams、提供商管理、技能/Agent、定时任务、IM 适配器。

### [安装指南](./04-installation.md)

下载安装、macOS / Windows 常见问题。

### [H5 访问](./06-h5-access.md)

面向个人和团队的可选浏览器访问：开启 H5、生成 Token、配置允许来源，通过局域网或反向代理在手机上访问聊天界面。

---

## 开始使用

1. 阅读[安装指南](./04-installation.md)下载安装对应平台客户端。
2. 阅读[快速上手](https://github.com/host1syan/claude-cn/blob/main/docs/desktop/01-quick-start.md)了解界面和操作。
3. 配置 AI 模型提供商，开始对话。
4. 按需启用记忆、Skills、Computer Use、IM 远程接入等功能。

## 核心概念

| 概念 | 说明 |
|------|------|
| **Session** | 一次对话会话，绑定工作目录 |
| **Tab** | 标签页，对应一个 Session 或特殊页面 |
| **Provider** | AI 模型提供商，支持 Anthropic 兼容接口 |
| **Adapter** | IM 适配器，Telegram / 飞书等接入 |

---

需要更深入的实现细节或参与贡献，请在[官方仓库](https://github.com/host1syan/claude-cn)查看源码与开发文档。