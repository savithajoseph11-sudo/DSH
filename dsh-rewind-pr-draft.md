# PR draft — LJH-dot/dsh-rewind

**Title:** fix: chat node wrapper clobbers the `chat` locale seat (raw `message.think` etc. in UI)

**Branch:** fix/locale-seat-namespace → `main` (repo ships compiled `lib/client.js` only, so the diff is in lib)

## Body (EN)

The shadowed-node renderers register with `locale: "conversation"`, but then
`makeRewindNodeView` delegates to the built-in component with the **same props**
(`react.createElement(builtin, props)`). The built-in chat nodes
(`AssistantNodeView`, `MessageItem`, …) translate their chrome through this `t`
prop, and their strings live in the `chat` namespace — under the `conversation`
binding every lookup misses and the UI shows raw keys:

> `message.think` instead of "Think" / "思考", same for `message.contextInjection` and friends.

Repro: install dsh-rewind in the web profile, open any conversation with a
reasoning row — the think row title shows the raw key. Verified via React fiber
walk: `ChatNodeSeat` receives `t("message.think") === "Think"`, the next fiber
(`RewindNodeView`) receives `t("message.think") === "message.think"`.

The wrapper itself never calls `t` (all its strings are literals), so it can
simply declare the namespace its delegate expects:

```diff
-            locale: "conversation"
+            locale: "chat"
```

Tested in a live harness web UI (hard reload after patching node_modules): all
49 reasoning rows render "Think", zero raw locale keys in the chrome, rewind
behavior (shadowing, anchors, prefill) unchanged.

## Body (中文, автору)

包装器用 `locale: "conversation"` 注册，但把 props 原样传给内置节点组件；内置组件的
文案 key（`message.think` 等）在 `chat` 命名空间里，用 `conversation` 的 `t` 查找全部
miss，界面上就直接显示原始 key。包装器自身不使用 `t`，改成 `locale: "chat"` 即可。
已在本地实测：修复后 think 行正常显示「Think/思考」，rewind 功能不受影响。
