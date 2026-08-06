# Claude Code Cn

<p align="center">
  <img src="docs/images/app-icon.svg" alt="Claude Code Cn" width="240">
</p>

<div align="center">

[![GitHub Stars](https://img.shields.io/github/stars/host1syan/claude-cn?style=social)](https://github.com/host1syan/claude-cn/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/host1syan/claude-cn?style=social)](https://github.com/host1syan/claude-cn/network/members)
[![GitHub Issues](https://img.shields.io/github/issues/host1syan/claude-cn)](https://github.com/host1syan/claude-cn/issues)
[![License](https://img.shields.io/github/license/host1syan/claude-cn)](https://github.com/host1syan/claude-cn/blob/main/LICENSE)
[![中文](https://img.shields.io/badge/中文-Available-green)](README.md)
[![English](https://img.shields.io/badge/English-Current-blue)](README.en.md)

</div>

Claude Code Cn is a Claude-like agent workbench built around Chinese prompts. It runs on macOS, Windows, and Linux and provides a terminal CLI, local server, and desktop app. Multi-session management, projects, branch and Worktree isolation, code diffs, permission review, model providers, Computer Use, H5 remote access, IM adapters, and scheduled tasks live in one workspace.

Chinese-first prompts reduce language switching in Chinese development workflows. Actual token usage depends on model, task complexity, and context. Model, upstream gateway, and platform policies still apply.

## Highlights

- **Chinese-first agent workflow**: core task-management, verification-agent, and tool prompts use Chinese.
- **Claude CN channel**: built-in Claude CN provider configuration with free-model routing available in current builds.
- **Kilo Free channel**: built-in Kilo Free provider, including free choices such as `stepfun/step-3.7-flash:free`.
- **Prompt cache support**: supports Anthropic prompt-cache fields and cache usage mapping so stable long-task context can reuse upstream cache.
- **RTK and Caveman direction**: tool-output and model-output compression are project directions; availability depends on build and runtime configuration.
- **Multi-session desktop**: tabs, projects, session history, terminals, and background tasks.
- **Branch and Worktree isolation**: start sessions on a selected branch in current tree or isolated Worktree.
- **Visual code review**: inspect edits, changed files, line counts, diffs, and workspace state.
- **Permission workflow**: review risky commands, tool calls, and model questions in desktop app.
- **Multiple model providers**: Anthropic-compatible APIs, third-party models, protocol proxies, and local models.
- **Memory and multi-agent systems**: persistent memory, agent orchestration, parallel tasks, Teams, and Skills.
- **Computer Use**: authorized screenshots, clicks, typing, and desktop-app control.
- **H5 and IM access**: remote sessions through browser, Telegram, Feishu, WeChat, and DingTalk.
- **Scheduled tasks and usage statistics**: plan work and inspect local token trends.

## Providers and Caching

Claude CN and Kilo Free depend on upstream services. Free-model catalogs, quotas, rate limits, latency, and availability can change. This project does not promise permanent free access, unlimited quota, or fixed uptime. See [Third-Party Models](docs/en/guide/third-party-models.md).

The runtime supports prompt-cache protocol fields and maps values such as `cache_read_input_tokens` and `cache_creation_input_tokens`. Stable system prompts, tool lists, and request prefixes improve cache reuse in long tasks. Actual hit rate depends on upstream model, cache TTL, request structure, and context changes; no fixed percentage is guaranteed.

RTK-style tool-output compression and Caveman-style model-output compression are project directions. Check current settings, build artifacts, and observed runtime behavior before relying on them.

## Install Desktop App

1. Download macOS, Windows, or Linux installer from [Releases](https://github.com/host1syan/claude-cn/releases).
2. Configure provider, API key, and default model in Settings.
3. Temporary macOS builds may require manual approval. Unsigned Windows installers may trigger SmartScreen. See [Desktop Installation](docs/en/desktop/04-installation.md).

## Run from Source

```bash
bun install
cp .env.example .env

# Start terminal CLI
./bin/claude-cn

# Start local server on default port 3456
SERVER_PORT=3456 bun run src/server/index.ts

# Start desktop app
cd desktop && bun install && bun run dev
```

See [Environment Variables](docs/en/guide/env-vars.md), [Global Usage](docs/en/guide/global-usage.md), and [Quick Start](docs/en/guide/quick-start.md).

## Documentation

| Document | Description |
|------|------|
| [Quick Start](docs/en/guide/quick-start.md) | Install, launch, and choose provider |
| [Environment Variables](docs/en/guide/env-vars.md) | Complete environment-variable reference |
| [Third-Party Models](docs/en/guide/third-party-models.md) | Anthropic, OpenAI, DeepSeek, Ollama, and built-in free providers |
| [Memory System](docs/en/memory/01-usage-guide.md) | Cross-session persistent memory |
| [Multi-Agent System](docs/en/agent/01-usage-guide.md) | Agent orchestration, parallel tasks, and Teams |
| [Skills System](docs/en/skills/01-usage-guide.md) | Extensible capabilities and custom workflows |
| [Computer Use](docs/en/features/computer-use.md) | Screenshot, mouse, and keyboard control |
| [Desktop App](docs/en/desktop/) | GUI client, installation, and build |
| [FAQ](docs/en/guide/faq.md) | Common troubleshooting |
| [Project Structure](docs/en/reference/project-structure.md) | Source layout |

## Technology

| Category | Technology |
|------|------|
| Language | TypeScript |
| Desktop shell | Electron with compiled sidecar runtime |
| Desktop UI | React, Vite, Tailwind CSS |
| State management | Zustand |
| Runtime | [Bun](https://bun.sh) |
| Terminal UI | React, Ink |
| CLI parsing | Commander.js |
| API | Anthropic SDK |
| Protocols | MCP, LSP |
| IM adapters | Telegram, Feishu, WeChat, DingTalk |

## Docker Deployment

Docker Hub image: [`host1syan/claude-cn`](https://hub.docker.com/r/host1syan/claude-cn). It provides browser-based Web UI, H5 remote access, Nginx, PHP 7.4, and optional WebDAV file management. Bun and Node.js are not required on the host.

### Pull and Run

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

Open `http://localhost:3456` and sign in with `CLAUDE_H5_TOKEN`. WebDAV is enabled only when both `WEBDAV_USER` and `WEBDAV_PASSWORD` are set.

### Docker Configuration

| Variable | Description |
|------|------|
| `CLAUDE_H5_TOKEN` | H5 access token and Web UI login credential |
| `WEBDAV_USER` / `WEBDAV_PASSWORD` | WebDAV credentials; enabled when both are set |
| `ANTHROPIC_AUTH_TOKEN` | API authentication token |
| `ANTHROPIC_BASE_URL` | Anthropic-compatible gateway URL |
| `ANTHROPIC_MODEL` | Default model |
| `CLAUDE_CONFIG_DIR` | Config, sessions, Skills, and plugin directory; `/data/.claude` in the container |
| `API_TIMEOUT_MS` | API timeout; default `3000000` |

Mount `/data/` to a Docker volume or host directory to preserve configuration, sessions, Skills, plugins, and PHP files across container replacement. See [Docker Usage](docker/USAGE.md) and [Docker Image Guide](DOCKER_README.md) for WebDAV, PHP, Hugging Face Spaces, persistence, and troubleshooting.

## Disclaimer

This project is rebuilt from Claude Code source. Original copyrights belong to their respective owners. It is provided for learning and research. Free providers, third-party models, and upstream services follow their own terms; users are responsible for usage, data security, and compliance.
