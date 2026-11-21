# Fork 说明（相对上游 0.1.0）

这个仓库是 [yaoshuo530/dsh-prompt-enhancer](https://github.com/yaoshuo530/dsh-prompt-enhancer) 的本地维护分支，用来解决 **DSH 0.2.2 起客户端插槽契约变化** 导致「✨ 增强」按钮永久禁用的问题。

## 故障现象

- 宿主侧正常：插件行 `fiberPhase: active`，`POST /__dsh-enhance/api` 的 `prepare` / `complete` 都能返回结果。
- 客户端按钮永远灰：DSH 0.2.2-rc.3 里 `conversation.input.right` 的 props 已改为 hook 形态（`useInput` / `inputActions` / `sessionId`），旧代码仍读 `props.input.draft`，于是 `draft` 恒为空、`canRun` 恒 false。

## 本 fork 的改动（全部在 `lib/client.js`）

| # | 上游写法 | 本 fork 写法 | 原因 |
|---|---|---|---|
| 1 | `exports.inject = []` | `exports.inject = ['slots']` | 契约要求显式声明对 slots 服务的依赖 |
| 2 | `ctx.get('slots')` | `ctx.slots ?? ctx.get('slots')` | 服务应直接从 ctx 取，保留回退以兼容旧版 |
| 3 | `props.input.draft` | `props.useInput((s) => s.draft)` | 新契约用选择器 hook，`input` 字段已不存在 |
| 4 | `props.session.sessionId` | `props.sessionId`（保留旧字段回退） | 新契约把 sessionId 提到 props 顶层 |
| 5 | `store.applyDraft(draft)` 直接覆写 | 覆写前读旧值、失败时回退 | 防止重渲染时草稿被清空 |

另外 `package.json` 增加了 `engines.dsh` 区间 `>=0.2.2-rc.3 <0.3.0-0`：**当未来 DSH 大版本彻底改掉插槽契约时，安装会被明确拒绝，而不是静默失效**。届时按下面的步骤重新适配即可。

## 如何验证

```powershell
node --check lib/index.js
node --check lib/client.js
```

装到 profile 后用 workspace 里的自检脚本跑四项检查（依赖 / 客户端补丁 / 宿主 API 冒烟 / 插槽契约）：

```powershell
cd C:\Users\fu\Documents\deepseek-harness\default-workspace\.tools
node check-enhancer.mjs
```

## 契约再次变化时怎么改

1. 从 DSH 安装目录的 `app.asar` 里读 `dsh/node_modules/@deepseek-ai/dsh-client-ui-conversation/src/client/contract/slots.ts`，确认 `conversation.input.right` 的 `standardProps` 现在是什么形状。
2. 同步 `dsh/node_modules/@deepseek-ai/dsh-client-ui-conversation/src/client/contract/input.ts` 里的 `InputState` / `InputActions` 签名。
3. 只改 `lib/client.js` 顶部那几行取 draft / sessionId 的代码；宿主半 `lib/index.js` 用的是稳定接口（`ctx.llm`、`ctx.webServer.register({ kind: 'prefix' })`），通常不需要动。
4. 若 DSH 大版本不再接受旧契约，把 `engines.dsh` 的区间改到新版本。

## 许可

上游为 MIT，`LICENSE` 保留原作者的版权声明（Copyright (c) 2026 Yao Shuo）。本分支改动同样以 MIT 发布。
