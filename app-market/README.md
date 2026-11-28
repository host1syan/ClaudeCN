# 应用市场（App Market）

容器部署下的应用市场，让 ClaudeCN 容器里能按需安装服务端工具（git、python、php、node、cloudflared 等）。

本目录是**远程目录（remote catalog）**的发布处：服务端启动后通过 jsDelivr 拉取这里的 `catalog.json`，覆盖镜像内置的应用列表。往目录里加一个应用**不需要重新构建镜像**。

## 工作机制

```
镜像内置目录（builtin，兜底）
        +
远程目录 catalog.json（本目录，可随时更新）
        ↓
应用市场 UI（设置 → 应用市场，仅容器部署可见）
        ↓
安装后端：apt / pip / npm / bun / binary 五种 installer，串行队列执行
```

- 安装落在持久卷（`$CLAUDE_CONFIG_DIR/claude-cn/app-market/`），容器重建不丢。
- apt 安装的应用在容器重建后由服务端启动时的 reconcile 自动补装。
- 磁盘剩余不足 1 GiB 时拒绝新安装。

## catalog.json 字段规范

顶层结构：

```json
{
  "version": 1,
  "revision": "2026-11-28-001",
  "apps": [ ... ]
}
```

`revision` 任意字符串，UI 原样展示，用于区分目录版本；改了内容就递增一次（建议 `日期-序号`）。

每个 app 的字段（服务端做**严格白名单清洗**，未列出的字段一律丢弃）：

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | 是 | `[a-z0-9][a-z0-9-]{0,63}`，安装/卸载 API 用它寻址 |
| `name` | 否 | 显示名，缺省用 `id`，最长 120 字符 |
| `summary` | 否 | 一句话说明，最长 300 字符 |
| `installer.type` | 是 | `apt` \| `pip` \| `npm` \| `bun` \| `binary` |
| `installer.packages` | apt | 包名数组，每个须匹配 `[A-Za-z0-9][A-Za-z0-9+._@/-]*` |
| `installer.spec` | pip/npm/bun | 包 spec 字符串，最长 200，只允许 `[A-Za-z0-9@/._:~^-]`（shell 元字符直接拒绝，不做转义） |
| `installer.aptSources` | apt（可选） | 额外 apt 源 id，只能引用内置 curated 源（见下） |
| `binDirs` | binary/npm/bun（可选） | 安装后导出到 PATH 的目录名 |
| `panel` | 否（可选） | `{ "title": "隧道管理" }` —— 仅声明「这个应用有管理面板」。**面板代码永远随镜像发布，远程目录给不出面板实现** |

当前内置 curated apt 源只有一个：`ondrej-php`（ondrej/php PPA，PHP 8.x）。需要新源时服务端代码里的 `CURATED_APT_SOURCES` 要同步加。

### 示例

apt 应用：

```json
{
  "id": "sqlite",
  "name": "SQLite",
  "summary": "嵌入式数据库 CLI",
  "installer": { "type": "apt", "packages": ["sqlite3"] }
}
```

binary 应用（下载静态可执行文件）：

```json
{
  "id": "cloudflared",
  "name": "Cloudflare Tunnel",
  "summary": "cloudflared 客户端：把内网服务通过隧道发布到公网",
  "installer": { "type": "binary" },
  "binDirs": ["bin"],
  "panel": { "title": "隧道管理" }
}
```

注意：binary 类型还要求服务端代码 `binaryDownloadUrls()` 里有该 id 对应的下载地址（amd64 + arm64），远程目录给不出 URL——这是防止远程数据变成任意代码执行入口的一部分。

## 新增一个应用（不改镜像）

1. 在本目录 `catalog.json` 的 `apps` 里加一个条目，遵守上面的字段规范。
2. 递增 `revision`。
3. 提交推送后 jsDelivr 会在几分钟内生效（必要时 purge：`curl https://purge.jsdelivr.net/gh/host1syan/ClaudeCN@main/app-market/catalog.json`）。
4. 应用市场 UI 里点刷新即可看到新应用。

限制：

- `apt` / `pip` 类型可以直接加（安装走系统包管理器，无额外服务端代码）。
- `binary` 类型需要在服务端代码加下载 URL（见上），所以要发版。
- `npm` / `bun` 类型直接加 spec 即可。

## 应用配套面板（per-app panel）

部分应用可以带一个**管理面板**：设置 → 应用市场 → 应用卡片上的「面板」按钮，打开后能看到应用的状态（如 cloudflared 的隧道列表）并执行面板声明的操作（如查看版本）。

架构原则：

- **面板代码随镜像走**（服务端进程内的 TS 模块，位于仓库 `src/server/services/appMarket/panels/<appId>.ts`），永不来自远程目录。远程目录只能声明「这个应用有面板」（`panel: { title }`），不能提供面板实现——远程数据永远是数据，不是代码。
- 面板通过 `Exec` seam 与系统交互：argv 数组、无 shell，注入的执行器让单测可以伪造命令输出。
- 面板声明的 action 与其实现必须在同一份镜像代码里，远程目录无法新增 action。

### 新增一个面板（需要发版）

1. 在 `src/server/services/appMarket/panels/<appId>.ts` 实现 `AppPanel`：

   ```ts
   export const myPanel: AppPanel = {
     title: '我的面板',
     async fetchState(exec) {
       // exec 接收 argv 数组（无 shell），返回 { stdout, stderr }
       // 失败时不要抛 —— 把信息放进 state.notes，UI 会渲染成提示
       const res = await exec(['my-binary', 'status', '--json'])
       // ... 解析 res.stdout，返回 AppPanelState
     },
     async runAction(exec, actionId) {
       // actionId 必须对应 fetchState 返回的 actions 列表
       return { id: actionId, ok: true, message: '...' }
     },
   }
   ```

2. 在 `panels/index.ts` 底部注册：

   ```ts
   import { myPanel } from './myapp.js'
   registerAppPanel('myapp', myPanel)
   ```

3. catalog 条目加 `panel: { title: '我的面板' }`（内置目录与远程目录都可以声明）。
4. UI 是通用的：`AppPanelState`（fields/rows/notes/actions/hint）驱动渲染，新面板不需要改前端代码。
5. 面板代码走和安装器一样的测试基建：假执行器跑 `fetchState`，见 `panels/panels.test.ts`。

### 面板状态结构（UI 契约）

```ts
type AppPanelState = {
  title: string                    // 面板标题（Modal 头）
  fields: { key, label }[]         // 每行展示哪些列，按顺序
  rows: {                          // 行（如隧道列表）
    id, title,
    values: Record<key, string>,   // key ↔ fields.key
    status: 'ok'|'warn'|'error'|'idle',
    statusLabel: string,           // 徽标文案（如「活跃」）
  }[]
  notes: string[]                  // 非行状信息（登录提示、错误文字）
  actions: { id, label, danger? }[] // UI 渲染成按钮
  hint?: string                    // 页脚提示（常用命令等）
  fetchedAt: number
}
```

## 目录文件

- `catalog.json` —— 远程目录本体（服务端经 jsDelivr 拉取，URL：`cdn.jsdelivr.net/gh/host1syan/ClaudeCN@main/app-market/catalog.json`）。

镜像内置目录在服务端仓库 `src/server/services/appMarket/catalog.ts`，两边字段规范一致。
