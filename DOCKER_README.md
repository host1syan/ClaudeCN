# Claude CN - Docker 镜像

<p align="center">
  <img src="docs/images/app-icon.png" alt="Claude CN" width="200">
</p>

<div align="center">

[![Docker Hub](https://img.shields.io/badge/Docker_Hub-host1syan/claude--cn-blue)](https://hub.docker.com/r/host1syan/claude-cn)
[![License](https://img.shields.io/github/license/host1syan/claude-cn)](https://github.com/host1syan/claude-cn/blob/main/LICENSE)
[![中文](https://img.shields.io/badge/🇨🇳_中文-当前-blue)](#中文版)
[![English](https://img.shields.io/badge/🇺🇸_English-Available-green)](#english-version)

</div>

---

## 中文版

### 项目简介

Claude CN 是一个重新构建的 **Claude Code 工作台**。它提供了一个图形化界面，集成了会话管理、多项目支持、分支/Worktree 隔离、代码 Diff 查看、权限审批、模型提供商配置、Computer Use、H5 远程访问、IM 接入和定时任务等功能。

Claude CN 面向中文开发工作流，内置中文优先的 Agent 和工具提示词，减少中英切换。实际 Token 消耗取决于模型、任务复杂度和上下文内容；模型、上游网关及平台自身策略仍然有效。

**Docker 镜像特点：**
- 🐳 **开箱即用**：无需安装 Bun、Node.js 等依赖，直接运行
- 🌐 **Web 界面**：通过浏览器访问图形化工作台
- 📁 **WebDAV 支持**：可远程挂载和管理文件
- 🐘 **PHP 环境**：内置 PHP 7.4，可运行 PHP 脚本
- 🔒 **安全隔离**：容器化运行，不影响宿主系统
- 📱 **H5 远程访问**：支持手机或其他设备接入

### 必要环境变量

以下三个环境变量**必须设置**，容器才能正常启动：

| 变量名 | 说明 |
|--------|------|
| `CLAUDE_H5_TOKEN` | H5 远程访问令牌 |
| `WEBDAV_USER` | WebDAV 用户名 |
| `WEBDAV_PASSWORD` | WebDAV 密码 |

所有 `docker run` 命令都必须包含这三个变量。

### 快速开始

#### 1. 拉取镜像

```bash
docker pull host1syan/claude-cn
```

#### 2. 基本运行

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_secret_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn
```

#### 3. 带 API 配置运行（推荐）

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_secret_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -e ANTHROPIC_AUTH_TOKEN=your_api_key \
  -e ANTHROPIC_BASE_URL=https://api.anthropic.com \
  -e ANTHROPIC_MODEL=claude-sonnet-4-20250514 \
  -v claude-data:/data \
  host1syan/claude-cn
```

#### 4. 访问界面

打开浏览器访问：`http://localhost:3456`

### 环境变量配置

#### 必须设置

| 变量名 | 说明 | 示例 |
|--------|------|------|
| `CLAUDE_H5_TOKEN` | H5 远程访问令牌 | `my_token_123` |
| `WEBDAV_USER` | WebDAV 用户名 | `admin` |
| `WEBDAV_PASSWORD` | WebDAV 密码 | `strong_password` |

#### 可选配置

| 变量名 | 说明 | 示例 |
|--------|------|------|
| `ANTHROPIC_AUTH_TOKEN` | API 认证令牌 | `sk-ant-xxx` |
| `ANTHROPIC_BASE_URL` | API 基础 URL | `https://api.anthropic.com` |
| `ANTHROPIC_MODEL` | 使用的模型 | `claude-sonnet-4-20250514` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet 模型 | `claude-sonnet-4-20250514` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku 模型 | `claude-haiku-4-5-20250414` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus 模型 | `claude-opus-4-7-20250514` |
| `API_TIMEOUT_MS` | API 超时时间（毫秒） | `3000000` |
| `CLAUDE_CONFIG_DIR` | 配置目录路径 | `/data/.claude` |
| `DISABLE_TELEMETRY` | 禁用遥测 | `1` |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 禁用非必要网络请求 | `1` |

### 功能说明

#### 🌐 Web 界面

- **会话管理**：创建、切换、删除会话
- **多项目支持**：同时管理多个代码项目
- **代码编辑**：在线编辑代码，支持 Diff 查看
- **权限审批**：集中管理 AI 操作权限

#### 📁 WebDAV 文件管理

容器已内置 WebDAV 服务（由 `WEBDAV_USER` / `WEBDAV_PASSWORD` 启用），可通过以下方式访问：

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn
```

WebDAV 访问地址：`http://localhost:3456/webdav/`

支持的操作系统：
- **macOS**：Finder → 前往 → 连接服务器（`http://localhost:3456/webdav/`）
- **Windows**：文件资源管理器 → 映射网络驱动器
- **Linux**：`davfs2` 或 `rclone`

#### 🐘 PHP 环境

内置 PHP 7.4，支持运行 PHP 脚本：

```bash
# 上传 PHP 文件到容器
docker cp your_script.php claude-cn:/data/php/

# 访问 PHP 脚本
# http://localhost:3456/php/your_script.php
```

#### 📱 H5 远程访问

1. 启动容器时设置 `CLAUDE_H5_TOKEN`
2. 在设备浏览器访问 `http://your-server-ip:3456`
3. 输入令牌进行认证

### 数据持久化

```bash
# 使用 Docker 卷
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v claude-data:/data \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn

# 或使用本地目录
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v /path/to/local/data:/data \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn
```

### 常用命令

```bash
# 查看日志
docker logs -f claude-cn

# 进入容器
docker exec -it claude-cn /bin/bash

# 停止容器
docker stop claude-cn

# 重启容器
docker restart claude-cn

# 查看容器状态
docker ps | grep claude-cn
```

### 故障排除

#### 1. 无法访问 Web 界面

```bash
# 检查容器是否正常运行
docker ps | grep claude-cn

# 检查端口是否被占用
netstat -tlnp | grep 3456

# 查看容器日志
docker logs claude-cn
```

#### 2. PHP 脚本无法执行

```bash
# 检查 PHP 文件权限
docker exec claude-cn ls -la /data/php/

# 重新启动 PHP-FPM
docker exec claude-cn service php7.4-fpm restart
```

#### 3. WebDAV 连接失败

```bash
# 确认环境变量已设置
docker inspect claude-cn | grep WEBDAV

# 测试 WebDAV 连接
curl -u admin:your_webdav_password http://localhost:3456/webdav/
```

### 高级配置

#### 使用 OpenAI 兼容接口

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -e ANTHROPIC_AUTH_TOKEN=sk-anything \
  -e ANTHROPIC_BASE_URL=http://your-litellm-proxy:4000 \
  -e ANTHROPIC_MODEL=gpt-4o \
  host1syan/claude-cn
```

#### 使用 DeepSeek

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -e ANTHROPIC_AUTH_TOKEN=sk-anything \
  -e ANTHROPIC_BASE_URL=http://your-litellm-proxy:4000 \
  -e ANTHROPIC_MODEL=deepseek-chat \
  host1syan/claude-cn
```

### 安全建议

1. **修改默认 H5 令牌**：不要使用 `123` 等弱令牌
2. **使用强密码**：WebDAV 密码应足够复杂
3. **限制访问**：仅在需要时暴露端口
4. **定期更新**：及时更新镜像以获取安全补丁

### 许可证

本项目基于 MIT 许可证。原始 Claude Code 源码版权归 Anthropic 所有。

---

## English Version

### Project Overview

Claude CN is a rebuilt **Claude Code workbench**. It provides a graphical interface integrating session management, multi-project support, branch/worktree isolation, code diff viewing, permission approval, model provider configuration, Computer Use, H5 remote access, IM integration, and scheduled tasks.

Claude CN targets Chinese development workflows with Chinese-first agent and tool prompts, reducing language switching. Actual token usage depends on model, task complexity, and context. Model, upstream gateway, and platform policies still apply.

**Docker Image Features:**
- 🐳 **Ready to Use**: No need to install Bun, Node.js, or other dependencies
- 🌐 **Web Interface**: Access the graphical workstation via browser
- 📁 **WebDAV Support**: Remote file mounting and management
- 🐘 **PHP Environment**: Built-in PHP 7.4 for running PHP scripts
- 🔒 **Secure Isolation**: Containerized operation, no impact on host system
- 📱 **H5 Remote Access**: Support access from mobile or other devices

### Required Environment Variables

The following **3 variables are mandatory** for the container to start properly:

| Variable | Description |
|----------|-------------|
| `CLAUDE_H5_TOKEN` | H5 remote access token |
| `WEBDAV_USER` | WebDAV username |
| `WEBDAV_PASSWORD` | WebDAV password |

Every `docker run` command must include these three.

### Quick Start

#### 1. Pull the Image

```bash
docker pull host1syan/claude-cn
```

#### 2. Basic Run

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_secret_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn
```

#### 3. Run with API Configuration (Recommended)

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_secret_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -e ANTHROPIC_AUTH_TOKEN=your_api_key \
  -e ANTHROPIC_BASE_URL=https://api.anthropic.com \
  -e ANTHROPIC_MODEL=claude-sonnet-4-20250514 \
  -v claude-data:/data \
  host1syan/claude-cn
```

#### 4. Access the Interface

Open browser: `http://localhost:3456`

### Environment Variables

#### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `CLAUDE_H5_TOKEN` | H5 remote access token | `my_token_123` |
| `WEBDAV_USER` | WebDAV username | `admin` |
| `WEBDAV_PASSWORD` | WebDAV password | `strong_password` |

#### Optional

| Variable | Description | Example |
|----------|-------------|---------|
| `ANTHROPIC_AUTH_TOKEN` | API authentication token | `sk-ant-xxx` |
| `ANTHROPIC_BASE_URL` | API base URL | `https://api.anthropic.com` |
| `ANTHROPIC_MODEL` | Model to use | `claude-sonnet-4-20250514` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet model | `claude-sonnet-4-20250514` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku model | `claude-haiku-4-5-20250414` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus model | `claude-opus-4-7-20250514` |
| `API_TIMEOUT_MS` | API timeout (milliseconds) | `3000000` |
| `CLAUDE_CONFIG_DIR` | Config directory path | `/data/.claude` |
| `DISABLE_TELEMETRY` | Disable telemetry | `1` |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Disable non-essential traffic | `1` |

### Features

#### 🌐 Web Interface

- **Session Management**: Create, switch, delete sessions
- **Multi-Project Support**: Manage multiple code projects simultaneously
- **Code Editing**: Online code editing with Diff viewing
- **Permission Approval**: Centralized AI operation permission management

#### 📁 WebDAV File Management

WebDAV is enabled via `WEBDAV_USER` / `WEBDAV_PASSWORD` env vars:

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn
```

WebDAV access URL: `http://localhost:3456/webdav/`

Supported operating systems:
- **macOS**: Finder → Go → Connect to Server (`http://localhost:3456/webdav/`)
- **Windows**: File Explorer → Map Network Drive
- **Linux**: `davfs2` or `rclone`

#### 🐘 PHP Environment

Built-in PHP 7.4 for running PHP scripts:

```bash
# Upload PHP files to container
docker cp your_script.php claude-cn:/data/php/

# Access PHP scripts
# http://localhost:3456/php/your_script.php
```

#### 📱 H5 Remote Access

1. Set `CLAUDE_H5_TOKEN` when starting container
2. Access `http://your-server-ip:3456` in device browser
3. Enter token for authentication

### Data Persistence

```bash
# Using Docker volume
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v claude-data:/data \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn

# Or using local directory
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v /path/to/local/data:/data \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  host1syan/claude-cn
```

### Common Commands

```bash
# View logs
docker logs -f claude-cn

# Enter container
docker exec -it claude-cn /bin/bash

# Stop container
docker stop claude-cn

# Restart container
docker restart claude-cn

# Check container status
docker ps | grep claude-cn
```

### Troubleshooting

#### 1. Cannot Access Web Interface

```bash
# Check if container is running
docker ps | grep claude-cn

# Check if port is occupied
netstat -tlnp | grep 3456

# View container logs
docker logs claude-cn
```

#### 2. PHP Scripts Not Executing

```bash
# Check PHP file permissions
docker exec claude-cn ls -la /data/php/

# Restart PHP-FPM
docker exec claude-cn service php7.4-fpm restart
```

#### 3. WebDAV Connection Failed

```bash
# Verify environment variables are set
docker inspect claude-cn | grep WEBDAV

# Test WebDAV connection
curl -u admin:your_webdav_password http://localhost:3456/webdav/
```

### Advanced Configuration

#### Using OpenAI Compatible Interface

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -e ANTHROPIC_AUTH_TOKEN=sk-anything \
  -e ANTHROPIC_BASE_URL=http://your-litellm-proxy:4000 \
  -e ANTHROPIC_MODEL=gpt-4o \
  host1syan/claude-cn
```

#### Using DeepSeek

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=your_token \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_webdav_password \
  -e ANTHROPIC_AUTH_TOKEN=sk-anything \
  -e ANTHROPIC_BASE_URL=http://your-litellm-proxy:4000 \
  -e ANTHROPIC_MODEL=deepseek-chat \
  host1syan/claude-cn
```

### Security Recommendations

1. **Change Default H5 Token**: Do not use weak tokens like `123`
2. **Use Strong Passwords**: WebDAV passwords should be sufficiently complex
3. **Restrict Access**: Only expose ports when needed
4. **Regular Updates**: Update images promptly to get security patches

### License

This project is licensed under the MIT License. Original Claude Code source code is copyrighted by Anthropic.

---

## Links

- **GitHub**: [github.com/host1syan/claude-cn](https://github.com/host1syan/claude-cn)
- **Documentation**: [docs.claudecode-cn.relakkesyang.org](https://docs.claudecode-cn.relakkesyang.org)
- **Issues**: [GitHub Issues](https://github.com/host1syan/claude-cn/issues)

---

Made with ❤️ for the AI coding community