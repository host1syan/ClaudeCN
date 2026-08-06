# QQ 接入

> QQ Bot Adapter 的接入教程。使用腾讯官方 QQ Bot API，支持私聊和群聊。

## 适用场景

QQ 方案适合在中国区环境下通过 QQ 机器人与 Claude Code 对话。当前实现支持私聊和群聊。

实现入口：`adapters/qq/index.ts`

## 1. 扫码绑定（推荐）

打开桌面端 `设置 → IM 接入 → QQ`，点击「扫码绑定」按钮。

使用 QQ 扫描二维码，确认绑定后，系统会自动获取 AppID 和 AppSecret。

## 2. 手动配置（可选）

如果你已经有 AppID 和 AppSecret，可以手动填写：

### 2.1 在桌面端填写

打开桌面端 `设置 → IM 接入 → QQ`，把 AppID 和 AppSecret 填进去。

### 2.2 生成配对码

点击「生成配对码」按钮，得到 6 位码。

**记得点保存！！**

## 3. 机器人与桌面端配对

随便给刚才创建的机器人发送一条消息，按提示把 6 位配对码发给它。

看到配对成功提示后，就可以用 QQ 远程驱动桌面端 Claude Code 了。

## 支持的命令

QQ Bot 支持以下文本命令：

- `/help` 或 `帮助` — 显示帮助
- `/status` 或 `状态` — 显示当前状态
- `/clear` 或 `清空` — 清空会话上下文
- `/projects` 或 `项目列表` — 列出最近项目
- `/new` 或 `新会话` — 开启新对话
- `/stop` 或 `停止` — 停止生成

## 权限审批

当 Claude 请求敏感权限时，adapter 会在 QQ 里发送权限请求，你可以回复：

- `/allow <id>` — 允许工具执行
- `/deny <id>` — 拒绝工具执行

## 消息限制

- 单条消息最大 2000 字符，超长自动分片
- 支持图片、文件、语音（amr/silk 格式）

## 启动 adapter

桌面端会自动把 adapter 作为 sidecar 拉起。如果你在本地开发，需要手动启动：

```bash
cd adapters
bun install
bun run qq
```

## 环境变量覆盖（可选）

```bash
export QQ_APP_ID="your_app_id"
export QQ_APP_SECRET="your_app_secret"
export ADAPTER_SERVER_URL="ws://127.0.0.1:3456"
```

## 常见问题

### 收不到消息

优先检查：

- 机器人是否已发布
- 是否真的是和 bot 的私聊，而不是群聊
- 群聊中是否 @ 了机器人

### 权限按钮点了没反应

检查配对码是否仍在有效期内，以及是否已正确配对。

### 一直提示未授权

- 配对码是否仍在 60 分钟有效期内
- 发的是不是桌面端当前这一枚（重新生成后旧的立即失效）
- `qq.pairedUsers` 里是否已经写入当前用户 ID

## 源码入口

- `adapters/qq/index.ts`
- `adapters/qq/protocol.ts`
- `adapters/qq/format.ts`
- `adapters/qq/media.ts`
- `adapters/common/pairing.ts`
- `adapters/common/session-store.ts`
- `adapters/common/ws-bridge.ts`
