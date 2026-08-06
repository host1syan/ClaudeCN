# Quick Start

ClaudeCN provides a CLI, desktop app, and local server. Install Bun first, then choose a model provider.

## 1. Install Bun

```bash
# macOS / Linux
curl -fsSL https://bun.sh/install | bash

# macOS (Homebrew)
brew install bun

# Windows (PowerShell)
powershell -c "irm bun.sh/install.ps1 | iex"
```

> On minimal Linux images, if you see `unzip is required`, run `apt update && apt install -y unzip` first.

## 2. Install Dependencies

```bash
bun install
cp .env.example .env
```

## 3. Choose a Provider

You can use your own Anthropic-compatible API or select a built-in provider in the desktop Settings:

- **Claude CN**: built-in provider configuration with free-model routing available in current builds.
- **Kilo Free**: Kilo Gateway free-model channel. Its default model is `stepfun/step-3.7-flash:free`.

Free providers depend on upstream services. Catalogs, quota, rate limits, and availability can change. Do not treat them as permanent, unlimited, or guaranteed services. See [Third-Party Models](./third-party-models.md) for custom providers and proxies.

For custom providers, edit `.env` with your credentials. See [Environment Variables](./env-vars.md) for the full reference.

## 4. Start

### macOS / Linux

```bash
./bin/claude-cn                          # Interactive TUI mode
./bin/claude-cn -p "your prompt here"    # Headless mode
./bin/claude-cn --help                   # Show all options
```

### Windows

> **Prerequisite**: [Git for Windows](https://git-scm.com/download/win) must be installed.

```powershell
# PowerShell / cmd — call Bun directly
bun --env-file=.env ./src/entrypoints/cli.tsx

# Or run inside Git Bash
./bin/claude-cn
```

## 5. Start the Desktop App

The desktop app uses the local server:

```bash
SERVER_PORT=3456 bun run src/server/index.ts
cd desktop && bun install && bun run dev
```

## 6. Global Usage (Optional)

Add `bin/` to your PATH to run from any directory. See [Global Usage Guide](./global-usage.md):

```bash
export PATH="$HOME/path/to/claude-code-cn/bin:$PATH"
```

## 7. Recovery Mode

If the Ink TUI has issues, use Recovery CLI mode:

```bash
CLAUDE_CODE_FORCE_RECOVERY_CLI=1 ./bin/claude-cn
```

## Cache Note

The runtime supports Anthropic prompt-cache fields and cache usage reporting. Stable system prompts, tool lists, and request prefixes are more reusable in long tasks. Actual hits depend on provider, cache TTL, request structure, and context changes; no fixed hit rate is guaranteed.
