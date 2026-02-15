# 解剖五把 AI 手术刀：Claude Code / Codex / Gemini CLI 技术架构全拆解

**Source:** [机智流 WeChat](https://mp.weixin.qq.com/s/of3Sp9rJQq-hL7pMSv5Sxw)  
**Author:** AI Insight 研究团队  
**Date:** 2026-02-15  

## TL;DR 速览

- **五把刀，五种哲学**：Claude Code（闭源 + 子代理团队）、Codex CLI（Rust + OS 级沙箱）、Gemini CLI（免费 + 1M 上下文）、OpenCode（模型自由 + 多前端）、Cursor（IDE 原生 + 8 并行 Agent）
- **Agent Loop 分野**：Claude Code 自由循环 + 扩展思考，Codex CLI 受控循环 + 多级审批，Gemini CLI ReAct 循环 + Google Search Grounding，OpenCode 双 Agent（Plan/Build）角色分工
- **安全是最大分歧**：Codex CLI 用 macOS Seatbelt + Linux Landlock 做 OS 级沙箱，Claude Code 依赖权限模式 + Hooks 拦截，Gemini CLI 用可信文件夹
- **MCP 成为公约数**：五个工具全部支持 Model Context Protocol，这是 2026 年 CLI Coding 领域唯一达成共识的标准
- **开源格局**：Codex CLI（Apache-2.0）、Gemini CLI（Apache-2.0）、OpenCode（MIT）完全开源；Claude Code 代码公开但使用专有许可；Cursor 闭源

## 一、终端里的 AI 军备竞赛

2025 年初，Andrej Karpathy 创造了 **Vibe Coding** 这个词。一年后的 2026 年 2 月，他自己修正了说法，提出 **Agentic Engineering**——开发者 99% 的时间不在写代码，而是在编排执行任务的 Agent。

GitHub Stars（截至 2026-02-15）：
- OpenCode: 105K
- Gemini CLI: 94.5K
- Claude Code: 66.9K
- Codex CLI: 60.4K

### 五个对手，一张卡片

| 工具 | 厂商 | 语言 | 许可证 | Stars | 最新版本 |
| --- | --- | --- | --- | --- | --- |
| Claude Code | Anthropic | TypeScript | 专有许可 | 66.9K | v2.1.42 |
| Codex CLI | OpenAI | Rust | Apache-2.0 | 60.4K | v0.101.0 |
| Gemini CLI | Google | TypeScript | Apache-2.0 | 94.5K | v0.28.2 |
| OpenCode | SST (anomalyco) | TypeScript (Bun) | MIT | 105K | v1.2.4 |
| Cursor | Anysphere | TypeScript | 闭源 | — | — |

## 二、Agent Loop：大脑的运转方式

### 循环模式对比

| 维度 | Claude Code | Codex CLI | Gemini CLI | OpenCode | Cursor |
| --- | --- | --- | --- | --- | --- |
| 循环模式 | 自由循环 | 受控循环 | ReAct 循环 | 双 Agent 切换 | Agent 模式 |
| 停止条件 | 自判完成/用户中断 | 步数上限+审批 | 自判完成 | Plan→Build 切换 | 自判完成 |
| 推理增强 | Extended Thinking | 链式推理 (codex-1) | 搜索 Grounding | LSP 反馈注入 | 多模型切换 |
| 流式输出 | 直接流式 | 批量输出 | 直接流式 | SSE 事件流 | IDE 集成 |
| 错误恢复 | 自动重试+上下文保持 | 用户干预 | 自动重试 | LSP 诊断+自动修复 | 终端输出反馈 |

### 三种设计哲学

**🔓 自由循环（Claude Code）**：经典 `while(tool_call) → execute → feed results → repeat` 模式，模型生成纯文本时循环自然终止。取舍：简单可调试，但 Token 消耗不可控。

**🔒 受控循环（Codex CLI）**：异步通道驱动，审批通过 oneshot 通道解耦。取舍：安全可控+可恢复，但速度受限。

**🔄 双 Agent 分工（OpenCode）**：Plan Agent 规划（只读），Build Agent 执行（可写）。取舍：结构化，但切换有延迟。

## 三、上下文管理：有限窗口里的无限仓库

### 五种上下文策略

| 策略 | 代表工具 | 原理 | 优势 | 代价 |
| --- | --- | --- | --- | --- |
| 按需读取 | Claude Code/Codex | Glob/Grep/Read 惰性加载 | 精确，Token 高效 | 初始理解慢 |
| Repo Map | Aider→OpenCode | tree-sitter 提取符号+图排序 | 全局结构感知 | 大仓库启动慢 |
| 自动压缩 | Claude Code | 75-92% 阈值自动 Compaction | 长对话不崩 | 可能丢细节 |
| 超大窗口 | Gemini CLI | 1M Token 上下文窗口 | 塞得下更多 | 注意力稀释 |
| LSP 增强 | OpenCode | 实时诊断注入上下文 | 结构化错误信息 | 需要 LSP 服务器 |

### Claude Code 的 Compaction 机制

当上下文利用率达到约 92% 时自动触发，对话摘要压缩——保留关键事实和决策，丢弃冗余细节，重要信息转存到 `CLAUDE.md` 项目记忆文件实现跨会话持久化。标准上下文窗口为 200K tokens，Opus 4.6 支持 1M tokens（Beta）。

## 四、工具系统与扩展性

| 维度 | Claude Code | Codex CLI | Gemini CLI | OpenCode |
| --- | --- | --- | --- | --- |
| 内置工具 | ~12 种 | ~8 种 | 62 个工具文件 | 44 个工具文件 |
| MCP 支持 | 原生支持 | 是 | 是 | 是 |
| Hook 系统 | PreToolUse/PostToolUse | — | 事件 Hook | — |
| 子代理 | .claude/agents/ + Agent Teams | — | agent-scheduler | Task 工具递归 |

**MCP 是唯一的共识**：五个工具全部支持 Model Context Protocol，数千个 MCP 服务器可用，SDK 覆盖所有主流编程语言。

## 五、沙箱与安全模型

| 工具 | 沙箱机制 | 网络控制 | 文件边界 |
| --- | --- | --- | --- |
| Codex CLI | OS 级沙箱 (Seatbelt/Landlock/seccomp) | 默认禁用，白名单 | 可配置 writable roots |
| Claude Code | 权限模式 + Hook 拦截 | 不限制 | 工作目录为主 |
| Gemini CLI | 可信文件夹 + Docker/Seatbelt | 可配置 | 可信文件夹边界 |
| OpenCode | 工作目录限制 | 不限制 | 绝对路径+目录检查 |
| Cursor | Git Worktree 隔离 | VM 有网络 | IDE 项目范围 |

### 两种安全哲学

**🛡️ 系统约束派**（Codex CLI, Cursor）：不信任模型自律，用系统级限制。

**🤝 信任+验证派**（Claude Code, Gemini CLI）：给用户工具来定义边界。

## 六、多代理架构

| 工具 | 多代理模式 | 并行能力 | 协调机制 |
| --- | --- | --- | --- |
| Claude Code | 子代理 + Agent Teams | 并行子代理 | TaskList + SendMessage |
| Codex CLI | 单 Agent | — | — |
| Gemini CLI | agent-scheduler | 支持并行 | A2A 通信 |
| OpenCode | 四类 Agent | 子代理并行 | 角色分工+权限隔离 |
| Cursor | 8 并行 Agent + Background Agent | VM 并行 | contracts.md 共享协议 |

## 七、技术栈与工程取舍

| 工具 | 运行时 | 核心语言 | 关键工程决策 |
| --- | --- | --- | --- |
| Claude Code | Node.js | TypeScript | 闭源核心+开源 SDK |
| Codex CLI | 原生二进制 | Rust (96%) | 性能优先、安全优先 |
| Gemini CLI | Node.js | TypeScript | CLI/Core 分包 |
| OpenCode | Bun | TypeScript | Client/Server 架构 |
| Cursor | Electron | TypeScript | VS Code Fork |

## 八、趋势与展望

### 定价对比

| 工具 | 定价模式 | 月费 | 免费层 |
| --- | --- | --- | --- |
| Gemini CLI | 免费+付费升级 | 免费（1000 请求/天） | 最慷慨 |
| OpenCode/Aider | 开源免费 | $0（自付 API） | 完全免费 |
| Codex CLI | ChatGPT 捆绑 | $20-200/月 | 无独立免费层 |
| Claude Code | 订阅+API | $20-200/月 | 无 |
| Cursor | 订阅制 | $20-70/月 | 有限免费 |

### 五个预判

1. **多 Agent 成为标配**：到 2026 年底，所有 Tier 1 CLI 工具都将支持多代理并行
2. **安全沙箱普及**：Docker 容器化或 OS 级沙箱将从可选变为必选
3. **开源工具蚕食份额**：OpenCode + Gemini CLI 免费策略将压缩 Claude Code 的中小开发者市场
4. **MCP 治理问题浮现**：谁控制 MCP 标准的演进，将成为下一个行业议题
5. **终端 vs IDE 不会有赢家**：两种交互范式将长期共存

> "There's no meaningful difference in code quality between the tools anymore; output quality is mostly determined by how clearly the task is planned, not which tool is used."
> — 多位独立开发者评测的共识
