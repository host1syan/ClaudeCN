# 常见问题

## Q：`undefined is not an object (evaluating 'usage.input_tokens')`

**原因**：`ANTHROPIC_BASE_URL` 配置不正确，API 端点返回的不是 Anthropic 协议格式的 JSON，而是 HTML 页面或其他格式。

本项目使用 **Anthropic Messages API 协议**，`ANTHROPIC_BASE_URL` 必须指向一个兼容 Anthropic `/v1/messages` 接口的端点。Anthropic SDK 会自动在 base URL 后拼接 `/v1/messages`，例如：

- MiniMax：`ANTHROPIC_BASE_URL=https://api.minimaxi.com/anthropic` ✅
- OpenRouter：`ANTHROPIC_BASE_URL=https://openrouter.ai/api` ✅
- OpenRouter 错误写法：`ANTHROPIC_BASE_URL=https://openrouter.ai/anthropic` ❌（返回 HTML）

如果模型供应商只支持 OpenAI 协议，需要通过 LiteLLM 等代理做协议转换，详见[第三方模型使用指南](./third-party-models.md)。

## Q：怎么接入 OpenAI / DeepSeek / Ollama 等非 Anthropic 模型？

本项目使用 **Anthropic Messages API 协议**。如果模型供应商不直接支持该协议，需要用 [LiteLLM](https://github.com/BerriAI/litellm) 等代理做协议转换（OpenAI → Anthropic）。详细配置见[第三方模型使用指南](./third-party-models.md)。

## Q：模型无法回复或超时？

- 确认 `ANTHROPIC_BASE_URL`、认证令牌和模型名配置正确
- 确认网络可以访问上游 API 端点
- API 超时可通过 `API_TIMEOUT_MS` 调整

## Q：免费渠道（Claude CN / Kilo Free）不可用？

免费渠道依赖上游服务，模型目录、额度、限流和可用性可能变化，可能出现暂时不可用或限流。建议配置备用渠道或使用自己的 API。

## Q：桌面端安装后启动异常？

- **macOS**：旧版或未签名临时包可能在首次打开报"已损坏"或"无法验证开发者"。可执行 `xattr -cr /Applications/Claude\ Code\ Cn.app`，或在「系统设置 → 隐私与安全性」点"仍要打开"。
- **Windows**：未签名安装包可能触发 SmartScreen，点击「更多信息」→「仍要运行」。
- 详见[桌面端安装指南](../desktop/04-installation.md)。

## Q：H5 访问登录失败？

- 确认 H5 Token 已正确设置且与访问时输入一致
- 在可控网络/域名范围内使用，H5 不是公开 SaaS 登录系统
- 详见[H5 访问](../desktop/06-h5-access.md)。

## Q：Docker 部署相关问题？

Web 界面、WebDAV、PHP 等排查见[Docker 使用文档](../../docker/USAGE.md)。