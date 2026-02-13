# Claude Code 逆向了它自己，发现 Agent 之间靠读写 JSON 文件聊天

> 作者：Jerome.Y. (@alterxyz4) | 2026-02-12
> 原文：https://x.com/alterxyz4/status/2021892207574405386
> 互动：259 likes, 45 reposts, 398 bookmarks, 79K views

---

## 核心发现

Claude Code 的 Agent Teams 系统建立在三个极其朴素的原语之上：

1. **文件系统消息队列** — 每个 agent 有一个 inbox JSON 文件
2. **AsyncLocalStorage** — Node.js 原生的异步上下文隔离
3. **共享任务列表** — 每个任务一个 JSON 文件

**没有消息中间件，没有数据库，没有网络通信。所有东西都是文件。**

---

## 发现过程

Claude Code 是一个 Bun 编译的单体二进制文件（~183MB）。作者问它 teammate 怎么实现的，它直接对自己的二进制跑 `strings` 命令，从编译产物里提取可读字符串。

找到的关键函数名：
- `injectUserMessageToTeammate` — 把消息注入为 user message
- `readUnreadMessages` — 读取未读消息
- `formatTeammateMessages` — 格式化 teammate 消息
- `waitForTeammatesToBecomeIdle` — 等待 teammate 空闲
- `isInProcessTeammate` — 判断是否为进程内 teammate

然后作者让 Claude Code **直接创建一个实验团队来验证逆向结论**。

---

## 文件系统结构

```
~/.claude/teams/<team-name>/
├── config.json          ← 团队配置（成员列表）
└── inboxes/
    ├── team-lead.json   ← lead 的收件箱
    └── observer.json    ← teammate 的收件箱

~/.claude/tasks/<team-name>/
└── 1.json               ← 任务文件（编号递增）
```

### config.json
```json
{
  "name": "探索实验",
  "leadAgentId": "team-lead@探索实验",
  "members": [
    { "name": "team-lead", "model": "claude-opus-4-6", "backendType": "in-process" },
    { "name": "observer", "agentType": "general-purpose", "model": "haiku", "color": "blue", "backendType": "in-process" }
  ]
}
```

### inbox 消息格式
```json
[
  {
    "from": "observer",
    "text": "你好 lead，我是 observer，我已经启动了！",
    "summary": "Observer reporting in",
    "timestamp": "2026-02-12T09:21:46.491Z",
    "color": "blue",
    "read": true
  }
]
```

Inbox 文件**按需创建**，不是预先分配的。

---

## 协议消息：JSON 套 JSON

系统级消息（空闲通知、关闭请求）把 JSON 序列化成字符串塞进 `text` 字段：

```json
{
  "from": "observer",
  "text": "{\"type\":\"idle_notification\",\"from\":\"observer\",\"idleReason\":\"available\"}"
}
```

完整生命周期：普通 DM → 任务报告 → idle_notification → shutdown_request → shutdown_approved

---

## 消息投递机制

- Teammate 消息被注入为 **user message**（和人类消息地位相同）
- **只在 conversation turn 之间投递**（agent 正在执行长 turn 时收不到消息）
- 消费者必须**主动轮询**，没有实时推送

---

## 两种运行模式

| 模式 | 说明 | 终止方式 |
|------|------|----------|
| **in-process** | 主进程内用 AsyncLocalStorage 隔离 | AbortController.abort() |
| **tmux** | 独立 tmux pane 里跑独立进程 | process.exit() |

默认 in-process，两者共用同样的 inbox 文件通信。

---

## 已知问题（全部 OPEN）

- **#23620** — Context compaction 杀死团队感知（压缩后 lead 忘记团队存在）
- **#25131** — 灾难性的 agent 生命周期失败（重复 spawn、mailbox 浪费）
- **#24130** — Auto memory 文件不支持并发（多个 teammate 同时写 MEMORY.md 会互相覆盖）
- **#24977** — 任务完成通知淹没上下文（加速 compaction 问题）
- **#23629** — 任务状态不同步

社区开发了 **Cozempic** 工具来缓解 compaction 问题。

---

## 为什么用文件系统做消息队列？

**部署成本几乎为零。** Claude Code 是 CLI 工具，`npm install` 就能用。如果依赖 Redis/RabbitMQ，用户还得额外安装。

文件系统的优势：
- 每个 OS 都有，不需要安装/配置/端口/权限
- **完全的可观察性** — `cat inbox.json` 就能看到所有消息
- `mkdir` + `writeFile` 就是全部 API

代价：
- 没有真正的原子性
- 没有实时推送（只能轮询）
- 没有 backpressure
- 并发安全靠君子协定

但在这个场景下（几十条消息、低延迟要求、2-4 个 agent）完全够用。

---

## 总结

> "先用最简单的方式跑起来，让真实用户去踩真实的坑，然后再决定哪些地方需要变复杂。"

Claude Code Agent Teams 做了一个聪明的决定：**不发明新东西**。文件系统 + JSON + AsyncLocalStorage，三样组合就得到了多 agent 通信系统。

最大优势：`~/.claude/teams/` 里一切可见，plain text，随便看。

最大局限：context compaction、生命周期管理等结构性挑战仍然 OPEN。

> "毕竟，文件系统这个东西，40 年了，还没挂过。"

---

### 快速验证

```bash
# 1. 在 Claude Code 里创建一个团队
# 2. 观察文件变化
ls -laR ~/.claude/teams/
# 3. Spawn teammate，让它发消息
# 4. 查看消息
cat ~/.claude/teams/<team>/inboxes/team-lead.json
```

---

*Collected: 2026-02-13*
