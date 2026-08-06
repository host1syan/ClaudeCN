# Electron 迁移验证清单

> 用于迁移收口，不替代 `docs/desktop/08-electron-migration-tasks.md`。`package-smoke` 只验证包结构和关键资源，不声明 GUI 启动或交互成功。

交互式执行版见 [`electron-migration-qa-checklist.html`](/desktop/electron-migration-qa-checklist.html)，可在本地浏览器直接打开并用 `localStorage` 保存勾选进度与问题记录。

## 已有自动化证据

- [x] `bun run check:native`：sidecar build、Electron TypeScript、83 个 Electron IPC/service 测试、main/preload/preview-preload bundle、Electron `--dir` 打包和当前平台 package-smoke 均通过。
- [x] `bun run check:desktop`：desktop lint、151 个 Vitest 文件、1199 个测试、renderer production build 均通过。
- [x] `bun test scripts/quality-gate/package-smoke/index.test.ts scripts/pr/release-workflow.test.ts`：package-smoke harness 与 release workflow 静态回归通过。
- [x] `bun test scripts/quality-gate/package-smoke/index.test.ts scripts/pr/release-workflow.test.ts scripts/quality-gate/runner.test.ts`：25 tests passed；覆盖 release lane、显式 GitHub publish 配置和 `latest-mac.yml` 引用真实 artifact 的 package-smoke 回归。
- [x] `bun test scripts/quality-gate/package-smoke/index.test.ts scripts/pr/release-workflow.test.ts scripts/release-update-metadata.test.ts`：26 tests passed；覆盖 Windows canonical output、Linux x64/arm64 `latest-linux*.yml` update metadata、release workflow Gatekeeper/signing preflight 和 post-matrix metadata republish。
- [x] `cd desktop && CSC_IDENTITY_AUTO_DISCOVERY=false bun run electron:package:dir`：macOS unpacked `.app` 可打包。
- [x] `bun run test:package-smoke --platform macos`：macOS `.app` 结构、`app.asar`、unpacked sidecar、unpacked `node-pty` native module 和 `spawn-helper` 通过静态检查。
- [x] macOS `.zip/latest-mac.yml` 发布产物复验：`MAC_TARGETS=zip SKIP_INSTALL=1 SIGN_BUILD=0 desktop/scripts/build-macos-arm64.sh` 通过，canonical 输出包含 `.app`、`.zip`、`.zip.blockmap` 和 `latest-mac.yml`，脚本末尾自动运行 canonical package-smoke 并 PASS。
- [x] macOS directory-only native/package 当前态：`bun run check:native` 通过，覆盖 sidecar 构建/ad-hoc signing、Electron 83 tests、production build、`electron-builder --dir` 和 `package-smoke --package-kind dir --artifacts-dir desktop/build-artifacts/electron`。该项只证明 unpacked bundle 结构和关键资源，不证明 `.dmg`、Gatekeeper 或 signed release launch。
- [x] macOS `.dmg/.zip/latest-mac.yml` 当前态复验：`SKIP_INSTALL=1 SIGN_BUILD=0 desktop/scripts/build-macos-arm64.sh` 已通过默认 `dmg zip` target；canonical `desktop/build-artifacts/macos-arm64` 包含 `.app`、`.dmg`、`.dmg.blockmap`、`.zip`、`.zip.blockmap` 和 `latest-mac.yml`，脚本末尾 release package-smoke PASS。
- [x] macOS canonical packaged launch smoke：使用隔离 `CLAUDE_CONFIG_DIR`、隔离 `--user-data-dir` 和 `CC_CN_ELECTRON_DISABLE_SINGLE_INSTANCE_LOCK=1` 启动 `desktop/build-artifacts/macos-arm64/ClaudeCN.app`，server sidecar 启动于临时端口，Computer Use `get_app_state` 成功读取 `Claude Code Companion` 主窗口和 packaged `file://.../app.asar/dist/index.html` renderer。
- [x] macOS canonical packaged system interaction smoke：同一 canonical `.app` 隔离实例中，Computer Use 确认项目选择器显示系统目录选择结果 `/private/tmp/claude-cn-electron-real-fixed-sRFUeO/project`；打开终端后执行 `printf 'canonical-terminal-ok\n'; pwd`，画面可见 `canonical-terminal-ok` 与实际工作目录输出，且未出现 Safe Storage 钥匙串弹窗。
- [x] 本地 `main` 同步后 targeted checks 通过：`cd desktop && bun run test -- src/components/workspace/WorkspaceFileOpenWith.test.tsx src/lib/previewEvents.test.ts electron/services/updater.test.ts`、`cd desktop && bun run check:electron`、`bun test src/server/middleware/cors.test.ts src/server/__tests__/h5-access-policy.test.ts`。

## Review 已收口

- [x] Packaged app 不再信任 `ELECTRON_RENDERER_URL`；只有 dev 模式允许 local HTTP renderer。
- [x] Preview external page bridge 增加消息大小上限、事件类型 allowlist、字段 schema 和 data URL 校验。
- [x] Packaged macOS `node-pty` runtime cache 会从 bundle manifest 校验，发现 tamper 后重建，cache 权限收紧。
- [x] macOS 通知设置入口改走 native command + allowlisted system settings URL。
- [x] Electron `preview.message()` 不再 silent no-op，会转发到 injected preview bridge。
- [x] Dev/release desktop workflows 在 Electron builder 后运行 `test:package-smoke`，阻止缺失关键 runtime 资源的产物上传/发布。
- [x] Packaged Electron `file://` renderer origin 已纳入 server CORS/H5 token local-origin policy，避免真实打包 app 的 chat WebSocket 在 H5 token mode 下被拒绝。
- [x] Electron Builder publish 配置不再依赖 git remote autodetection；`artifactName` 改为无空格稳定文件名，package-smoke 会检查 `latest*.yml` 引用的本地 artifact 是否存在。

