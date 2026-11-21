# Changelog

## 0.1.1 — fork，适配 DSH 0.2.2 的 composer 插槽契约

### Fixed

- **「✨ 增强」按钮永久禁用**：`conversation.input.right` 的 props 在 DSH 0.2.2 改为 hook 形态，`props.input.draft` 不复存在，导致 `draft` 恒为空、`canRun` 恒 false。改读 `props.useInput((s) => s.draft)`。
- `props.session.sessionId` 取不到会话 id，改为读顶层的 `props.sessionId`（旧形状保留回退）。
- 客户端 `inject` 从 `[]` 改为 `['slots']`，并优先使用 `ctx.slots`（`ctx.get('slots')` 保留为回退）。

### Changed

- `store.applyDraft` 写回输入框前先记录旧值，失败时回退，避免重渲染把草稿清空。
- `package.json` 增加 `engines.dsh: ">=0.2.2-rc.3 <0.3.0-0"`：DSH 大版本改契约时安装会被明确拒绝，而不是静默失效。
- 版本号 0.1.0 → 0.1.1。

### Unchanged

- 宿主半 `lib/index.js` 未改动：路由 `/__dsh-enhance/api`、`prepare` / `complete` 链路、上下文采集（`systemPrompt.assemble` + `sessionQuery.readSurface`）、模型路由均与上游一致。
