# AI Team Skills：让 Claude Code 指挥 Gemini 和 Codex 协作的多 Agent 编排系统

> 来源：[ThendCN/ai-team-skills](https://github.com/ThendCN/ai-team-skills)
> 许可：MIT

---

## 1. 概述

AI Team Skills 是一套 Claude Code 自定义技能，实现了 **多 AI Agent 协作编排**：

```
Claude Code (Team Lead / 编排者)
    ├── gemini-agent → Gemini CLI → UI/前端设计
    ├── codex-agent  → Codex CLI  → 代码编写/审查
    └── ai-team      → 多 Agent 流水线编排
```

**核心理念：** 让 Claude Code 做"技术负责人"，把 UI 设计委派给 Gemini（gemini-3-pro-preview），代码实现委派给 Codex（gpt-5.3-codex），自己负责任务拆分、审查和整合。

**三大优势：**
- 🎨 Gemini 擅长 UI 设计和视觉审美
- 💻 Codex 擅长代码实现和 bug 修复（reasoning: high）
- 🧠 Claude 擅长编排、审查和质量把控

---

## 2. 架构

### 文件结构

```
ai-team-skills/
├── ai-team/                         # 多 Agent 流水线
│   ├── SKILL.md                     # 编排技能入口
│   └── references/
│       └── pipeline-templates.md    # 3 种流水线模板
├── codex-agent/                     # Codex 代码专家
│   ├── SKILL.md                     # 技能定义
│   ├── references/
│   │   └── prompt-templates.md      # 6 种 prompt 模板
│   └── scripts/
│       ├── codex-run.sh             # Linux/macOS 包装脚本
│       └── codex-run.ps1            # Windows 包装脚本
├── gemini-agent/                    # Gemini UI 专家
│   ├── SKILL.md
│   ├── references/
│   │   └── prompt-templates.md      # 5 种 UI 模板
│   └── scripts/
│       ├── gemini-run.sh
│       └── gemini-run.ps1
├── README.md
└── README_EN.md
```

### 团队角色

| 角色 | Agent | 模型 | 职责 |
|------|-------|------|------|
| Team Lead | Claude | claude-opus-4 | 任务拆分、分配、审查、整合 |
| codex-worker | Codex CLI | gpt-5.3-codex | 代码编写、修复、审查、测试 |
| gemini-worker | Gemini CLI | gemini-3-pro-preview | UI 设计、前端组件、样式 |

---

## 3. 三种协作模式

### 模式 A：UI → 实现（串行）

```
gemini-worker 设计 UI
       ↓
Team Lead (Claude) 审查 UI 代码
       ↓
codex-worker 实现后端逻辑
       ↓
运行测试
```

适用于：全栈功能开发

### 模式 B：审查 → 修复（串行）

```
codex-worker 审查代码
       ↓
Team Lead (Claude) 确认问题
       ↓
codex-worker 修复问题
       ↓
运行测试
```

适用于：代码质量改进

### 模式 C：多模块并行

```
codex-worker-1 实现模块 A ─┐
codex-worker-2 实现模块 B ─┤→ Claude 整合 → 集成测试
gemini-worker 设计 UI    ─┘
```

适用于：大型功能开发、多模块并行

---

## 4. Codex Agent 详解

### 两种模式

**exec 模式（默认）— 代码编写/修复**
```bash
bash codex-run.sh -f /tmp/prompt.txt -s dangerous -d ./project -o /tmp/result.txt
```

**review 模式 — 代码审查**
```bash
bash codex-run.sh -r --uncommitted -d ./project -o /tmp/review.txt
```

### 沙箱模式

| 模式 | 参数 | 适用场景 |
|------|------|---------|
| `full-auto` | `--full-auto` | 大多数代码编写 |
| `dangerous` | `--dangerously-bypass-approvals-and-sandbox` | 安装依赖、运行测试 |
| `read-only` | `-s read-only` | 代码审查、分析 |

### 并行任务拆分

Codex 运行较慢（5-15 分钟），可拆分并行：
- 按文件/模块拆分
- 按功能拆分（API + 测试）
- 按层次拆分（前端 vs 后端）
- 使用 Bash `run_in_background: true` 并行执行

### Prompt 模板库

提供 6 种模板：
1. **通用代码实现** — 功能开发
2. **Bug 修复** — 问题定位和修复
3. **API 实现** — 端点定义、验证、错误处理
4. **重构** — 保持接口不变，改善内部实现
5. **测试编写** — 覆盖正常路径和边界情况
6. **流水线实现** — 基于 Gemini UI 代码实现逻辑

---

## 5. Gemini Agent 详解

### 使用方式

```bash
bash gemini-run.sh -f /tmp/gemini-prompt.txt -d ./project
```

### 设计规范

每个 Gemini prompt 都强调：
- 语义化 HTML（button, nav, main, article）
- 可访问性（ARIA, 键盘导航, label）
- 响应式（移动优先）
- 状态处理（loading, error, empty）
- TypeScript Props 接口

### Prompt 模板库

提供 5 种 UI 模板：
1. **通用 UI 设计** — 任意组件
2. **表单组件** — 验证、错误提示、无障碍
3. **导航组件** — 响应式、键盘导航
4. **模态框/对话框** — 焦点陷阱、ESC 关闭
5. **Dashboard 页面** — 网格布局、数据卡片、骨架屏

---

## 6. AI Team 编排流水线

### 四阶段执行

**Phase 1: 分析与拆分**
- 识别子任务类型（前端→gemini, 后端→codex）
- 确定依赖关系（独立并行，有依赖串行）

**Phase 2: 创建团队**
```
TeamCreate → "ai-team-{timestamp}"
TaskCreate → 创建子任务（含依赖）
Task tool → 启动 worker
SendMessage → 发送项目上下文
```

**Phase 3: 执行与监控**
- Worker 自主执行 → SendMessage 汇报
- Team Lead 审查 → 解锁依赖任务
- 处理上下文传递

**Phase 4: 整合与交付**
- 最终审查 → 运行测试 → 向用户汇报
- shutdown_request → TeamDelete 清理

### 上下文传递机制

1. **文件路径** — 前序 worker 生成的文件已在工作目录，后续 worker 直接读取
2. **摘要传递** — Team Lead 在 SendMessage 中包含关键信息
3. **任务描述** — 后续任务的 description 包含前序的接口定义

---

## 7. 与其他多 Agent 方案的对比

| | AI Team Skills | Claude Code Agent Teams | Anthropic /code-review |
|---|---|---|---|
| 编排者 | Claude Code | Claude Code | Claude Code |
| Worker | Gemini + Codex | 多个 Claude | 多个 Claude |
| 模型多样性 | ✅ 3 家厂商 | ❌ 只有 Claude | ❌ 只有 Claude |
| 专业化分工 | ✅ UI vs 代码 | ❌ 通用 Agent | ✅ 按类型分工 |
| 跨平台脚本 | ✅ sh + ps1 | ❌ | ❌ |
| Prompt 模板 | ✅ 11 种 | ❌ | ❌ |
| 并行支持 | ✅ 后台任务 | ✅ 文件队列 | ✅ 并行 Agent |

**核心差异：** AI Team Skills 是唯一一个**跨厂商多模型协作**的方案——用每个模型最擅长的能力。

---

## 8. 安装与使用

### 安装

```bash
# Linux/macOS
cp -r ai-team gemini-agent codex-agent ~/.claude/skills/

# Windows (PowerShell)
@("ai-team", "gemini-agent", "codex-agent") | ForEach-Object {
    Copy-Item -Recurse $_ "$env:USERPROFILE\.claude\skills\"
}
```

### 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 已安装
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) 已安装并登录
- [Codex CLI](https://github.com/openai/codex) 已安装并配置

### 快速使用

```bash
# 单独委派 UI 任务给 Gemini
/gemini-agent 设计一个用户登录页面

# 单独委派代码任务给 Codex
/codex-agent 实现用户认证 API

# 多 Agent 协作
/ai-team 开发完整的用户管理功能（登录页面 + API + 测试）
```

---

## 9. 总结

AI Team Skills 展示了一个重要趋势：**不同 AI 模型各有所长，最好的结果来自协作而非单打独斗。**

| 模型 | 最擅长 |
|------|--------|
| Claude | 编排、推理、审查、整合 |
| Gemini | UI 设计、视觉审美、前端组件 |
| Codex | 代码实现、bug 修复、高推理编码 |

通过 Claude Code 的 skill 系统，开发者可以用一句话启动跨厂商多模型协作，让对的模型做对的事。

---

*本文基于 [ThendCN/ai-team-skills](https://github.com/ThendCN/ai-team-skills) 仓库整理。*
*日期：2026-02-22*