## 必跑本地门禁

- [x] `bun run check:native`
- [x] `bun run check:desktop`
- [x] `bun run check:docs`
- [x] `bun run verify`：最新为 `artifacts/quality-runs/2026-05-31T21-42-57-279Z/report.md`，`passed=9 failed=0 skipped=1`。
- [x] `bun run check:coverage`：最新为 `artifacts/coverage/2026-05-31T21-46-15-873Z/coverage-report.md`，`passed=5 failed=0`。
- [x] 同步本地 `main`、`file://` CORS/H5 修复和 Electron 迁移当前态后复跑 `bun run check:server`：88 files / 969 tests passed。
- [x] 同步本地 `main` 与 updater no-op 修复后复跑 `cd desktop && bun run check:electron`：16 files / 76 tests passed。
- [x] `bun run test:package-smoke --platform macos`
- [x] `cd desktop && bun run test -- --run scripts/dev-launcher.test.ts && bun run check:electron`：新增 Electron dev launcher 代理绕过回归，最新 Electron checks 为 17 files / 79 tests passed。
- [x] `bun run test:package-smoke --platform macos`：结构检查仍通过，并明确提示该命令不做 Gatekeeper launch approval。
- [x] `cd desktop && bun run test -- src/lib/desktopRuntime.test.ts src/components/shared/UpdateChecker.test.tsx src/lib/composerAttachments.test.ts src/components/chat/ChatInput.test.tsx src/pages/EmptySession.test.tsx src/components/layout/AppShell.test.tsx src/components/controls/PermissionModeSelector.test.tsx src/components/shared/RepositoryLaunchControls.test.tsx`：8 files / 75 tests passed；覆盖 Electron desktop runtime 判断、更新提示、native attachment picker 和移动布局分支。
- [x] `cd desktop && bun run check:electron`：16 files / 77 tests passed；覆盖 Electron updater proxy contract、IPC payload validation、main/preload/preview-preload bundle。
- [x] `bun test scripts/quality-gate/package-smoke/index.test.ts`：7 tests passed；发布型 macOS 包缺失 resources/app-update.yml 会失败，纯 `--dir` 开发包不强制该文件。
- [x] `bun test scripts/quality-gate/package-smoke/index.test.ts scripts/pr/quality-contract.test.ts`：12 tests passed；覆盖 host platform 到 package-smoke platform 的映射，并锁定 `check:native` 必须包含 `electron:package:dir` 与 `test:package-smoke:current`。
- [x] `bun run check:native`：最新复跑完成 sidecar build、Electron 77 tests、Electron `--dir` package，并执行 `bun run test:package-smoke:current` -> macOS `.app` 结构检查 PASS；notes 明确当前纯 `--dir` artifact set 不强制 `app-update.yml`，也不做 GUI/Gatekeeper launch approval。
- [x] Linux `electron-builder --dir` package-smoke 回归：Linux 纯 `linux-unpacked` 开发目录包会检查 `app.asar`、unpacked sidecar 和 `node-pty` 后通过，不要求 AppImage/deb；发布型 AppImage/deb artifact set 仍要求 update metadata 和 `app-update.yml`。
- [x] PR native workflow Electron 化：`desktop-native-checks` 不再安装 WebKitGTK/Rust/Rust cache，`build-sidecars` 不再用 `rustc -vV` 推导 host triple；`check:native` 只依赖 Bun/Electron sidecar/package-smoke 链路。
- [x] package-smoke 显式包类型：`check:native` 通过 `--package-kind dir --artifacts-dir desktop/build-artifacts/electron` 验开发目录包；dev/release workflows 和平台构建脚本通过 `--package-kind release --artifacts-dir <精确输出目录>` 验发布包，避免旧 artifact 混入。
- [x] release gate 验包收窄：`quality:gate --mode release` 现在包含 PR checks，并使用当前平台 canonical release dir 运行 `test:package-smoke --package-kind release --artifacts-dir desktop/build-artifacts/<platform-arch>`，不再把 `desktop/build-artifacts/electron` 的开发目录包当 release 证据；macOS release lane 还会追加 `--require-macos-gatekeeper`，与 release workflow 上传前 Gatekeeper 检查保持一致。
- [x] 当前平台 canonical release package-smoke 复验：`bun run test:package-smoke --platform macos --package-kind release --artifacts-dir desktop/build-artifacts/macos-arm64` 通过，确认 canonical zip/update metadata/app-update.yml/sidecar/node-pty 结构仍完整；该命令仍不声明 GUI/Gatekeeper launch success。
- [x] release workflow preflight：GitHub tag/workflow_dispatch 打包矩阵前先运行 `bun run verify`，避免 tag release 绕过 PR-quality 基线。
- [x] signing/notarization secrets preflight：独立非 matrix `signing-preflight` job 在打包矩阵前检查 Developer ID、notarization 和 Windows signing secrets；缺失时 GitHub Actions 直接报错并阻止所有平台 artifact 上传，不再等到 Electron Builder/Gatekeeper 阶段或发布 partial release 后才失败。
- [x] 多架构 update metadata 上传冲突规避：release workflow 上传前将 `latest*.yml` 重命名为带 matrix label 的唯一 asset，避免 GitHub Release 中 `latest-mac.yml/latest-linux.yml` 同名覆盖或上传失败。
- [x] 多架构标准 updater metadata 合并：`scripts/release-update-metadata.ts` 会在 matrix build 全部通过后重新发布标准 channel metadata；macOS 合并 x64/arm64 entries 到 `latest-mac.yml`，Linux 恢复 `latest-linux.yml` / `latest-linux-arm64.yml`，Windows 恢复 `latest.yml`。完整自动更新 release 仍需在 signed/notarized artifact 上复验。
- [x] `MAC_TARGETS=zip SKIP_INSTALL=1 SIGN_BUILD=0 desktop/scripts/build-macos-arm64.sh`：canonical macOS zip/update artifact set 通过；脚本自动执行 `bun run test:package-smoke --platform macos --package-kind release --artifacts-dir desktop/build-artifacts/macos-arm64`，确认 `latest-mac.yml` 引用真实 zip，安装后 resources 包含 `app-update.yml`。
- [x] `bun run quality:gate --mode baseline --allow-live --only 'provider-smoke:*' --provider-model sub2api-chatgpt:main:sub2api-chatgpt-main`：`artifacts/quality-runs/2026-05-31T21-28-07-154Z/report.md`，`passed=1 failed=0 skipped=0`，直接复用本机 claude-cn provider selector。
- [x] 当前 release provider smoke 复验：`bun run quality:gate --mode release --allow-live --only 'provider-smoke:*' --provider-model sub2api-chatgpt:main:sub2api-chatgpt-main` 通过，报告为 `artifacts/quality-runs/2026-06-01T12-18-47-144Z/report.md`，`passed=1 failed=0 skipped=0`。
- [x] `cd desktop && bun test electron/services/updater.test.ts`：10 tests passed；覆盖 unpacked `app-update.yml` 缺失、packaged update config 缺失时不调用 updater、GitHub release 缺少 `latest-mac.yml` 时降级为无更新、关闭 electron-updater 内置 logger，以及非 metadata updater 错误继续抛出。
- [x] `cd desktop && bun run check:electron`：最新 97 tests passed；覆盖通知 smoke `close/failed` lifecycle JSONL、synthetic action + renderer ack JSONL、updater missing channel metadata 修复、Keychain guard、single-instance validation bypass、update smoke stub、window smoke 诊断和 Electron main/preload/preview-preload bundle。
- [ ] `bun run test:package-smoke --platform macos --package-kind release --artifacts-dir desktop/build-artifacts/macos-arm64 --require-macos-gatekeeper`：当前失败；非 Gatekeeper 结构检查 PASS，包含 `latest-mac.yml` 引用 zip、安装后 `app-update.yml`、sidecar 和 `node-pty`。追加 Gatekeeper 后 `codesign` verify/details 通过，但 `spctl` 返回 `bundle format unrecognized, invalid, or unsuitable`，`stapler` 返回 `does not have a ticket stapled to it`。package-smoke 现已在 `spctl` 报 `Too many open files` 时用 raised file descriptor limit 自动重试，避免该临时诊断掩盖真实 Gatekeeper 结果。release-ready 需要 Developer ID signing/notarization 或在真实签名产物上复验。
- [x] Gatekeeper 失败诊断：`--require-macos-gatekeeper` 现在在 `spctl` 失败时记录 `codesign --verify --deep --strict --verbose=2`、`codesign -dv --verbose=4` 和 `xcrun stapler validate` 摘要，便于 signed/notarized artifact 在 CI/release runner 上定位签名链、bundle 格式或 notarization ticket 问题。
- [x] Gatekeeper 诊断复验：`bun run test:package-smoke --platform macos --package-kind release --artifacts-dir desktop/build-artifacts/macos-arm64 --require-macos-gatekeeper` 预期失败；输出中包含 `spctl Gatekeeper assessment exited with status 1`、`codesign verification exited with status 0`、bundle identifier/signature detail 摘要，以及 `notarization ticket validation exited with status 65` / `does not have a ticket stapled to it`。
- [x] release workflow macOS 上传前强制运行 `bun run test:package-smoke --platform macos --package-kind release --artifacts-dir desktop/build-artifacts/electron --require-macos-gatekeeper`；开发构建仍只跑结构验包，因为 dev workflow 明确禁用签名自动发现。
- [x] Security/Runtime Review 修复：Electron renderer 不再有 `previewEval` 任意 JS 注入 IPC；预览截图/元素选择走结构化 `preview.message`；`shell.openPath` 拒绝 app bundle、脚本/安装器/Windows 可执行扩展和 POSIX executable 文件；Electron `windowControls=false`，Windows 暂不显示自定义 window chrome。
- [x] 历史 `bun run quality:gate --mode release --allow-live --provider-model sub2api-chatgpt:main:sub2api-chatgpt-main`：`artifacts/quality-runs/2026-05-31T20-29-45-758Z/report.md`，`passed=15 failed=0 skipped=0`。SubAgent Review 后 release gate 已收窄为 canonical release artifact 验包；该历史报告不能再作为当前 release-ready 证据。
- [ ] 重新运行当前 `bun run quality:gate --mode release --allow-live --provider-model sub2api-chatgpt:main:sub2api-chatgpt-main`：需要 signed/notarized artifact、canonical release metadata 和可用 Computer Use 桌面会话。
- [x] `bun run quality:gate --mode release --dry-run --only 'desktop-package-smoke:*'`：报告 `artifacts/quality-runs/2026-06-01T12-15-48-692Z/report.md`，确认 macOS release package-smoke 命令包含 `--require-macos-gatekeeper`。
- [x] `bun run quality:gate --mode release --only 'desktop-package-smoke:*'`：报告 `artifacts/quality-runs/2026-06-01T12-16-06-442Z/report.md`，按预期失败于 Gatekeeper；结构检查均通过，`spctl` 返回 `bundle format unrecognized, invalid, or unsuitable`，`stapler` 返回 `does not have a ticket stapled to it`。
- [x] `bun run check:policy`：77 tests passed，并执行 `check:quarantine`；覆盖 release workflow、quality gate runner、provider/desktop smoke、change policy、quality contract 和 quarantine governance。
- [x] 本地 `main` 同步后 focused regression：已应用 `main` 最近 10 个提交的非重叠改动，并手动合并 `ChatInput.tsx` / `previewEvents.ts` / `previewEvents.test.ts` 冲突面。验证：desktop ChatInput/previewEvents 24 tests passed；server/CLI/image/powershell/provider 124 tests passed；Feishu streaming card 36 tests passed；`cd desktop && bun run check:electron` 94 tests passed。
- [x] 本地 `main` 同步后重新打包：`CSC_IDENTITY_AUTO_DISCOVERY=false bun run electron:package:dir` 完成当前源码 packaged dir；`bun run test:package-smoke --platform macos --package-kind dir --artifacts-dir desktop/build-artifacts/electron` PASS。
- [x] release update blockmap 验证补强：`bun test scripts/quality-gate/package-smoke/index.test.ts` 通过 17 tests；`bun run test:package-smoke --platform macos --package-kind release --artifacts-dir desktop/build-artifacts/macos-arm64` PASS，并确认当前 macOS release artifact set 包含 `.dmg.blockmap` 和 `.zip.blockmap`。Windows `.exe.blockmap` 与 Linux `.AppImage.blockmap` 已由 fixture 回归覆盖，仍需实机 release artifact smoke。
- [x] 通知点击 renderer ack 证据补强：Electron renderer 处理通知 action 并打开目标 session 后，会通过 `desktop:notification:action-ack` 回写 `renderer_ack` 到 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_LOG`，用于后续可点击 OS 通知环境证明 main action 已进入 UI 导航链路。验证：focused notification/desktopHost tests 42 passed；`cd desktop && bun run check:electron` 97 tests passed。
- [x] `git diff --check`

## macOS Computer Use Smoke

- 2026-06-01 复测阻塞：早期 Computer Use 工具层对 `list_apps` 和 `get_app_state(Finder)` 返回 `NSOSStatusErrorDomain Code=-600 procNotFound`；后续 `list_apps` 恢复，但 `get_app_state(Electron)` 仍超时，`get_app_state(Finder)` 为 `cgWindowNotFound`，同时系统 `screencapture` 返回 `could not create image from display`。当前阻塞已定位为本机会话显示/截图能力不可用，不能把它解释为 Electron app 单点故障。
- 2026-06-01 continuation 复测：Computer Use `list_apps` 可列出运行中应用，但 `get_app_state(Finder)` 仍为 `cgWindowNotFound`；系统 `screencapture` 仍返回 `could not create image from display`。当前仍无法进行点击、输入、选择文件等 Computer Use GUI 操作。
- 2026-06-01 Electron dev smoke：本机代理环境会让 `electron:dev` 的 renderer 等待请求命中 502；已把默认 renderer URL 改为 `http://localhost:1420`，并对当前进程、Vite 子进程和 Electron 子进程补齐 `NO_PROXY/no_proxy=localhost,127.0.0.1,::1`。重启后 Electron dev 壳可启动 sidecar 和 renderer。
- 2026-06-01 窗口可见性修正：Electron main 在创建主窗口后立即 `show/focus`，renderer load 完成后再补一次，避免隐藏窗口等待 renderer load 时被 Computer Use/系统自动化看成无窗口。
- 2026-06-01 packaged launch policy：`open -n desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app` 被 macOS `AppleSystemPolicy` 直接终止；ad-hoc 和本机 Apple Development 重新签名的临时副本均未通过 `spctl`。因此当前 unpacked `.app` 只能证明结构完整，不能声明 release launchable。
- 2026-06-01 DMG builder policy：清理所有旧 `ClaudeCN` disk image 挂载后，zip-only Electron Builder 可成功产出 update metadata；DMG target 仍在本机 `hdiutil create` 超时，且失败后的临时读写 disk image 无法正常 detach。该项保持 macOS runner/DiskImages blocker。
- 2026-06-01 final-definition 复测：Computer Use `list_apps` 仍可列出运行中应用，`get_app_state(Finder)` 仍返回 `cgWindowNotFound`；系统 `screencapture -x` 仍返回 `could not create image from display`。当前机器仍不能作为 Computer Use GUI 验收环境。
- 2026-06-01 cross-platform-script 复测：Computer Use `list_apps` 可列出应用且包含 `/Applications/ClaudeCN.app`，但 `get_app_state(Finder)` 仍返回 `cgWindowNotFound`，`get_app_state(ClaudeCN)` 返回 `remoteConnection`，系统 `screencapture -x /tmp/claude-cn-computer-use-check.png` 仍失败为 `could not create image from display`。当前机器继续不能完成真实 GUI 操作验收。
- 2026-06-01 package-kind 复测：Computer Use `list_apps` 可列出运行中的 `/Applications/ClaudeCN.app`，但 `get_app_state(ClaudeCN)` 和 `get_app_state(Finder)` 均返回 `cgWindowNotFound`；系统 `screencapture -x /tmp/claude-cn-computer-use-check-2.png` 仍失败为 `could not create image from display`。当前机器继续不能完成点击、输入、系统文件选择器、托盘/通知等 GUI 验收。
- 2026-06-01 Gatekeeper 诊断后复测：Computer Use `list_apps` 可列出运行中的 `/Applications/ClaudeCN.app`，但 `get_app_state(ClaudeCN)` 和 `get_app_state(Finder)` 均返回 `cgWindowNotFound`；系统 `screencapture -x /tmp/claude-cn-computer-use-check-3.png` 仍失败为 `could not create image from display`。当前机器仍不是可用的 Computer Use GUI 验收环境。
- 2026-06-01 release metadata 合并后复测：Computer Use `list_apps` 仍可列出运行中的 `/Applications/ClaudeCN.app`，但 `get_app_state(ClaudeCN)` 和 `get_app_state(Finder)` 仍返回 `cgWindowNotFound`；系统 `screencapture -x /tmp/claude-cn-computer-use-check-4.png` 仍失败为 `could not create image from display`。当前机器仍不能执行真实 GUI 操作验收。
- 2026-06-01 mock update feed UI 补强后复测：Computer Use `list_apps` 仍可列出运行中的 `/Applications/ClaudeCN.app`，但 `get_app_state(ClaudeCN)` 和 `get_app_state(Finder)` 仍返回 `cgWindowNotFound`；系统 `screencapture -x /tmp/claude-cn-computer-use-check-5.png` 仍失败为 `could not create image from display`。当前机器仍不能执行真实 GUI 操作验收。
- 2026-06-01 Keychain prompt mitigation：Electron main 在 macOS 启动早期启用 Chromium `use-mock-keychain`，避免 `claude-code-desktop Safe Storage` 反复弹登录钥匙串授权；`ModelSelector` 不再挂载即请求 Claude/OpenAI 官方 OAuth status，而是 runtime 下拉打开时按需请求一次。验证：focused ModelSelector/keychain tests 12 passed，`check:electron` 80 passed。
- 2026-06-01 Keychain prompt Computer Use 复测：当前 Electron dev 壳启动后 `get_app_state(Electron)` 成功返回 `localhost:1420/` 主界面；Computer Use 截图和 AX 树没有出现 Safe Storage 钥匙串授权弹窗。
- 2026-06-01 Keychain prompt 当前态复验：focused ModelSelector/keychain tests 12 passed；`check:electron` 83 passed；Computer Use `get_app_state(Electron)` 返回 `Claude Code Companion` 主窗口和 `localhost:1420/` renderer，未出现 Safe Storage 钥匙串授权弹窗。
- 2026-06-01 附件选择 Computer Use 复测：当前 Electron dev 壳中点击 composer `添加文件或图片`，macOS 原生 `打开` 面板可用；选择 `/Users/nanmi/claude-cn-cua-attachment-smoke.txt` 后回到 composer，并显示 `claude-cn-cua-attachment-smoke.txt` 附件 chip，未出现 Safe Storage 钥匙串弹窗。
- 2026-06-01 Python 路径选择 Computer Use 复测：当前 Electron dev 壳中点击 Computer Use 设置里的 `选择`，macOS 原生 `选择 Python 解释器` 面板可用；选择 `/Users/nanmi/claude-cn-cua-python3-smoke` 后设置页保存并解析到 `/opt/homebrew/Cellar/python@3.14/3.14.5/Frameworks/Python.framework/Versions/3.14/bin/python3.14`；随后恢复自动检测，`/api/computer-use/authorized-apps` 返回 `pythonPath: null`。
- 2026-06-01 外链 Computer Use 复测：当前 Electron dev 壳中点击关于页 GitHub 项目卡片后，系统浏览器 Google Chrome 被拉到前台，地址栏为 `github.com/host1syan/claude-cn`；Electron renderer 仍停留在 `localhost:1420/`，未在应用内导航外链。
- 2026-06-01 窗口关闭/恢复 Computer Use 复测：当前 Electron dev 壳中点击 macOS 关闭按钮后，Electron 进程仍在 Computer Use `list_apps` 中保持 running 且主窗口消失；通过 macOS app activation 路径重新激活同一 Electron app 后，Computer Use 再次看到 `Claude Code Companion` 主窗口和 `localhost:1420/` renderer。Computer Use 当前不能稳定枚举 Dock，因此本轮验证的是与 Dock 点击等价的 app activation/show 主窗口路径。
- 2026-06-01 通知点击 smoke hook：新增显式环境变量 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_SESSION_ID=<session-id>`，用于让 Electron main process 发送带 session target 的真实 Electron 通知；focused tests 覆盖默认不发送、target payload 透传和窗口恢复链路。真实 OS 点击复测时，Computer Use 对 Notification Center/SystemUIServer 均超时，且 `get_app_state(Electron)` 本身会激活 app，因此不能作为点击通知证据；该项仍保持未勾选。
- 2026-06-01 通知点击二次复测：用 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_DELAY_MS=30000` 启动 Electron dev 壳，Computer Use 先确认 `localhost:1420/` 主窗口可见，再点击 macOS close button；Electron 随后无主窗口，符合隐藏到后台预期。等待通知触发后，Computer Use 对 `SystemUIServer` 和 `/System/Library/CoreServices/NotificationCenter.app` 仍返回 `timeoutReached`，前台 Chrome 截图/AX 树也未出现可点击通知横幅，因此仍不能证明真实 OS 通知点击回跳。
- 2026-06-01 packaged 通知 JSONL smoke：新增 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_LOG=<path>` 后重新打包 packaged app；隔离配置启动后 Computer Use 确认 packaged `file://.../app.asar/dist/index.html` 主窗口可见，JSONL 记录 `scheduled` 与 `sent:true`。Computer Use 对 `SystemUIServer` 和 Notification Center 仍 `timeoutReached`，JSONL 未出现 `action`，因此仍不能勾选“点击通知后回到目标 session”。
- 2026-06-01 packaged 通知点击当前态复测：canonical packaged app 使用 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_SESSION_ID=notification-click-smoke-session`、`CC_CN_ELECTRON_NOTIFICATION_SMOKE_DELAY_MS=8000` 和 JSONL log 启动；Computer Use 关闭主窗口后 app 留在后台，JSONL 记录 `scheduled` 与 `sent:true`。Computer Use 对 `UserNotificationCenter.app` 返回安全策略拒绝，对 `NotificationCenter.app` 仍 `timeoutReached`，因此真实 OS 点击仍未完成。通知 smoke 现已记录 `lifecycle: close|failed`，后续可在可操作通知横幅的 macOS/Windows runner 上区分 close、failed 和 action。
- 2026-06-01 packaged 通知点击 launchctl 复测：直接执行 app binary 时 smoke env 没有进入最终 Electron app 进程；改用 `launchctl setenv` + `open -n` 启动隔离 packaged app 后，JSONL 正常记录 `scheduled` 和 `sent:true`。Computer Use 对 Notification Center 与 SystemUIServer 仍 `timeoutReached`，Finder 状态读取返回 ScreenCaptureKit stream error，JSONL 未出现 `action`，所以真实点击回跳仍不能勾选。
- 2026-06-01 通知点击再次复测：worktree packaged app 通过 `launchctl setenv` 启动后，通知 JSONL 正常记录 `scheduled` 和 `sent:true`；Computer Use 对 worktree app 连续返回 ScreenCaptureKit stream error，`list_apps` 仍能看到该 app running。复制当前 packaged app 为唯一 bundle id smoke app 并 ad-hoc 重签名后，LaunchServices 能注册 `ClaudeCN Smoke`，但 app 立即退出，系统日志显示 appDeath；本轮仍没有真实 OS 点击 `action` 证据。
- 2026-06-01 renderer ack 补强后 packaged 复测：重新打包 `desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app` 并通过 package-smoke dir PASS；首次以 notification smoke env 启动时 Computer Use 读取 worktree packaged app 返回 `timeoutReached`，直接执行也曾退出 137。随后用隔离 `CLAUDE_CONFIG_DIR` / `--user-data-dir` 重新直接运行同一 `.app` 后保持运行，Computer Use 成功读取 `Claude Code Companion` 主窗口，renderer URL 为 `file:///Users/nanmi/.codex/worktrees/2392/claude-code-cn/desktop/build-artifacts/electron/mac-arm64/Claude%20Code%20Cn.app/Contents/Resources/app.asar/dist/index.html`；点击设置入口后进入 packaged 设置页。`spctl` 仍返回 `bundle format unrecognized, invalid, or unsuitable`，`codesign --verify --deep --strict` 通过，因此 signed/notarized Gatekeeper 与真实 OS 通知点击仍未完成。
- 2026-06-01 packaged synthetic notification action 复测：新增 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_TRIGGER_ACTION=1`，仅在显式 smoke 环境中由 main process 触发同一条 notification action -> renderer navigation -> `desktop:notification:action-ack` 诊断链路。用 `launchctl setenv` + `open -n desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app --args --user-data-dir=<tmp>` 启动当前 worktree packaged app 后，Computer Use 成功读取 packaged `file://.../app.asar/dist/index.html` 主窗口；JSONL 记录 `scheduled`、`sent:true`、`synthetic_action` 与 `renderer_ack`。该 synthetic action 不代表真实 OS 通知点击，真实点击回跳仍未勾选。
- 2026-06-01 DiskImages 复核：`hdiutil info` 仍显示 Electron Builder 失败遗留的 `.temp...Claude-Code-Cn-0.3.2-arm64.dmg` 挂载在 `/dev/disk4`，但无 `hdiutil` / `diskutil` / `electron-builder` 残留进程；此前 `hdiutil detach` 和 `diskutil` 对该设备会阻塞，继续强制处理会污染当前验证。
- 2026-06-01 macOS build script hardening：默认 `SKIP_INSTALL=1 SIGN_BUILD=0 desktop/scripts/build-macos-arm64.sh` 现在在 stale `.temp...Claude-Code-Cn-0.3.2-arm64.dmg` 挂载存在时 fail fast，并提示改用 `MAC_TARGETS=zip` 验证 update zip；`MAC_TARGETS=zip` 路径通过且自动 package-smoke PASS。
- 2026-06-01 macOS canonical release 复测：默认 `SKIP_INSTALL=1 SIGN_BUILD=0 desktop/scripts/build-macos-arm64.sh` 已完成 `.dmg + .zip + latest-mac.yml`，package-smoke release PASS。启动 canonical `.app` 后 Computer Use 成功读取 `Claude Code Companion` 主窗口；electron-updater 内置 logger 已关闭，缺失 GitHub `latest-mac.yml` 不再向启动日志输出 404 stack。
- 2026-06-01 canonical packaged system interaction 复测：启动 `desktop/build-artifacts/macos-arm64/ClaudeCN.app` 的隔离实例后，Computer Use 读取到 packaged `file://.../app.asar/dist/index.html` 主界面，项目选择器显示 `/private/tmp/claude-cn-electron-real-fixed-sRFUeO/project`；点击 `打开终端` 后终端面板运行 `/bin/zsh`，输入 `printf 'canonical-terminal-ok\n'; pwd` 后显示 `canonical-terminal-ok` 与 `/tmp/claude-cn-canonical-cua-flow.xTxSmt/claude-config`。该轮未出现 `claude-code-desktop Safe Storage` 钥匙串授权弹窗。
- 2026-06-01 canonical packaged Computer Use 设置流程复测：启动 `desktop/build-artifacts/macos-arm64/ClaudeCN.app` 的隔离实例后，用 Computer Use 进入 `设置 > Computer Use`。首次状态请求曾 30s timeout，点击 `重试` 后状态页正常；通过 `安装环境` 按钮完成 venv 创建与依赖安装。UI 显示 Python 3.14.5、虚拟环境已就绪、依赖包已安装、辅助功能权限已授权、屏幕录制权限未授权；`GET /api/computer-use/status` 返回 `venv.created=true`、`dependencies.installed=true`、`permissions.accessibility=true`、`permissions.screenRecording=false`。点击 `打开屏幕录制设置` 后，Computer Use 读取到 macOS 系统设置窗口 `录屏与系统录音`，列表中包含 `ClaudeCN` 权限项。
- 2026-06-01 canonical packaged Computer Use 授权应用复测：同一 packaged app 隔离实例中，Computer Use 在 `已授权应用` 搜索框输入 `Terminal`，列表返回 `Terminal / com.apple.Terminal`；点击后 UI 显示 check 状态，隔离 `claude-cn/computer-use-config.json` 与 `GET /api/computer-use/authorized-apps` 均返回 `authorizedApps[0].bundleId=com.apple.Terminal`，grant flags 为 `clipboardRead=true`、`clipboardWrite=true`、`systemKeyCombos=true`。该项覆盖真实 UI 输入、点击、滚动、应用枚举和授权配置持久化。
- 2026-06-01 packaged menu/window lifecycle 复测：在隔离 `desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app` 中，Computer Use 点击 macOS app menu `Settings...` 后打开设置页；点击窗口关闭按钮后主窗口消失且同一 PID 继续运行，System Events 显示 `count=0`；随后 app activation 恢复窗口，Computer Use 再次读取到 packaged `file://.../app.asar/dist/index.html` 设置页。继续通过 app menu `Quit` 退出后，隔离配置写入 `window-state.json`；用同一 `CLAUDE_CONFIG_DIR` 和 `--user-data-dir` 重启，Computer Use 成功读取新 PID 的 packaged 主窗口，System Events 显示 `count=1 names=Claude Code Companion`。该项覆盖菜单打开设置、关闭隐藏、恢复窗口和重启后窗口仍可见；真实 tray 图标点击仍未单独完成。
- 2026-06-01 Gatekeeper 诊断补强后 packaged launch 复测：用隔离 `CLAUDE_CONFIG_DIR` 和 `--user-data-dir` 启动 `desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app`，Computer Use 成功读取 `Claude Code Companion` 主窗口和 packaged `file://.../app.asar/dist/index.html` renderer，未出现 Safe Storage 钥匙串弹窗。
- 2026-06-01 release provider smoke 后 packaged launch 复测：真实 provider smoke 通过后，再次用隔离 `CLAUDE_CONFIG_DIR` 和 `--user-data-dir` 启动 `desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app`，Computer Use 成功读取 packaged `file://.../app.asar/dist/index.html` 主窗口。
- 2026-06-01 policy gate 后 packaged launch 复测：`check:policy` 通过后，再次用隔离 `CLAUDE_CONFIG_DIR` 和 `--user-data-dir` 启动 packaged app；Computer Use 首次读取遇到 ScreenCaptureKit stream error，System Events 同时显示同一 PID 有 `Claude Code Companion` 窗口，重试后 Computer Use 成功读取 packaged renderer。
- 2026-06-01 本地 `main` 同步后 packaged launch 复测：重新打包 `desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app`，使用隔离 `CLAUDE_CONFIG_DIR=/tmp/claude-cn-main-sync-cua-1780317323/config` 和隔离 `--user-data-dir` 启动；Computer Use 成功读取 `Claude Code Companion` 主窗口和当前 packaged renderer，未出现 Safe Storage 钥匙串弹窗。
- [x] 启动真实 packaged app：`desktop/build-artifacts/electron-smoke/mac-arm64/ClaudeCN Smoke.app`。该 smoke bundle 使用同一构建产物重新打包，仅更换临时 appId/productName 以避开本机已运行的正式 app 单实例锁。
- [x] 本地 `main` 同步后启动真实 packaged app：`desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app`，使用隔离 `--user-data-dir=/tmp/claude-cn-electron-real-fixed-sRFUeO/user-data` 和隔离 `CLAUDE_CONFIG_DIR`。
- [x] 确认主界面从 packaged renderer 加载，local server sidecar 自动启动，无需手动 `SERVER_PORT`。
- [x] 打开终端，执行 `printf electron-terminal-ok`，确认输出可见。
- [x] 关闭终端后通过进程列表确认没有残留 `/tmp/claude-cn-electron-smoke-config` PTY shell。
- [x] 选择工作目录，确认系统目录对话框可打开。
- [x] 复制本机 claude-cn provider/model 配置到隔离临时配置目录，真实模型选择器显示 `gpt-5.5 Sub2API-ChatGPT`。
- [x] 真实模型 chat smoke：会话 `e0d7abc2-27db-4ff1-9954-7923f6c3385e` 通过 WebSocket 启动 CLI subprocess，读取 `package.json` 与 `src/greeting.ts`，经 UI 权限审批执行 `bun test`，JSONL 记录 `1 pass / 0 fail`，Computer Use 看到最终回复“测试通过。”。
- [x] 选择附件文件，确认文件可加入 composer。
- [x] 打开 Python 路径选择，确认系统文件选择器可用。
- [x] 点击外链或 OAuth 帮助链接，确认走系统浏览器。
- [x] 关闭主窗口，确认 Dock/app activation 可恢复窗口；真实 tray 图标点击仍需可枚举 tray 环境复验。
- [ ] 触发通知，点击通知后回到目标 session。
- [x] 触发通知发送 smoke：packaged app 通过 `CC_CN_ELECTRON_NOTIFICATION_SMOKE_LOG` 记录 `scheduled` 和 `sent:true`；真实 OS 点击仍因 Computer Use 无法操作 Notification Center/SystemUIServer 而未完成。
- [x] 打开 preview/workbench browser panel，导航 `https://example.com/`，确认 WebContentsView 加载成功。
- [x] 点击 preview 截图，确认 `screenshot-full.png` 回填到 composer。
- [x] packaged Computer Use 设置流程：隔离配置下完成环境安装，确认 venv/依赖状态、辅助功能/屏幕录制权限状态，并从应用打开 macOS `录屏与系统录音` 设置页。
- [x] packaged Computer Use 授权应用流程：搜索并授权 `Terminal / com.apple.Terminal`，确认剪贴板和系统快捷键 grant flags 持久化。
- [x] unpacked dir package 缺少 `app-update.yml` 时 update check 视为无更新，不阻塞 UI；非 metadata updater 错误仍会抛出。
- [x] 更新代理 contract：Electron updater 会接收 renderer 传入的 manual proxy，并在切回 system proxy 时清理手动代理；IPC validator 只允许 `{ proxy: string }` 形状，拒绝空 proxy 或额外字段。
- [x] 关闭 electron-updater 内置 logger，避免缺失远端 `latest-mac.yml` 时把 404 stack 打到 packaged app 启动日志；业务层仍把 metadata 缺失分类为“无更新”。
- [x] 发布型包 updater metadata：如果 artifact set 中存在 release archive/update metadata，`package-smoke` 会强制检查安装后 resources/app-update.yml，防止“有 latest*.yml 但安装后永远无法检查更新”的包进入 release；macOS zip-only canonical artifact set 已通过该检查。
- [x] packaged app updater 缺失 channel metadata smoke：重新打包 `desktop/build-artifacts/electron/mac-arm64/ClaudeCN.app` 后，用隔离配置目录直接运行 app executable；Computer Use 确认 packaged renderer 从 `file://.../app.asar/dist/index.html` 加载，local server sidecar 自动启动，未出现 Safe Storage 钥匙串弹窗，日志未再出现 `ERR_UPDATER_CHANNEL_FILE_NOT_FOUND` IPC handler 崩错。纯 `--dir` 包缺少 `app-update.yml` 的提示仍按无更新处理。
- [x] 触发 mock feed，确认 check/download/install/relaunch 状态流不阻塞 UI：`UpdateChecker` 集成测试通过 Electron `desktopHost` mock update feed 驱动真实 `updateStore`，覆盖下载完成弹窗、点击 install、prepare/install/relaunch 调用和 prompt 退出。signed/release update feed 仍需在真实 signed/notarized artifact 上复验。
- [x] packaged main/preload update smoke：`CC_CN_ELECTRON_UPDATE_SMOKE_VERSION=9.9.9-smoke` 启动唯一 bundle id 的 packaged dir app，packaged renderer 通过 preload 调用 `desktopHost.updates.check/download/prepareInstall/install/relaunch`，JSONL 记录 `check`、`download-start`、`download-finish`、`quit-and-install`。Computer Use `list_apps` 可见该 smoke app，但 `get_app_state` 超时，真实桌面点击安装仍按未完成处理。
- [x] packaged window smoke 诊断：`CC_CN_ELECTRON_WINDOW_SMOKE_LOG` 记录 packaged app 在 `after-initial-show` 和 `after-final-show` 均为 `visible:true`，bounds 为 `1280x820`，renderer URL 为 packaged `file://.../app.asar/dist/index.html`；早前 System Events 曾返回 `windows count=0` 且 Computer Use `get_app_state` 超时。随后用完整 `.app` 路径复测，Computer Use 成功读取 `Claude Code Companion` 主窗口和 packaged renderer，System Events 返回 `front=true count=1 names=Claude Code Companion`。因此之前卡点是 macOS AX/窗口捕获状态波动，不是 renderer、server sidecar 或 update preload smoke 失败。

