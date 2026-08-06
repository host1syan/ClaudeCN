---
layout: home

hero:
  name: ClaudeCN
  text: 全中文提示词的 Claude 智能体工作台
  tagline: CLI、桌面端和本地服务端一体化，内置 Claude CN、Kilo Free 渠道，支持长任务缓存与多 Agent 工作流
  image:
    src: /images/logo-horizontal.svg
    alt: ClaudeCN
  actions:
    - theme: brand
      text: 快速开始
      link: /guide/quick-start
    - theme: alt
      text: GitHub
      link: https://github.com/host1syan/claude-cn

features:
  - icon: "\U0001F5A5"
    title: 完整 TUI 交互
    details: 与官方 Claude Code 一致的 Ink 终端界面，支持 --print 无头模式
  - icon: "\U0001F9E0"
    title: 中文提示词与缓存
    details: 中文 Agent 工作流，支持 Anthropic Prompt Cache 用量映射；实际命中取决于上游服务
  - icon: "\U0001F193"
    title: Claude CN 与 Kilo Free
    details: 内置免费模型渠道；模型目录、额度、限流和可用性由上游服务决定
  - icon: "\U0001F916"
    title: 多 Agent 系统
    details: 多代理编排、并行任务执行、Teams 协作、Worktree 隔离
    link: /agent/
  - icon: "\U0001F9E9"
    title: Skills 系统
    details: 可扩展能力插件、自定义工作流、条件激活
    link: /skills/01-usage-guide
  - icon: "\U0001F310"
    title: 第三方模型支持
    details: 接入 Anthropic、OpenAI、DeepSeek、Ollama 等兼容模型
    link: /guide/third-party-models
  - icon: "\U0001F4AC"
    title: IM 接入
    details: 在桌面端配置 Telegram、飞书、微信和钉钉，远程对话并审批权限
    link: /im/
  - icon: "\U0001F4BB"
    title: Computer Use
    details: 桌面控制功能 — 截屏、鼠标、键盘操作
    link: /features/computer-use
  - icon: "\U0001F5A5"
    title: 桌面端
    details: 基于 Electron 和 React 的图形化客户端，支持 macOS、Windows 和 Linux
    link: /desktop/
  - icon: "\U0001F4BE"
    title: Memory 系统
    details: 跨会话持久化记忆、自动提取、智能检索和 AutoDream 整合
    link: /memory/
  - icon: "\U0001F527"
    title: 输出压缩方向
    details: RTK 工具输出压缩与 Caveman 模型输出压缩属于实验方向，以当前构建版本为准
---

ClaudeCN 面向中文开发工作流，提供 CLI、桌面端和本地服务端三种入口。先从[快速开始](/guide/quick-start)启动，再按需配置模型渠道、记忆、Skills、Computer Use、远程访问和 IM 适配器。
