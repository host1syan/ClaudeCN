# 快速开始

Claude Code Cn 提供 CLI、桌面端和本地服务端三种入口。首次使用先安装 Bun，再选择模型渠道。

## 1. 安装 Bun

```bash
# macOS / Linux
curl -fsSL https://bun.sh/install | bash

# macOS（Homebrew）
brew install bun

# Windows（PowerShell）
powershell -c "irm bun.sh/install.ps1 | iex"
```

> 精简版 Linux 如提示 `unzip is required`，先运行 `apt update && apt install -y unzip`。

## 2. 安装依赖

```bash
bun install
cp .env.example .env
```

## 3. 选择模型渠道

可以使用自己的 Anthropic 兼容 API，也可以在桌面端设置中选择内置渠道：

- **Claude CN**：项目内置渠道配置，提供当前版本可用的免费模型路由。
- **Kilo Free**：Kilo Gateway 免费模型渠道，默认模型为 `stepfun/step-3.7-flash:free`。

免费渠道依赖上游服务，模型目录、额度、限流和可用性可能变化。不要把免费渠道当作永久、无限或固定可用的服务。第三方模型和代理配置见[第三方模型](./third-party-models.md)。

使用自有渠道时，编辑 `.env` 填入 API 凭据，完整变量说明见[环境变量配置](./env-vars.md)。

## 4. 启动

### macOS / Linux

```bash
./bin/claude-cn                          # 交互 TUI 模式
./bin/claude-cn -p "your prompt here"    # 无头模式
./bin/claude-cn --help                   # 查看所有选项
```

### Windows

> **前置要求**：必须安装 [Git for Windows](https://git-scm.com/download/win)。

```powershell
# PowerShell / cmd 直接调用 Bun
bun --env-file=.env ./src/entrypoints/cli.tsx

# 或在 Git Bash 中运行
./bin/claude-cn
```

## 5. 启动桌面端

桌面端需要本地服务端：

```bash
SERVER_PORT=3456 bun run src/server/index.ts
cd desktop && bun install && bun run dev
```

## 6. 全局使用（可选）

将 `bin/` 加入 PATH 后，可在任意目录启动。详见[全局使用指南](./global-usage.md)：

```bash
export PATH="$HOME/path/to/claude-code-cn/bin:$PATH"
```

## 7. 降级模式

如果 Ink TUI 出现问题，可以使用 Recovery CLI 模式：

```bash
CLAUDE_CODE_FORCE_RECOVERY_CLI=1 ./bin/claude-cn
```

## 缓存提示

项目支持 Anthropic Prompt Cache 相关字段和缓存用量统计。长任务中，稳定系统提示词、工具列表和请求前缀更容易复用缓存；实际命中取决于模型渠道、缓存 TTL、请求结构和上下文变化，不承诺固定命中率。