## Windows 待实机

- [ ] 构建 NSIS 安装器并安装。
- [x] Windows build script 已补 canonical package-smoke：`desktop/scripts/build-windows-x64.ps1` 会复制 installer/update metadata/blockmap/`win-unpacked` 到 `desktop/build-artifacts/windows-x64`，并默认运行 `bun run test:package-smoke --platform windows --package-kind release --artifacts-dir desktop/build-artifacts/windows-x64`。
- [ ] 在 Windows runner/实机运行 `bun run test:package-smoke --platform windows` 或 build script 内置 canonical package-smoke。
- [ ] 启动安装后的 app，验证 sidecar 自动启动。
- [ ] 验证 sidecar 文件锁、更新前 stop process、通知、托盘、窗口隐藏/恢复、系统对话框。
- [ ] 完成一次 Computer Use 设置和操作 smoke。

## Linux 待实机

- [ ] 构建 AppImage/deb。
- [x] Linux build script 已补 canonical package-smoke：`desktop/scripts/build-linux.sh` 支持 `LINUX_ARCH=x64|arm64`，会复制 AppImage/deb/update metadata/blockmap/`linux-unpacked` 到 `desktop/build-artifacts/linux-<arch>`，并默认运行 `bun run test:package-smoke --platform linux --package-kind release --artifacts-dir desktop/build-artifacts/linux-<arch>`。
- [x] Linux arm64 update metadata preflight 已补：`desktop/scripts/build-linux.sh` 复制 `latest-linux*.yml`，`package-smoke` 识别 `latest-linux.yml` 和 `latest-linux-arm64.yml` 并校验 metadata 引用的 AppImage/deb artifact；回归测试覆盖 canonical `linux-arm64` 输出。
- [ ] 在 Linux runner/实机运行 `bun run test:package-smoke --platform linux` 或 build script 内置 canonical package-smoke。
- [ ] 启动 AppImage/deb 安装后的 app，验证 sidecar 自动启动。
- [ ] 验证 tray、通知、系统对话框、外链、更新策略；如平台能力降级，写入 release note。

## 完成判定

- [ ] macOS packaged app 完成 Computer Use smoke。历史上已完成启动、server、终端、目录选择、preview、真实模型 chat 与 unpacked updater no-op；dev 壳另已补附件选择、Python 路径选择、外链打开和窗口关闭后 app activation 恢复证据。最终完成仍需要可启动的 signed/notarized artifact、可截图/可点击桌面会话、通知点击回跳和完整 update feed 复验。
- [ ] Windows packaged app 完成实机 smoke，或明确记录 release blocker。
- [ ] Linux packaged app 完成实机 smoke，或明确记录 release blocker。
- [x] `verify` 通过，并记录 quality report 与 coverage report 路径。最新 quality report：`artifacts/quality-runs/2026-05-31T21-42-57-279Z/report.md`；coverage report：`artifacts/coverage/2026-05-31T21-46-15-873Z/coverage-report.md`。
- [x] live release gate 通过，或明确记录 provider/live access blocker。最新 `release --allow-live` 已通过：`artifacts/quality-runs/2026-05-31T20-29-45-758Z/report.md`，`passed=15 failed=0 skipped=0`。
