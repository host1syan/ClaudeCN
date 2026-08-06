# claude-cn Docker 使用文档

## 概述

基于 [claude-cn](https://github.com/host1syan/claude-cn) 改造的 Docker 镜像，将原本的桌面端 Claude Code 包装为 **Web 可访问的服务**，集成了 Nginx + PHP 7.4 环境，支持一键部署到 Hugging Face Spaces。

### 主要特性

- 🌐 **Web 访问** — 浏览器即可使用，无需安装桌面客户端
- 🐘 **PHP 7.4 环境** — `/php/` 路由托管 PHP 网站，与 claude-cn 共享数据目录
- 📁 **WebDAV 文件管理** — `/webdav/` 路由将 `/data/` 目录映射出来，可挂载到本地电脑
- 🔄 **断线重连** — API 不稳定时自动指数退避重连，不丢失任务
- 🛡️ **零权限打扰** — Docker 模式自动跳过所有权限询问，后台静默运行
- 🔗 **H5 Token 认证** — 简单的访问令牌保护，环境变量可覆盖
- 📦 **Hugging Face Spaces 兼容** — 持久化目录运行时创建，构建时不写数据

---

## 快速开始

### 构建镜像

```bash
docker build --no-cache -t claude-cn /path/to/claude-cn
```

### 运行容器

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v /path/to/data:/data \
  claude-cn
```

然后打开浏览器访问 `http://localhost:3456`，输入令牌 `123` 即可。

---

## 环境变量

| 变量 | 默认值 | 说明 |
|---|---|---|
| `CLAUDE_H5_TOKEN` | `123` | H5 访问令牌。Web UI 登录时使用，可通过 `-e` 覆盖 |
| `WEBDAV_USER` | — | （可选）WebDAV 用户名。设置后启用 `/webdav/` 路由 |
| `WEBDAV_PASSWORD` | — | （可选）WebDAV 密码。与 `WEBDAV_USER` 同时设置才生效 |
| `CC_CN_DOCKER_MODE` | `1`（固定） | 启用 Docker 模式优化（权限跳过、断线保持、CORS 放行） |
| `SERVER_HOST` | `0.0.0.0` | claude-cn 服务器监听地址 |
| `SERVER_PORT` | `3457` | claude-cn 内部端口（nginx 反向代理此端口） |
| `HOME` | `/data/php` | 默认工作目录，也是 PHP 网站文件根目录 |
| `CLAUDE_CONFIG_DIR` | `/data/.claude` | claude-cn 持久化配置、会话、Skills、插件等 |
| `IS_SANDBOX` | `1`（固定） | 绕过 `--dangerously-skip-permissions` root 用户限制 |
| `DISABLE_TELEMETRY` | `1` | 禁用遥测 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `1` | 减少非必要网络请求 |

### CLI 子进程环境变量

以下变量自动注入到 CLI 子进程，无需手动配置：

| 变量 | 值 | 说明 |
|---|---|---|
| `CLAUDE_CODE_UNATTENDED_RETRY` | `1` | 启用无值守重试模式 |
| `CLAUDE_CODE_MAX_RETRIES` | `20` | API 最大重试次数 |

---

## 端口说明

| 端口 | 用途 | 说明 |
|---|---|---|
| `3456` | nginx 入口 | 外部访问端口，Web UI + API + PHP |
| `3457` | claude-cn 内部 | nginx 反向代理，不直接暴露 |

### URL 路由

| 路径 | 目标 | 说明 |
|---|---|---|
| `/` | claude-cn Web UI | Claude Code 的 Web 界面 |
| `/php/` | PHP 7.4 | PHP 网站（文件位于 `/data/php/`） |
| `/webdav/` | WebDAV | 文件管理（需设置 `WEBDAV_USER` + `WEBDAV_PASSWORD`） |
| `/health` | claude-cn 健康检查 | 容器健康检查 |

---

## 持久化挂载

所有持久数据存储在 `/data/` 目录下，需挂载外部卷：

```bash
-v /host/path/data:/data
```

| 路径 | 用途 |
|---|---|
| `/data/php/` | PHP 网站文件 |
| `/data/.claude/claude-cn/settings.json` | claude-cn 配置文件（含 H5 token） |
| `/data/.claude/projects/` | 项目会话数据 |
| `/data/.claude/skills/` | 安装的 Skills |
| `/data/.claude/plugins/` | 安装的插件 |
| `/data/.claude/claude-cn/providers.json` | API 提供商配置 |
| `/data/.claude/claude-cn/oauth-*.json` | OAuth 令牌 |

> **注意**：首次启动时会自动创建上述目录结构和默认配置。无需在构建镜像时预写数据，兼容 Hugging Face Spaces 的构建约束。

---

## 改造优化详情

### 1. 🔌 WebSocket 断线不终止任务

**文件**: `src/server/ws/handler.ts`

当 Web 页面关闭（WebSocket 断开）时，原有逻辑会启动 30 秒清理定时器销毁会话。Docker 模式下此定时器被跳过，CLI 子进程继续在后台运行。

```typescript
const dockerMode = process.env.CC_CN_DOCKER_MODE === '1'
// Docker 模式跳过 cleanup — WebSocket 断开不停止任务
```

### 2. 🛡️ 零权限弹窗（四层防护）

| 层级 | 文件 | 机制 |
|---|---|---|
| 1. CLI 参数 | `conversationService.ts:888` | Docker 模式自动追加 `--dangerously-skip-permissions` |
| 2. Root 绕过 | `Dockerfile` `ENV IS_SANDBOX=1` | 绕过 CLI 的 root 用户检查 |
| 3. SDK 自动批准 | `conversationService.ts:692` | SDK 通道的 `can_use_tool` 请求自动 respond |
| 4. WS 过滤 | `ws/handler.ts:1218` | 到达 WebSocket 层的权限请求直接丢弃 |

### 3. 🔄 API 断线指数退避重连

**文件**: `src/services/api/withRetry.ts`

移除了 `isPersistentRetryEnabled()` 中的 `feature('UNATTENDED_RETRY')` 门禁，使持久重试模式始终可用（桌面端需登录后才启用）。

配合子进程环境变量（见上方表格），API 请求在遇到网络错误/限流时按以下间隔自动重试（在 `withRetry.ts` 中实现）：

1. 5 秒 ✓
2. 30 秒 ✓  
3. 60 秒 ✓  
4. 5 分钟 ✓
5. 之后固定 5 分钟间隔，最多 20 次

### 4. 🔗 H5 Token 三层验证

**文件**: `src/server/services/h5AccessService.ts`

| 层级 | 机制 |
|---|---|
| 1. 环境变量 | `validateToken()` 优先检查 `CLAUDE_H5_TOKEN` 环境变量 |
| 2. 持久化文件 | 启动时 `initFromEnv()` 将 token hash 预写入 `settings.json` |
| 3. Entrypoint 预写 | `entrypoint.sh` 在容器启动时计算 SHA-256 hash 写入持久化路径 |

### 5. 🌐 CORS 跨域修复

**文件**: `src/server/index.ts`

H5 token 页面登录时`403` — 根因是 CORS 中间件拒绝非 localhost 的 origin。Docker 模式下 `isOriginAllowed` 回调放过所有 origin。

### 6. 🐘 Nginx + PHP 7.4 集成

- **Nginx**: 路由 `/php/` 至 PHP-FPM，其余路由至 claude-cn
- **PHP 7.4**: 已安装扩展 `gd`、`zip`、`bcmath`、`curl`
- **Socket**: Nginx 以 `www-data` 用户运行，与 PHP-FPM socket 权限一致

### 7. 📁 WebDAV 远程文件管理

通过 Nginx 的 WebDAV 模块将 `/data/` 目录暴露为网络磁盘，可直接挂载到本地电脑管理文件。

#### 启用方式

设置两个环境变量即可启用（不设置则不启动 WebDAV，无安全风险）：

```bash
docker run -d \
  --name claude-cn \
  -p 3456:3456 \
  -v /path/to/data:/data \
  -e WEBDAV_USER=admin \
  -e WEBDAV_PASSWORD=your_password \
  claude-cn
```

#### 挂载到本地

| 系统 | 方法 |
|---|---|
| **macOS Finder** | 菜单栏 → 前往 → 连接服务器 → `http://your-host:3456/webdav/` → 输入用户名密码 |
| **Windows 资源管理器** | 右键"此电脑" → 映射网络驱动器 → `http://your-host:3456/webdav/` |
| **Linux** | `sudo mount -t davfs http://your-host:3456/webdav/ /mnt/webdav`（需安装 `davfs2`） |
| **第三方工具** | Rclone、Cyberduck、WinSCP 等都支持 WebDAV 协议 |

> **安全提示**：WebDAV 使用 HTTP Basic Auth 认证，建议在内网或使用 HTTPS 代理时启用。如果通过公网访问，建议搭配 IP 白名单或 VPN 使用。

### 8. 🏗️ 基础设施选型

- **Runtime 基础镜像**: `ubuntu:22.04`（而非 `oven/bun:1-slim`）
- **PHP 安装源**: `ppa:ondrej/php`（Ondřej Surý 维护）
- **Bun 安装**: 官方安装脚本（非 multi-stage copy），避免 glibc 版本兼容问题

> **背景**: `oven/bun:1-slim` 基于 Debian Trixie（testing），其 `libzip4`→`libzip4t64` 重命名与 sury.org 的 Bookworm PHP 包不兼容。切换至 Ubuntu 22.04 + ondrej/php PPA 后完全解决。

---

## 部署到 Hugging Face Spaces

### Docker Space 配置

在 Space 的 `Dockerfile` 中直接包含本项目的 Dockerfile。

### 持久化存储

Hugging Face Spaces 自动挂载持久卷到 `/data/`。所有数据在容器启动时由 `entrypoint.sh` 创建目录结构，**构建阶段不会写入持久卷**，符合 Spaces 约束。

### 环境变量配置

在 HF Spaces 的 Settings > Repository Secrets 中设置：

| 变量 | 建议值 |
|---|---|
| `CLAUDE_H5_TOKEN` | 自定义访问令牌 |

---

## 常见问题

### PHP 页面 502 Bad Gateway

检查 Nginx 用户和 PHP-FPM socket 权限是否匹配。默认配置下 Nginx 以 `www-data` 运行，应与 PHP-FPM socket 的 owner/group 一致。

### 如何修改 PHP 配置？

在容器的 `/etc/php/7.4/fpm/php.ini` 中修改，或通过 entrypoint 脚本添加自定义配置。

### 如何添加更多 PHP 扩展？

修改 Dockerfile 中 `php7.4-*` 包列表，重新构建镜像。

### 令牌登录失败

- 确保环境变量 `CLAUDE_H5_TOKEN` 已设置
- 如果持久化目录已有旧的 `settings.json`，删除后重启容器会重新生成
- 检查 nginx 日志是否有 CORS 相关的 403 错误

### 容器启动后服务无响应

- 容器健康检查每 30 秒检测 `/health` 端点
- 查看日志：`docker logs claude-cn`
- 启动顺序：php-fpm → nginx → claude-cn（可能需要 5-10 秒完成初始化）
