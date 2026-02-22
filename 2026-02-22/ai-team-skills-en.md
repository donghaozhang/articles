# AI Team Skills: Multi-Agent Orchestration with Claude Code, Gemini, and Codex

> Source: [ThendCN/ai-team-skills](https://github.com/ThendCN/ai-team-skills)
> License: MIT

---

## 1. Overview

AI Team Skills is a set of Claude Code custom skills that enables **multi-AI agent collaboration**:

```
Claude Code (Team Lead / Orchestrator)
    ├── gemini-agent → Gemini CLI → UI/frontend design
    ├── codex-agent  → Codex CLI  → Code writing/review
    └── ai-team      → Multi-agent pipeline orchestration
```

**Core idea:** Claude Code acts as "tech lead" — delegating UI design to Gemini (gemini-3-pro-preview), code implementation to Codex (gpt-5.3-codex), while handling task decomposition, review, and integration itself.

**Key strengths of each model:**
- 🎨 Gemini excels at UI design and visual aesthetics
- 💻 Codex excels at code implementation and bug fixes (reasoning: high)
- 🧠 Claude excels at orchestration, review, and quality control

---

## 2. Architecture

### File Structure

```
ai-team-skills/
├── ai-team/                         # Multi-agent pipeline
│   ├── SKILL.md                     # Orchestration skill entry
│   └── references/
│       └── pipeline-templates.md    # 3 pipeline templates
├── codex-agent/                     # Codex code expert
│   ├── SKILL.md
│   ├── references/
│   │   └── prompt-templates.md      # 6 prompt templates
│   └── scripts/
│       ├── codex-run.sh             # Linux/macOS wrapper
│       └── codex-run.ps1            # Windows wrapper
├── gemini-agent/                    # Gemini UI expert
│   ├── SKILL.md
│   ├── references/
│   │   └── prompt-templates.md      # 5 UI templates
│   └── scripts/
│       ├── gemini-run.sh
│       └── gemini-run.ps1
└── README.md
```

### Team Roles

| Role | Agent | Model | Responsibility |
|------|-------|-------|---------------|
| Team Lead | Claude | claude-opus-4 | Task decomposition, assignment, review, integration |
| codex-worker | Codex CLI | gpt-5.3-codex | Code writing, fixing, review, testing |
| gemini-worker | Gemini CLI | gemini-3-pro-preview | UI design, frontend components, styling |

---

## 3. Three Collaboration Modes

### Mode A: UI → Implementation (Sequential)

```
gemini-worker designs UI
       ↓
Team Lead (Claude) reviews UI code
       ↓
codex-worker implements backend logic
       ↓
Run tests
```

Use case: Full-stack feature development

### Mode B: Review → Fix (Sequential)

```
codex-worker reviews code
       ↓
Team Lead (Claude) confirms issues
       ↓
codex-worker fixes issues
       ↓
Run tests
```

Use case: Code quality improvement

### Mode C: Multi-Module Parallel

```
codex-worker-1 implements Module A ─┐
codex-worker-2 implements Module B ─┤→ Claude integrates → Integration tests
gemini-worker designs UI           ─┘
```

Use case: Large features, multi-module parallel development

---

## 4. Codex Agent

### Two Modes

**exec mode (default) — Code writing/fixing**
```bash
bash codex-run.sh -f /tmp/prompt.txt -s dangerous -d ./project -o /tmp/result.txt
```

**review mode — Code review**
```bash
bash codex-run.sh -r --uncommitted -d ./project -o /tmp/review.txt
```

### Sandbox Modes

| Mode | Flag | Use Case |
|------|------|----------|
| `full-auto` | `--full-auto` | Most code writing tasks |
| `dangerous` | `--dangerously-bypass-approvals-and-sandbox` | Installing deps, running tests |
| `read-only` | `-s read-only` | Code review, analysis |

### Parallel Task Splitting

Codex runs slowly (5-15 min), so split and parallelize:
- Split by file/module
- Split by function (API + tests)
- Split by layer (frontend vs backend)
- Use Bash `run_in_background: true` for parallel execution

### Prompt Template Library (6 templates)

1. **General implementation** — Feature development
2. **Bug fix** — Diagnosis and repair
3. **API implementation** — Endpoints, validation, error handling
4. **Refactoring** — Keep interfaces, improve internals
5. **Test writing** — Happy paths and edge cases
6. **Pipeline implementation** — Logic based on Gemini's UI code

---

## 5. Gemini Agent

### Usage

```bash
bash gemini-run.sh -f /tmp/gemini-prompt.txt -d ./project
```

### Design Standards

Every Gemini prompt emphasizes:
- Semantic HTML (button, nav, main, article)
- Accessibility (ARIA, keyboard navigation, labels)
- Responsive design (mobile-first)
- State handling (loading, error, empty)
- TypeScript Props interfaces

### Prompt Template Library (5 templates)

1. **General UI design** — Any component
2. **Form components** — Validation, errors, a11y
3. **Navigation** — Responsive, keyboard navigable
4. **Modal/Dialog** — Focus trap, ESC close
5. **Dashboard** — Grid layout, data cards, skeleton screens

---

## 6. AI Team Orchestration Pipeline

### Four-Phase Execution

**Phase 1: Analysis & Decomposition**
- Identify subtask types (frontend → gemini, backend → codex)
- Determine dependencies (independent = parallel, dependent = sequential)

**Phase 2: Create Team**
- TeamCreate → TaskCreate → Launch workers → Send project context

**Phase 3: Execute & Monitor**
- Workers execute autonomously → report via SendMessage
- Team Lead reviews → unlocks dependent tasks
- Handle context passing between workers

**Phase 4: Integration & Delivery**
- Final review → run tests → report to user → cleanup

### Context Passing

1. **File paths** — Prior worker's files are in the working directory
2. **Summary passing** — Team Lead includes key info in SendMessage
3. **Task descriptions** — Subsequent tasks include prior interface definitions

---

## 7. Comparison with Other Multi-Agent Approaches

| | AI Team Skills | Claude Agent Teams | Anthropic /code-review |
|---|---|---|---|
| Orchestrator | Claude Code | Claude Code | Claude Code |
| Workers | Gemini + Codex | Multiple Claudes | Multiple Claudes |
| Model diversity | ✅ 3 vendors | ❌ Claude only | ❌ Claude only |
| Specialization | ✅ UI vs Code | ❌ Generic agents | ✅ By type |
| Cross-platform scripts | ✅ sh + ps1 | ❌ | ❌ |
| Prompt templates | ✅ 11 templates | ❌ | ❌ |
| Parallel support | ✅ Background tasks | ✅ File queues | ✅ Parallel agents |

**Key differentiator:** AI Team Skills is the only approach using **cross-vendor multi-model collaboration** — leveraging each model's unique strengths.

---

## 8. Installation & Usage

### Install

```bash
# Linux/macOS
cp -r ai-team gemini-agent codex-agent ~/.claude/skills/

# Windows (PowerShell)
@("ai-team", "gemini-agent", "codex-agent") | ForEach-Object {
    Copy-Item -Recurse $_ "$env:USERPROFILE\.claude\skills\"
}
```

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- [Codex CLI](https://github.com/openai/codex)

### Quick Start

```bash
# Delegate UI task to Gemini
/gemini-agent Design a user login page

# Delegate code task to Codex
/codex-agent Implement user authentication API

# Multi-agent collaboration
/ai-team Build complete user management (login page + API + tests)
```

---

## 9. Summary

AI Team Skills demonstrates an important trend: **different AI models have different strengths — the best results come from collaboration, not solo work.**

| Model | Best At |
|-------|---------|
| Claude | Orchestration, reasoning, review, integration |
| Gemini | UI design, visual aesthetics, frontend components |
| Codex | Code implementation, bug fixing, high-reasoning coding |

Through Claude Code's skill system, developers can launch cross-vendor multi-model collaboration with a single command — letting the right model do the right job.

---

*Based on the [ThendCN/ai-team-skills](https://github.com/ThendCN/ai-team-skills) repository.*
*Date: 2026-02-22*
