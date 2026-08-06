# 剔除历史 Thinking 块 — 上下文浪费修复

## 问题

使用思考模型（DeepSeek R1、Claude Opus 4 thinking 等）时，上下文增长极快，远超非思考模型。原因：**每一轮对话中模型产出的 thinking/redacted_thinking 块会累积在消息历史中**，后续请求时全部发送给 API，但除了最后一条 assistant 的 thinking 用于延续推理链，**历史 thinking 块对后续对话无任何价值**，纯粹浪费上下文预算。

## 影响

| 指标 | 非思考模型 | 思考模型（修复前） |
|------|-----------|----------------|
| 10 轮对话上下文量 | ~5K tokens | ~30-50K tokens |
| auto-compact 触发频率 | 正常 | 频繁过早触发 |
| 前端上下文格子图 | 准确 | 显示虚高 |

## 修复原理

三路径对齐，确保 token 统计口径一致：

```
发送给 API 的消息      前端上下文格子统计      auto-compact 触发阈值
    │                       │                       │
    ▼                       ▼                       ▼
normalizeMessagesForAPI  approximateMessageTokens  tokenCountWithEstimation
    │                       │                       │
    ▼                       ▼                       ▼
filterHistoricalThinking  normalizeMessagesForAPI  roughTokenCountEstimation
FromAssistantMessages          │                   ForContent
    │                       (同一份过滤)               │
    ▼                                               ▼
API payload 不含                                    跳过 thinking 块
历史 thinking                                       计数
```

## 涉及文件

### 文件 1：`src/utils/messages.ts`

**新增函数** `filterHistoricalThinkingFromAssistantMessages()`（位于 `isThinkingBlock` 函数之后）：

```typescript
export function filterHistoricalThinkingFromAssistantMessages(
  messages: (UserMessage | AssistantMessage)[],
): (UserMessage | AssistantMessage)[] {
  // 找到最后一条 assistant 消息 — 保留它的 thinking
  let lastAssistantIdx = -1
  for (let i = messages.length - 1; i >= 0; i--) {
    if (messages[i]!.type === 'assistant') {
      lastAssistantIdx = i
      break
    }
  }

  return messages.map((msg, i) => {
    if (msg.type !== 'assistant' || i === lastAssistantIdx) {
      return msg  // 保留最后一条的 thinking
    }

    const content = msg.message.content
    const hasThinking = content.some(isThinkingBlock)
    if (!hasThinking) return msg  // 没有 thinking 块，跳过

    const filteredContent = content.filter(block => !isThinkingBlock(block))

    // 全是 thinking 块 => 替换为占位文本
    if (filteredContent.length === 0) {
      return {
        ...msg,
        message: {
          ...msg.message,
          content: [{ type: 'text' as const, text: '[Thinking removed]' }],
        },
      }
    }

    return {
      ...msg,
      message: {
        ...msg.message,
        content: filteredContent,
      },
    }
  })
}
```

**逻辑**：
1. 从后往前遍历消息列表，找到最后一条 assistant 消息
2. 对除最后一条外的所有 assistant 消息：
   - 如果有 thinking/redacted_thinking 块，移除它们
   - 如果移除后内容为空，插入 `[Thinking removed]` 占位文本（避免空消息导致 API 400）
3. 最后一条 assistant 消息原样保留（它的 thinking 用于延续推理链）

**在 `normalizeMessagesForAPI()` 中接入**（在 `reorderAssistantToolUseBlocks` 之后、`filterTrailingThinkingFromLastAssistant` 之前）：

```typescript
  // 在 reorder 之后插入
  const withFilteredHistoricalThinking =
    filterHistoricalThinkingFromAssistantMessages(withReorderedToolUse)

  // 原有逻辑，输入从 withReorderedToolUse 改为 withFilteredHistoricalThinking
  const withFilteredThinking =
    filterTrailingThinkingFromLastAssistant(withFilteredHistoricalThinking)
```

