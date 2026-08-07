# 桌面端常见问题

## Q：报错 `Failed to authenticate. API Error: 401`

**原因**：API 认证失败。第三方 Provider（如 Kimi）可能通过 `x-api-key` 请求头发送密钥，而配置写成了 `Bearer` 令牌头；或 Provider 配置未正确注入。

**排查**：

1. 在设置中确认选择的 Provider，是「直连 API Key」还是「Auth Token」方式。
2. 检查 `~/.claude/claude-cn/settings.json` 中的 `env` 配置是否正确，例如使用 Kimi 时：
   ```json
   {
     "env": {
       "ANTHROPIC_BASE_URL": "https://api.kimi.com/coding/",
       "ANTHROPIC_API_KEY": "sk-kimi-...",
       "ANTHROPIC_MODEL": "kimi-for-coding"
     }
   }
   ```
3. 修改后重启桌面端，重新发起会话。

> 区分：`ANTHROPIC_API_KEY` 以 `x-api-key` 头发送，`ANTHROPIC_AUTH_TOKEN` 以 `Authorization: Bearer <token>` 发送。多数第三方 Provider 期望 `x-api-key`。

## 更多问题

通用错误排查见[常见问题](../guide/faq.md)。安装相关问题见[安装指南](./04-installation.md)。