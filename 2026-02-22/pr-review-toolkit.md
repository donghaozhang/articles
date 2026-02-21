# Claude Code PR Review Toolkit：6 个专业 AI Agent 的全方位代码审查

> 来源：[anthropics/claude-code/plugins/pr-review-toolkit](https://github.com/anthropics/claude-code/tree/main/plugins/pr-review-toolkit)
> 作者：Daisy (daisy@anthropic.com) | 版本：1.0.0

---

## 1. 概述

PR Review Toolkit 是 Claude Code 的另一个官方代码审查插件，与 `/code-review` 插件不同，它提供了 **6 个专业化 Agent**，每个专注一个审查维度。你可以单独使用某个 Agent 做针对性审查，也可以组合使用做全方位分析。

**与 `/code-review` 的区别：**

| | /code-review | PR Review Toolkit |
|---|---|---|
| Agent 数量 | 4 个（固定并行） | 6 个（灵活组合） |
| 审查方式 | 一次性全量审查 | 可选择性审查 |
| 触发方式 | 手动输入命令 | 自然语言自动触发 + 命令 |
| 专业深度 | 通用审查 | 每个 Agent 有深度专业知识 |
| 输出方式 | PR inline 评论 | 结构化报告 |
| 作者 | Boris Cherny | Daisy |

---

## 2. 插件架构

### 文件结构

```
plugins/pr-review-toolkit/
├── .claude-plugin/
│   └── plugin.json              # 插件元数据
├── commands/
│   └── review-pr.md             # 综合审查命令
├── agents/                      # 6 个专业 Agent
│   ├── code-reviewer.md         # 通用代码审查
│   ├── code-simplifier.md       # 代码简化
│   ├── comment-analyzer.md      # 注释分析
│   ├── pr-test-analyzer.md      # 测试覆盖分析
│   ├── silent-failure-hunter.md # 静默失败猎手
│   └── type-design-analyzer.md  # 类型设计分析
└── README.md
```

### 关键区别：agents/ 目录

与 `/code-review` 只有一个 command 文件不同，PR Review Toolkit 把每个 Agent 定义为独立的 agent 文件。这意味着：
- 每个 Agent 有自己的 **模型选择**（Opus/Sonnet/inherit）
- 每个 Agent 有自己的 **颜色标识**
- 每个 Agent 可以被 **自然语言触发**，不需要输入命令
- Agent 之间 **松耦合**，可以灵活组合

---

## 3. 六大专业 Agent 详解

### 3.1 🟢 code-reviewer（通用代码审查）

**模型：** Opus（最强推理）
**职责：** 项目规范合规 + Bug 检测 + 代码质量

**评分系统（0-100）：**
- 0-25：可能是误报
- 26-50：小问题，CLAUDE.md 未提及
- 51-75：有效但低影响
- 76-90：重要问题
- 91-100：关键 Bug 或明确违规

**只报告 ≥80 分的问题。**

**触发方式：**
```
"Review my recent changes"
"Check if everything looks good"
"Review this code before I commit"
```

---

### 3.2 🟢 code-simplifier（代码简化）

**模型：** Opus
**职责：** 简化代码，提高可读性，保持功能不变

**核心原则：**
- 保留所有功能——只改写法，不改行为
- 减少不必要的复杂性和嵌套
- 消除冗余代码和抽象
- **避免嵌套三元运算符**——使用 switch 或 if/else
- 清晰 > 简洁——显式代码优于过度紧凑的代码

**触发方式：**
```
"Simplify this code"
"Make this clearer"
"Refine this implementation"
```

**使用时机：** 在代码通过审查之后使用，做最后的打磨。

---

### 3.3 🟢 comment-analyzer（注释分析）

**模型：** inherit（继承父级模型）
**职责：** 分析注释的准确性、完整性、长期可维护性

**分析维度：**
1. **事实准确性** — 注释是否与代码实现匹配
2. **完整性** — 是否缺少关键假设、前置条件、副作用
3. **长期价值** — 注释是否会随代码变化而过时
4. **误导性** — 是否存在歧义或过时引用
5. **改进建议** — 具体的重写建议

**输出格式：**
```
Critical Issues:     事实错误或严重误导的注释
Improvement:         可以增强的注释
Recommended Removals: 无价值或造成困惑的注释
Positive Findings:   写得好的注释（作为范例）
```

**核心理念：** "每一个静默的注释腐烂都会在未来复合成技术债务。"

---

### 3.4 🔵 pr-test-analyzer（测试覆盖分析）

**模型：** inherit
**职责：** 分析测试覆盖质量，识别关键缺口

**关键区分：** 关注 **行为覆盖** 而非 **行覆盖率**。

**评分标准（1-10）：**
- 9-10：关键功能（数据丢失、安全问题、系统故障）
- 7-8：重要业务逻辑（用户可见错误）
- 5-6：边界情况（轻微问题）
- 3-4：完整性补充
- 1-2：可选改进

**分析要点：**
- 未测试的错误处理路径
- 缺失的边界条件覆盖
- 未覆盖的关键业务逻辑分支
- 缺失的负面测试用例
- 测试是否过度耦合实现细节

---

### 3.5 🟡 silent-failure-hunter（静默失败猎手）

**模型：** inherit
**职责：** 找出所有静默失败、不充分的错误处理、不当的回退行为

**这是最严格的 Agent。** 核心原则：

> "静默失败是不可接受的。任何发生错误却没有适当日志和用户反馈的情况都是关键缺陷。"

**审查清单：**

1. **日志质量**
   - 错误是否用适当的严重级别记录？
   - 日志是否包含足够的上下文？
   - 6 个月后有人能凭日志调试吗？

2. **用户反馈**
   - 用户是否收到清晰、可操作的错误信息？
   - 错误信息是否说明了用户可以做什么？

3. **Catch 块特异性**
   - 是否只捕获预期的错误类型？
   - 这个 catch 块是否会意外吞掉无关错误？

4. **回退行为**
   - 回退是否掩盖了底层问题？
   - 用户是否知道正在使用回退行为？

5. **隐藏失败模式**
   - 空 catch 块（绝对禁止）
   - 只记录日志就继续的 catch 块
   - 错误时返回 null/undefined/默认值且不记录
   - 可选链 (?.) 静默跳过可能失败的操作

**严重级别：**
- CRITICAL：静默失败、宽泛 catch
- HIGH：差的错误信息、不合理的回退
- MEDIUM：缺少上下文、可以更具体

---

### 3.6 🩷 type-design-analyzer（类型设计分析）

**模型：** inherit
**职责：** 分析类型设计质量和不变量

**四维评分（每项 1-10）：**

1. **封装性 (Encapsulation)**
   - 内部实现细节是否隐藏？
   - 不变量是否可以从外部被违反？

2. **不变量表达 (Invariant Expression)**
   - 不变量是否通过类型结构清晰传达？
   - 是否在编译时尽可能强制执行？

3. **不变量实用性 (Invariant Usefulness)**
   - 不变量是否防止了真实的 bug？
   - 是否既不过于严格也不过于宽松？

4. **不变量强制执行 (Invariant Enforcement)**
   - 构造时是否检查不变量？
   - 所有变异点是否有防护？

**核心理念：** "使非法状态无法表示。" (Make illegal states unrepresentable)

**常见反模式：**
- 贫血领域模型（无行为的类型）
- 暴露可变内部的类型
- 仅通过文档强制执行的不变量
- 职责过多的类型

---

## 4. 综合审查命令 /review-pr

### 使用方式

```bash
# 全量审查（默认运行所有适用的 Agent）
/pr-review-toolkit:review-pr

# 指定审查方面
/pr-review-toolkit:review-pr tests errors    # 只审查测试和错误处理
/pr-review-toolkit:review-pr comments        # 只审查注释
/pr-review-toolkit:review-pr simplify        # 只做代码简化

# 并行审查（更快）
/pr-review-toolkit:review-pr all parallel
```

### 可选参数

| 参数 | Agent | 说明 |
|------|-------|------|
| `comments` | comment-analyzer | 注释准确性分析 |
| `tests` | pr-test-analyzer | 测试覆盖质量 |
| `errors` | silent-failure-hunter | 错误处理审查 |
| `types` | type-design-analyzer | 类型设计分析 |
| `code` | code-reviewer | 通用代码审查 |
| `simplify` | code-simplifier | 代码简化 |
| `all` | 所有适用 | 全量审查 |

### 智能匹配

系统会根据改动自动判断哪些 Agent 适用：
- **总是适用**：code-reviewer
- **测试文件改动**：pr-test-analyzer
- **注释/文档改动**：comment-analyzer
- **错误处理改动**：silent-failure-hunter
- **类型添加/修改**：type-design-analyzer
- **审查通过后**：code-simplifier

### 输出格式

```markdown
# PR Review Summary

## Critical Issues (X found)
- [agent-name]: Issue description [file:line]

## Important Issues (X found)
- [agent-name]: Issue description [file:line]

## Suggestions (X found)
- [agent-name]: Suggestion [file:line]

## Strengths
- What's well-done in this PR

## Recommended Action
1. Fix critical issues first
2. Address important issues
3. Consider suggestions
4. Re-run review after fixes
```

---

## 5. 推荐工作流

### 提交前
```
1. 写代码
2. 运行: /pr-review-toolkit:review-pr code errors
3. 修复关键问题
4. 提交
```

### 创建 PR 前
```
1. 暂存所有改动
2. 运行: /pr-review-toolkit:review-pr all
3. 处理所有关键和重要问题
4. 针对性重新审查
5. 创建 PR
```

### PR 反馈后
```
1. 做要求的改动
2. 针对性运行相关 Agent
3. 验证问题已解决
4. 推送更新
```

### 审查通过后（最终打磨）
```
/pr-review-toolkit:review-pr simplify
```

---

## 6. 完整源码

### 6.1 plugin.json

```json
{
  "name": "pr-review-toolkit",
  "version": "1.0.0",
  "description": "Comprehensive PR review agents specializing in comments, tests, error handling, type design, code quality, and code simplification"
}
```

### 6.2 commands/review-pr.md

```markdown
---
description: "Comprehensive PR review using specialized agents"
argument-hint: "[review-aspects]"
allowed-tools: ["Bash", "Glob", "Grep", "Read", "Task"]
---

# Comprehensive PR Review

Run a comprehensive pull request review using multiple specialized agents, each focusing on a different aspect of code quality.

**Review Aspects (optional):** "$ARGUMENTS"

## Review Workflow:

1. **Determine Review Scope**
   - Check git status to identify changed files
   - Parse arguments to see if user requested specific review aspects
   - Default: Run all applicable reviews

2. **Available Review Aspects:**
   - comments, tests, errors, types, code, simplify, all

3. **Identify Changed Files**
   - Run `git diff --name-only` to see modified files
   - Check if PR already exists: `gh pr view`

4. **Determine Applicable Reviews** based on changes

5. **Launch Review Agents** (sequential or parallel)

6. **Aggregate Results** into Critical / Important / Suggestions / Strengths

7. **Provide Action Plan** with prioritized recommendations
```

### 6.3 agents/code-reviewer.md

```markdown
---
name: code-reviewer
description: Review code for adherence to project guidelines, style, and best practices.
model: opus
color: green
---

Expert code reviewer. Reviews unstaged changes from `git diff` by default.

Core responsibilities:
- Project guidelines compliance (CLAUDE.md)
- Bug detection (logic errors, null handling, race conditions, security)
- Code quality (duplication, error handling, accessibility, test coverage)

Issue scoring 0-100, only reports ≥80.
Groups by severity: Critical (90-100), Important (80-89).
```

### 6.4 agents/code-simplifier.md

```markdown
---
name: code-simplifier
description: Simplify code for clarity, consistency, and maintainability while preserving functionality.
model: opus
---

Expert code simplification specialist. Principles:
1. Preserve functionality — never change what code does
2. Apply project standards (ES modules, function keyword, explicit return types)
3. Enhance clarity — reduce complexity, eliminate redundancy
4. Maintain balance — avoid over-simplification
5. Focus scope — only recently modified code

Key rules:
- Avoid nested ternary operators — use switch/if-else
- Clarity over brevity
- Remove unnecessary comments describing obvious code
```

### 6.5 agents/comment-analyzer.md

```markdown
---
name: comment-analyzer
description: Analyze code comments for accuracy, completeness, and long-term maintainability.
model: inherit
color: green
---

Meticulous code comment analyzer. Protects against comment rot.

Analyzes:
1. Factual accuracy — cross-reference claims vs actual code
2. Completeness — critical assumptions, side effects, error conditions
3. Long-term value — will comment become outdated?
4. Misleading elements — ambiguous language, outdated references
5. Improvement suggestions — specific rewrites

Output: Critical Issues → Improvements → Recommended Removals → Positive Findings
Advisory only — does not modify code directly.
```

### 6.6 agents/pr-test-analyzer.md

```markdown
---
name: pr-test-analyzer
description: Review PR test coverage quality and completeness.
model: inherit
color: cyan
---

Expert test coverage analyst. Focuses on behavioral coverage, not line coverage.

Identifies:
- Untested error handling paths
- Missing edge case coverage
- Uncovered critical business logic
- Absent negative test cases
- Tests too tightly coupled to implementation

Rating 1-10 (10 = critical). Pragmatic — focuses on tests preventing real bugs.
```

### 6.7 agents/silent-failure-hunter.md

```markdown
---
name: silent-failure-hunter
description: Identify silent failures, inadequate error handling, and inappropriate fallback behavior.
model: inherit
color: yellow
---

Elite error handling auditor. Zero tolerance for silent failures.

Non-negotiable rules:
1. Silent failures are unacceptable
2. Users deserve actionable feedback
3. Fallbacks must be explicit and justified
4. Catch blocks must be specific
5. Mock implementations belong only in tests

Severity: CRITICAL → HIGH → MEDIUM
Checks: logging quality, user feedback, catch specificity, fallback behavior, error propagation, hidden failures.
```

### 6.8 agents/type-design-analyzer.md

```markdown
---
name: type-design-analyzer
description: Expert analysis of type design, encapsulation, and invariant quality.
model: inherit
color: pink
---

Type design expert. Rates 4 dimensions (1-10):
1. Encapsulation — internal details hidden?
2. Invariant Expression — clearly communicated through structure?
3. Invariant Usefulness — prevent real bugs?
4. Invariant Enforcement — checked at construction, all mutation points guarded?

Core belief: "Make illegal states unrepresentable."
Flags: anemic models, exposed mutables, documentation-only invariants.
```

---

## 7. 总结

PR Review Toolkit 代表了 AI 代码审查的 **模块化** 方向：

| 设计决策 | 说明 |
|---------|------|
| 6 个独立 Agent | 每个专注一个维度，深度 > 广度 |
| 灵活组合 | 按需使用，不必全量运行 |
| 自然语言触发 | 不需要记命令，对话即可触发 |
| 模型混合 | Opus 做复杂推理，inherit 做一般分析 |
| 结构化输出 | 统一的严重级别和行动建议 |
| 推荐工作流 | 从写代码到合并的全链路覆盖 |

**与 `/code-review` 互补使用：**
- `/code-review` 适合 **一次性全量审查**，自动发 PR 评论
- `PR Review Toolkit` 适合 **开发过程中持续使用**，在写代码→测试→提交→PR 的每个阶段做针对性审查

---

*本文基于 Anthropic 官方 [claude-code](https://github.com/anthropics/claude-code) 仓库的 plugins/pr-review-toolkit 目录整理。*
*日期：2026-02-22*
