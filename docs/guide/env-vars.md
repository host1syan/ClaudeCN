# 环境变量参考

以下环境变量用于配置模型渠道、行为开关等。多数变量可在**桌面端设置**中直接配置；命令行或服务端场景下，也可通过环境变量或配置文件指定。

## 常用变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `ANTHROPIC_API_KEY` | 二选一 | API Key，通过 `x-api-key` 头发送 |
| `ANTHROPIC_AUTH_TOKEN` | 二选一 | Auth Token，通过 `Authorization: Bearer` 头发送 |
| `ANTHROPIC_BASE_URL` | 否 | 自定义 API 端点，默认 Anthropic 官方 |
| `ANTHROPIC_MODEL` | 否 | 默认模型 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 否 | Sonnet 级别模型映射 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 否 | Haiku 级别模型映射 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 否 | Opus 级别模型映射 |
| `API_TIMEOUT_MS` | 否 | API 请求超时，默认 `600000` (10 分钟) |
| `DISABLE_TELEMETRY` | 否 | 设为 `1` 禁用遥测 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 否 | 设为 `1` 禁用非必要网络请求 |

> API Key 与 Auth Token 只需二选一。模型供应商只支持 OpenAI 协议时，需通过 LiteLLM 等代理转换为 Anthropic 协议，见[第三方模型](./third-party-models.md)。

## 配置方式

### 方式一：配置文件

在 `~/.claude/settings.json` 中写入环境变量：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-xxx",
    "ANTHROPIC_BASE_URL": "https://api.minimaxi.com/anthropic",
    "ANTHROPIC_MODEL": "MiniMax-M3"
  }
}
```

### 方式二：环境变量

在命令行启动前通过 export 或进程环境注入：

```bash
export ANTHROPIC_BASE_URL=https://api.minimaxi.com/anthropic
export ANTHROPIC_AUTH_TOKEN=sk-xxx
export ANTHROPIC_MODEL=MiniMax-M3
```

> 配置优先级：进程环境变量 > 配置文件。