### 文件 2：`src/services/tokenEstimation.ts`

**修改函数** `roughTokenCountEstimationForContent()` — 在 content block 循环中跳过 thinking 块：

```typescript
function roughTokenCountEstimationForContent(
  content:
    | string
    | Array<Anthropic.ContentBlock>
    | Array<Anthropic.ContentBlockParam>
    | undefined,
): number {
  if (!content) {
    return 0
  }
  if (typeof content === 'string') {
    return roughTokenCountEstimation(content)
  }
  let totalTokens = 0
  for (const block of content) {
    // 新增：跳过 thinking/redacted_thinking 块
    if ('type' in block && (block.type === 'thinking' || block.type === 'redacted_thinking')) {
      continue
    }
    totalTokens += roughTokenCountEstimationForBlock(block)
  }
  return totalTokens
}
```

**为什么需要这个修改**：`tokenCountWithEstimation()` → `roughTokenCountEstimationForMessages()` 是 auto-compact 触发和 token 估算的底层函数，它不经过 `normalizeMessagesForAPI`。不同步的话，auto-compact 会因历史 thinking 块的 token 而**过早触发**压缩。

## 部署步骤

### 1. 确认代码版本

当前工作基于 claude-cn 项目，入口文件为 `bin/claude-cn`，使用 Bun 运行时。确认目标系统代码结构一致：

```bash
ls src/utils/messages.ts src/services/tokenEstimation.ts
```

### 2. 打补丁（手动 Patch）

**方式 A — 直接编辑（推荐）：**

按上方代码修改两个文件。

**方式 B — git patch：**

```bash
git diff src/utils/messages.ts src/services/tokenEstimation.ts > thinking-filter.patch
```

然后将 `.patch` 文件复制到目标机器，目标机器上：

```bash
cd /path/to/claude-cn
git apply thinking-filter.patch
```

### 3. 验证编译

```bash
bunx tsc --noEmit --project tsconfig.json 2>&1 | grep -v "bun-types\|baseUrl"
```

预期无新增错误（"bun-types" 和 "baseUrl" 问题是预存的，与本修改无关）。

### 4. 运行目标测试

```bash
# 无 messages.test.ts 专用测试，跑服务端编译检查
bun run check:server 2>&1 | grep -v "expired quarantine"
```

### 5. 重启服务

修改生效需要重启 server：

```bash
# 找到 server 进程
ps aux | grep "server/index.ts"

# 重启
kill <PID>
nohup bun run src/server/index.ts > /tmp/server.log 2>&1 &
```

### 6. 验证效果

1. 打开发送到 API 的请求日志或 debug 输出，确认 assistant 消息中的 `thinking` 块已被移除（仅保留最后一条）
2. 观察前端上下文格子图，Messages 段的 token 量应显著下降
3. 使用思考模型多轮对话后，auto-compact 触发间隔应显著延长

## 回滚

```bash
git checkout -- src/utils/messages.ts src/services/tokenEstimation.ts
# 或
git revert HEAD
```

## 注意事项

1. **只影响历史 thinking，不影响当前 thinking**：最后一条 assistant 消息的 thinking 块完整保留，模型可以正常延续推理链
2. **`[Thinking removed]` 占位符**：当某条 assistant 消息所有 content 都是 thinking 块时，会被替换为一段纯文本。这是为了避免发送空消息给 API（会 400 错误）
3. **与 `filterTrailingThinkingFromLastAssistant` 的关系**：后者是已有逻辑，移除最后一条消息末尾的 thinking（API 要求不能以 thinking 结尾）。新加的 `filterHistoricalThinkingFromAssistantMessages` 处理**历史**消息，两者互补不冲突
4. **claude-cn 特定**：此修复针对 claude-cn fork 版本的代码路径。原始 Claude Code 代码结构相同，但接入点可能因版本差异需微调
