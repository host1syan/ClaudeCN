# H5 访问

H5 访问是一个可选的个人/团队使用入口。开启后，你可以把桌面端使用的本地服务通过局域网或自己的反向代理暴露出去，然后在手机浏览器里打开同一套聊天 UI。

它不是公开 SaaS 登录系统。任何拿到 H5 Token 的人，都可以访问当前桌面端服务暴露的核心聊天能力，所以只应该在你可控的网络、域名和团队范围内使用。

## 开启方式

1. 在桌面端打开 `设置 -> General -> H5 访问`。
2. 打开 `启用 H5 访问`。
3. 点击生成或重新生成 Token。
4. 复制生成的 Token。之后仍可在本机设置页或本机可信接口中取回；需要撤销旧 Token 时再重新生成。
5. 按你的访问方式填写允许来源：
   - 局域网访问：例如 `http://192.168.1.20:5173`
   - 反向代理访问：例如 `https://cc.example.com`
6. 如果你使用固定域名，可以填写公开访问地址，设置页会据此生成 H5 URL。

H5 设置保存到 `~/.claude/claude-cn/settings.json`。为让已配对设备在重启后继续使用，当前版本会持久化完整 Token；只有本机可信的控制接口可以读回它。请像保护 API Key 一样保护这个文件和 Token。

### 无桌面界面时开启

无界面的 Linux 服务器也可以通过只允许本机调用的控制接口启用 H5。源码运行时先构建 Web UI，再从项目根目录启动服务端；服务端会自动提供 `desktop/dist` 中的页面。需要局域网或反向代理访问时，让它监听外部接口：

```bash
cd desktop
bun run build
cd ..
SERVER_HOST=0.0.0.0 SERVER_PORT=3456 bun run src/server/index.ts
```

如果不是从项目根目录启动服务端，请额外设置 `CLAUDE_H5_DIST_DIR` 为 `desktop/dist` 的绝对路径。没有构建产物时，控制接口仍可生成 Token，但浏览器入口会返回 404。

再从服务器本机的另一个终端生成 Token。响应 JSON 的 `token` 字段就是浏览器需要填写的完整 Token：

```bash
curl -sS -X POST http://127.0.0.1:3456/api/h5-access/enable
```

配置实际前端来源；`allowedOrigins` 必须写浏览器地址栏中前端页面的来源，不能使用通配符。使用反向代理时还可以设置公开访问地址：

```bash
curl -sS -X PUT http://127.0.0.1:3456/api/h5-access \
  -H 'Content-Type: application/json' \
  --data '{
    "allowedOrigins": ["https://cc.example.com"],
    "publicBaseUrl": "https://cc-api.example.com"
  }'
```

稍后需要重新取回完整 Token 或核对配置时，只能在服务器本机调用：

```bash
curl -sS http://127.0.0.1:3456/api/h5-access
```

如果只是通过 SSH 端口转发在自己的电脑上访问，请保持服务监听 `127.0.0.1`，无需开启 H5；完整命令见 [安装指南](./04-installation.md#无界面-linux-服务器)。

## 访问地址

浏览器入口使用前端地址加 `serverUrl` 参数：

```text
https://cc.example.com/?serverUrl=https%3A%2F%2Fcc-api.example.com
```

如果前端和后端由同一个反向代理路径提供，可以把 `serverUrl` 指向同一个公开服务根地址。

首次打开时，H5 页面会要求输入：

| 字段 | 说明 |
|------|------|
| Server URL | 桌面端服务或反向代理后的后端地址 |
| H5 Token | 设置页生成的 Token |

验证成功后，浏览器会把 Server URL 和 Token 保存在当前浏览器的 `localStorage`，之后自动带上 REST `Authorization` 和 WebSocket `token` 参数。

## 推荐部署

### 局域网

适合个人手机测试：

1. 让后端监听局域网地址或由你自己的代理转发到本机端口。
2. 在设置页把手机浏览器打开的前端来源加入允许来源。
3. 手机和电脑连接同一个可信网络。
4. 手机打开 H5 URL，输入 Token。

### 反向代理

适合小团队访问：

1. 使用 HTTPS 域名代理前端静态资源和后端 API。
2. 把前端来源加入 H5 允许来源，不要使用通配来源。
3. 确保 `/api/*`、`/proxy/*` 和 `/ws/*` 都转发到桌面端服务。
4. WebSocket 代理需要支持协议升级。
5. 只把域名分享给可信成员，并单独发送 Token。

反向代理连接本机服务时，后端看到的来源地址通常也是 `127.0.0.1`。为避免把远程请求误认成本地直连，请保留公开 `Host`，或至少传递一个标准代理头：`Forwarded`、`X-Forwarded-For`、`X-Forwarded-Host`、`X-Forwarded-Proto`、`X-Real-IP`、`Via`。

Nginx 可以这样配置：

```nginx
proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

Caddy 的 `reverse_proxy` 默认会保留原始 Host，并补充 `X-Forwarded-*` 头。如果你自定义了 `header_up`，请确保仍保留公开 Host 或上述任一代理头。不要同时把 Host 改成上游的 `127.0.0.1` 并删除全部代理头；缺少这些信息时，服务无法安全区分远程反代请求和本地直连请求。

## 手机体验范围

H5 的第一版优先保证聊天主流程：

- 会话列表通过抽屉打开，默认收起。
- 聊天区是主界面。
- 输入框、发送/停止、附件和权限按钮使用手机可点的尺寸。
- `@` 文件菜单会适配手机宽度。
- Workspace 面板和底部 Terminal 面板在手机宽度下不作为主流程显示。

Electron 桌面端仍保留原来的布局和交互。

## 安全注意

- H5 默认关闭。
- 完整 Token 会持久化，并可从本机可信的设置界面或控制接口读回。
- 远程 API、代理接口和 WebSocket 都需要 H5 Token。
- CORS 只允许设置页配置的来源。
- 禁用 H5 或重新生成 Token 后，旧 Token 会失效。
- 不要把 H5 暴露到公共网络后再把 Token 发到公开渠道。

## 排查

| 现象 | 处理 |
|------|------|
| 页面提示需要 Token | 在设置页复制最新 Token，重新输入 |
| Token 无效 | 重新生成 Token，并更新手机浏览器里保存的 Token |
| 无法连接 Server URL | 检查后端地址、端口、防火墙和反向代理 |
| 浏览器被 CORS 拦截 | 把当前前端来源加入 H5 允许来源 |
| WebSocket 连接失败 | 检查反向代理是否转发 `/ws/*` 并启用 WebSocket upgrade |
