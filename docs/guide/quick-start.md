# 快速开始

ClaudeCN 提供**桌面客户端、终端 CLI、Docker** 三种使用方式。首次使用先在设置中选择模型渠道，再开始对话。

## 1. 选择使用方式并安装

| 方式 | 适用场景 | 安装 |
|------|----------|------|
| 桌面客户端 | 日常图形化使用 | 从 [GitHub Releases](https://github.com/host1syan/claude-cn/releases) 下载安装包 |
| Docker | 浏览器访问、远程部署 | 见 [Docker 部署](../../DOCKER_README.md) |
| 终端 CLI | 命令行使用 | 随桌面端/服务端一起提供 |

桌面端支持 macOS、Windows、Linux，具体安装步骤见[桌面端安装指南](../desktop/04-installation.md)。

## 2. 配置模型渠道

首次启动后，在**设置**中选择模型渠道：

- **Claude CN**：内置渠道配置，提供当前版本可用的免费模型路由。
- **Kilo Free**：Kilo Gateway 免费模型渠道，默认模型为 `stepfun/step-3.7-flash:free`。
- **自定义 API**：使用自己的 Anthropic 兼容 API，配置 `ANTHROPIC_BASE_URL`、`ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_MODEL`。

> 免费渠道依赖上游服务，模型目录、额度、限流和可用性可能变化。不要把免费渠道当作永久、无限或固定可用的服务。第三方模型和代理配置见[第三方模型](./third-party-models.md)。

## 3. 新建会话并开始

1. 点击侧边栏 `+` 新建会话。
2. 选择关联的**工作目录**（项目文件夹）。
3. 输入第一条消息，开始与 AI 对话。

ClaudeCN 支持在会话中执行任务、查看代码改动、审批权限、切换分支和隔离 Worktree 等操作，详见[桌面端功能](../desktop/03-features.md)。

## 4. 按需扩展

- [接入第三方模型](./third-party-models.md)
- [配置环境变量](./env-vars.md)
- [使用记忆系统](../memory/)
- [配置 IM 远程接入](../im/)
- [启用 Computer Use](../features/computer-use.md)

## 缓存提示

项目支持 Anthropic Prompt Cache 相关字段和缓存用量统计。长任务中，稳定系统提示词、工具列表和请求前缀更容易复用缓存；实际命中取决于模型渠道、缓存 TTL、请求结构和上下文变化，不承诺固定命中率。