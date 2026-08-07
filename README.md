# ClaudeCN

<p align="center">
  <img src="docs/images/app-icon.svg" alt="ClaudeCN" width="220">
</p>

<div align="center">

[![GitHub Stars](https://img.shields.io/github/stars/host1syan/claude-cn?style=social)](https://github.com/host1syan/claude-cn/stargazers)
[![Docker Hub](https://img.shields.io/badge/Docker_Hub-host1syan/claude--cn-blue)](https://hub.docker.com/r/host1syan/claude-cn)
[![中文](https://img.shields.io/badge/中文-当前-blue)](README.md)
[![English](https://img.shields.io/badge/English-Available-green)](README.en.md)

</div>

ClaudeCN 是一套**全中文提示词**的仿 Claude 智能体工作台，支持 macOS、Windows、Linux。它把多会话、多项目、分支与 Worktree 隔离、代码 Diff、权限审批、多模型渠道、Computer Use、H5 远程访问、IM 接入和定时任务集中到一个工作台，并提供终端 CLI、本地服务端和桌面客户端三种入口。

中文优先的提示词减少中英切换，适合中文开发和长任务工作流。实际 Token 消耗取决于模型、任务复杂度和上下文内容；模型、上游网关及平台自身策略仍然有效。

## 快速上手

1. 前往 [GitHub Releases](https://github.com/host1syan/claude-cn/releases) 下载对应平台的安装包，或用 [Docker 镜像](DOCKER_README.md) 一键部署。
2. 首次启动后在设置中配置模型渠道、API Key 和默认模型。
3. 新建会话并选择工作目录，开始对话。

macOS 临时包可能需要手动放行；Windows 未签名安装包可能触发 SmartScreen。详见[安装指南](docs/desktop/04-installation.md)。

## 核心亮点

- **全中文 Agent 工作流**：核心任务管理、验证 Agent 和工具提示词采用中文，面向中文开发场景优化。
- **内置免费渠道**：内置 Claude CN 与 Kilo Free 渠道配置，提供当前版本可用的免费模型路由。
- **多会话工作台**：标签页、项目切换、会话历史、终端入口和后台任务集中管理。
- **分支与 Worktree 隔离**：新会话可选择仓库分支，并决定使用当前工作树或隔离 Worktree。
- **代码改动可视化**：查看 AI 文件修改、增删行、Diff 和工作区状态。
- **权限与确认流**：危险命令、工具调用和模型反问在桌面端集中审批。
- **多模型渠道**：支持 Anthropic 兼容 API、第三方模型、协议转换代理和本地模型。
- **Memory 与多 Agent**：跨会话记忆、Agent 编排、并行任务、Teams 协作和 Skills 扩展。
- **Computer Use**：授权后截图、点击、输入并控制桌面应用。
- **H5 与 IM 接入**：从手机或其他设备访问会话，也可通过 Telegram、飞书、微信、钉钉等渠道远程对话和审批。
- **定时任务与用量统计**：创建计划任务，查看 Token 使用趋势。

## 文档

| 文档 | 说明 |
|------|------|
| [快速开始](docs/guide/quick-start.md) | 安装、启动和渠道选择 |
| [第三方模型](docs/guide/third-party-models.md) | 配置 Anthropic 兼容 API、LiteLLM、DeepSeek 和内置免费模型 |
| [环境变量](docs/guide/env-vars.md) | 环境变量参考 |
| [桌面端](docs/desktop/index.md) | 安装、界面布局、功能详解、H5 访问 |
| [记忆系统](docs/memory/) | 跨会话持久化记忆 |
| [多 Agent 系统](docs/agent/) | Agent 编排、并行任务、Teams 协作 |
| [Skills 系统](docs/skills/) | 可扩展能力插件和自定义工作流 |
| [IM 接入](docs/im/) | 远程对话、项目切换和管理 |
| [Computer Use](docs/features/computer-use.md) | 截屏、鼠标和键盘控制 |
| [Docker 部署](DOCKER_README.md) | 浏览器访问的 Docker 部署 |
| [常见问题](docs/guide/faq.md) | 常见错误排查 |

## Docker 部署

Docker Hub 镜像：[`host1syan/claude-cn`](https://hub.docker.com/r/host1syan/claude-cn)。运行容器后直接用浏览器访问 Web 界面，无需在宿主机安装 Bun 或 Node.js。

```bash
docker pull host1syan/claude-cn

docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_secret_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -v claude-data:/data \
  host1syan/claude-cn
```

启动后打开 `http://localhost:3456`，输入 `CLAUDE_H5_TOKEN` 登录。完整说明见[Docker 使用文档](docker/USAGE.md) 和 [Docker 镜像说明](DOCKER_README.md)。

## 免责声明

本项目仅供学习与研究使用。免费渠道、第三方模型和上游服务遵循各自服务条款；请用户自行确认使用场景、数据安全和合规要求。原始 Claude Code 源码相关版权归原作者所有。