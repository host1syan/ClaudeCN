# Claude CN - Docker 使用文档

## 概述

`host1syan/claude-cn` 镜像将 Claude CN 工作台封装为 **Web 可访问的服务**，集成了 Nginx、PHP 7.4 环境和可选的 WebDAV 文件管理。运行容器后，直接用浏览器访问图形化工作台，无需在宿主机安装 Bun 或 Node.js。

### 主要特性

- 🌐 **Web 访问** — 浏览器即可使用，无需安装桌面客户端
- 🐘 **PHP 7.4 环境** — `/php/` 路由托管 PHP 网站，与工作台共享数据目录
- 📁 **WebDAV 文件管理** — `/webdav/` 路由将 `/data/` 目录映射出来，可挂载到本地电脑
- 🔄 **断线重连** — API 不稳定时自动指数退避重连，不丢失任务
- 🛡️ **零权限打扰** — Docker 模式自动跳过所有权限询问，后台静默运行
- 🔗 **H5 Token 认证** — 简单的访问令牌保护，环境变量可覆盖
- 📦 **Hugging Face Spaces 兼容** — 持久化目录运行时创建，构建时不写数据

---

## 快速开始

### 1. 拉取镜像

```bash
docker pull host1syan/claude-cn
```

### 2. 运行容器

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -e CLAUDE_H5_TOKEN=你的访问令牌 \
  -v /path/to/data:/data \
  host1syan/claude-cn
```

### 3. 访问

打开浏览器访问 `http://localhost:3456`，输入令牌即可。

> `CLAUDE_H5_TOKEN` 默认值为 `123`，生产环境务必通过 `-e` 覆盖为强令牌。建议将 `/data` 挂载到卷或宿主机目录，避免容器重建后数据丢失。

---

## 环境变量

| 变量 | 默认值 | 说明 |
|---|---|---|
| `CLAUDE_H5_TOKEN` | `123` | Web 界面登录令牌，通过 `-e` 覆盖 |
| `WEBDAV_USER` | — | （可选）WebDAV 用户名，设置后启用 `/webdav/` 路由 |
| `WEBDAV_PASSWORD` | — | （可选）WebDAV 密码，与 `WEBDAV_USER` 同时设置才生效 |
| `ANTHROPIC_AUTH_TOKEN` | — | API 认证令牌 |
| `ANTHROPIC_BASE_URL` | — | Anthropic 兼容网关地址 |
| `ANTHROPIC_MODEL` | — | 默认模型 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` / `ANTHROPIC_DEFAULT_HAIKU_MODEL` / `ANTHROPIC_DEFAULT_OPUS_MODEL` | — | Sonnet / Haiku / Opus 级别模型映射 |
| `API_TIMEOUT_MS` | `3000000` | API 请求超时（毫秒） |
| `CLAUDE_CONFIG_DIR` | `/data/.claude` | 配置、会话、Skills、插件目录 |
| `DISABLE_TELEMETRY` | `1` | 禁用遥测 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `1` | 禁用非必要网络请求 |

> Docker 模式相关优化（权限跳过、断线保持、CORS 放行等）由镜像内部自动启用，无需手动配置。

---

## 端口与路由

| 端口 | 用途 | 说明 |
|---|---|---|
| `3456` | nginx 入口 | 外部访问端口，Web UI + API + PHP |

| 路径 | 目标 | 说明 |
|---|---|---|
| `/` | Web 工作台 | Claude CN 的浏览器界面 |
| `/php/` | PHP 7.4 | PHP 网站（文件位于 `/data/php/`） |
| `/webdav/` | WebDAV | 文件管理（需设置 `WEBDAV_USER` + `WEBDAV_PASSWORD`） |
| `/health` | 健康检查 | 容器健康检查端点 |

---

## 数据持久化

所有持久数据存储在 `/data/` 目录下，挂载外部卷或宿主机目录：

```bash
-v /host/path/data:/data
```

| 路径 | 用途 |
|---|---|
| `/data/php/` | PHP 网站文件 |
| `/data/.claude/claude-cn/settings.json` | 配置文件（含 H5 token） |
| `/data/.claude/projects/` | 项目会话数据 |
| `/data/.claude/skills/` | 安装的 Skills |
| `/data/.claude/plugins/` | 安装的插件 |
| `/data/.claude/claude-cn/providers.json` | API 提供商配置 |

> 首次启动会自动创建上述目录结构和默认配置，无需在构建镜像时预写数据，兼容 Hugging Face Spaces 构建约束。

---

## WebDAV 远程文件管理

设置两个环境变量即可启用（不设置则不启动，无额外暴露）：

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v /path/to/data:/data \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=你的密码 \
  host1syan/claude-cn
```

挂载到本地：

| 系统 | 方法 |
|---|---|
| **macOS Finder** | 前往 → 连接服务器 → `http://your-host:3456/webdav/` |
| **Windows 资源管理器** | 右键"此电脑" → 映射网络驱动器 → `http://your-host:3456/webdav/` |
| **Linux** | `sudo mount -t davfs http://your-host:3456/webdav/ /mnt/webdav`（需安装 `davfs2`） |
| **第三方工具** | Rclone、Cyberduck、WinSCP 等都支持 WebDAV 协议 |

> **安全提示**：WebDAV 使用 HTTP Basic Auth 认证，建议在内网或搭配 HTTPS 代理使用；通过公网访问时建议配合 IP 白名单或 VPN。

---

## 访问模型渠道

| 场景 | 配置 |
|---|---|
| Anthropic 官方 | `-e ANTHROPIC_BASE_URL=https://api.anthropic.com` + `-e ANTHROPIC_AUTH_TOKEN=sk-ant-xxx` |
| OpenAI 兼容 | 通过 LiteLLM 等协议转换代理，`-e ANTHROPIC_BASE_URL=http://your-proxy:4000`，`ANTHROPIC_MODEL` 设为目标模型（如 `gpt-4o`） |
| DeepSeek | 通过协议转换代理，`ANTHROPIC_MODEL=deepseek-chat` |
| 内置免费渠道 | 在 Web 界面设置中选择 Claude CN 或 Kilo Free |

免费渠道依赖上游服务，模型目录、额度、限流和可用性可能变化，不承诺永久免费或固定可用率。

---

## 部署到 Hugging Face Spaces

镜像兼容 Hugging Face Spaces 的 Docker Space 部署：

- Space 使用本镜像作为基础镜像即可，持久化卷由 Spaces 自动挂载到 `/data/`
- 数据目录由 `entrypoint.sh` 在容器启动时创建，构建阶段不写入持久卷，符合 Spaces 约束
- 在 Space 的 Settings > Repository Secrets 中设置 `CLAUDE_H5_TOKEN`（自定义访问令牌）

---

## 常见问题

### PHP 页面 502 Bad Gateway

检查 Nginx 用户和 PHP-FPM socket 权限是否匹配。默认配置下 Nginx 以 `www-data` 运行，应与 PHP-FPM socket 的 owner/group 一致。

### 如何修改 PHP 配置？

在容器的 `/etc/php/7.4/fpm/php.ini` 中修改，或通过 entrypoint 脚本添加自定义配置。

### 令牌登录失败

- 确认环境变量 `CLAUDE_H5_TOKEN` 已设置
- 如果持久化目录已有旧的 `settings.json`，删除后重启容器会重新生成
- 检查 nginx 日志中是否有 CORS 相关的 403 错误

### 容器启动后服务无响应

- 容器健康检查每 30 秒检测 `/health` 端点
- 查看日志：`docker logs claude-cn`
- 启动顺序：php-fpm → nginx → claude-cn（可能需要 5-10 秒完成初始化）
