# Claude Code PR Review Toolkit: 6 Specialized AI Agents for Comprehensive Code Review

> Source: [anthropics/claude-code/plugins/pr-review-toolkit](https://github.com/anthropics/claude-code/tree/main/plugins/pr-review-toolkit)
> Author: Daisy (daisy@anthropic.com) | Version: 1.0.0

---

## 1. Overview

The PR Review Toolkit is another official Claude Code review plugin. Unlike `/code-review` which runs 4 fixed parallel agents, this toolkit provides **6 specialized agents**, each focusing on a specific review dimension. Use them individually for targeted reviews or combine them for comprehensive analysis.

**Comparison with `/code-review`:**

| | /code-review | PR Review Toolkit |
|---|---|---|
| Agent count | 4 (fixed parallel) | 6 (flexible combination) |
| Review style | One-shot full review | Selective per-aspect review |
| Trigger method | Manual command | Natural language auto-trigger + command |
| Specialization depth | General review | Deep expertise per agent |
| Output | PR inline comments | Structured reports |
| Author | Boris Cherny | Daisy |

---

## 2. Plugin Architecture

### File Structure

```
plugins/pr-review-toolkit/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── commands/
│   └── review-pr.md             # Comprehensive review command
├── agents/                      # 6 specialized agents
│   ├── code-reviewer.md         # General code review
│   ├── code-simplifier.md       # Code simplification
│   ├── comment-analyzer.md      # Comment analysis
│   ├── pr-test-analyzer.md      # Test coverage analysis
│   ├── silent-failure-hunter.md # Silent failure detection
│   └── type-design-analyzer.md  # Type design analysis
└── README.md
```

### Key Difference: agents/ Directory

Unlike `/code-review` which has a single command file, PR Review Toolkit defines each agent as an independent agent file. This means:
- Each agent has its own **model selection** (Opus/Sonnet/inherit)
- Each agent has its own **color identifier**
- Each agent can be **triggered by natural language** — no command needed
- Agents are **loosely coupled** and can be flexibly combined

---

## 3. The Six Specialized Agents

### 3.1 🟢 code-reviewer (General Code Review)

**Model:** Opus (strongest reasoning)
**Role:** Project guideline compliance + bug detection + code quality

**Scoring System (0-100):**
- 0-25: Likely false positive
- 26-50: Minor nitpick not in CLAUDE.md
- 51-75: Valid but low-impact
- 76-90: Important issue
- 91-100: Critical bug or explicit violation

**Only reports issues scoring ≥80.**

**Triggers:**
```
"Review my recent changes"
"Check if everything looks good"
"Review this code before I commit"
```

---

### 3.2 🟢 code-simplifier (Code Simplification)

**Model:** Opus
**Role:** Simplify code, improve readability, preserve functionality

**Core Principles:**
- Preserve all functionality — change only how, not what
- Reduce unnecessary complexity and nesting
- Eliminate redundant code and abstractions
- **Avoid nested ternary operators** — use switch or if/else
- Clarity > brevity — explicit code beats overly compact code

**Triggers:**
```
"Simplify this code"
"Make this clearer"
"Refine this implementation"
```

**When to use:** After code passes review, as a final polish step.

---

### 3.3 🟢 comment-analyzer (Comment Analysis)

**Model:** inherit (inherits parent model)
**Role:** Analyze comment accuracy, completeness, and long-term maintainability

**Analysis Dimensions:**
1. **Factual Accuracy** — Do comments match the code implementation?
2. **Completeness** — Are critical assumptions, preconditions, and side effects documented?
3. **Long-term Value** — Will comments become outdated as code evolves?
4. **Misleading Elements** — Ambiguous language or outdated references?
5. **Improvement Suggestions** — Specific rewrite recommendations

**Output Format:**
```
Critical Issues:      Factually incorrect or seriously misleading comments
Improvements:         Comments that could be enhanced
Recommended Removals: Comments adding no value or causing confusion
Positive Findings:    Well-written comments as examples
```

**Core belief:** "Every undetected comment rot compounds into future technical debt."

---

### 3.4 🔵 pr-test-analyzer (Test Coverage Analysis)

**Model:** inherit
**Role:** Analyze test coverage quality, identify critical gaps

**Key distinction:** Focuses on **behavioral coverage**, not line coverage.

**Rating Scale (1-10):**
- 9-10: Critical functionality (data loss, security, system failures)
- 7-8: Important business logic (user-facing errors)
- 5-6: Edge cases (minor issues)
- 3-4: Nice-to-have completeness
- 1-2: Optional improvements

**Analysis Focus:**
- Untested error handling paths
- Missing boundary condition coverage
- Uncovered critical business logic branches
- Missing negative test cases
- Tests overly coupled to implementation details

---

### 3.5 🟡 silent-failure-hunter (Silent Failure Detection)

**Model:** inherit
**Role:** Find all silent failures, inadequate error handling, and inappropriate fallback behavior

**This is the strictest agent.** Core principles:

> "Silent failures are unacceptable. Any error occurring without proper logging and user feedback is a critical defect."

**Review Checklist:**

1. **Logging Quality**
   - Is the error logged with appropriate severity?
   - Does the log include sufficient context?
   - Could someone debug this issue 6 months from now?

2. **User Feedback**
   - Does the user receive clear, actionable error messages?
   - Does the message explain what the user can do?

3. **Catch Block Specificity**
   - Does the catch block catch only expected error types?
   - Could it accidentally suppress unrelated errors?

4. **Fallback Behavior**
   - Does the fallback mask the underlying problem?
   - Is the user aware they're seeing fallback behavior?

5. **Hidden Failure Patterns**
   - Empty catch blocks (absolutely forbidden)
   - Catch blocks that only log and continue
   - Returning null/undefined/defaults on error without logging
   - Optional chaining (?.) silently skipping failing operations

**Severity Levels:** CRITICAL → HIGH → MEDIUM

---

### 3.6 🩷 type-design-analyzer (Type Design Analysis)

**Model:** inherit
**Role:** Analyze type design quality and invariants

**Four-Dimensional Rating (1-10 each):**

1. **Encapsulation** — Are internal details properly hidden?
2. **Invariant Expression** — Are invariants clearly communicated through type structure?
3. **Invariant Usefulness** — Do invariants prevent real bugs?
4. **Invariant Enforcement** — Are invariants checked at construction time? Are all mutation points guarded?

**Core belief:** "Make illegal states unrepresentable."

**Common Anti-patterns Flagged:**
- Anemic domain models (types with no behavior)
- Types exposing mutable internals
- Invariants enforced only through documentation
- Types with too many responsibilities

---

## 4. The /review-pr Command

### Usage

```bash
# Full review (runs all applicable agents)
/pr-review-toolkit:review-pr

# Specific aspects
/pr-review-toolkit:review-pr tests errors    # Only tests and error handling
/pr-review-toolkit:review-pr comments        # Only comments
/pr-review-toolkit:review-pr simplify        # Only code simplification

# Parallel review (faster)
/pr-review-toolkit:review-pr all parallel
```

### Available Parameters

| Parameter | Agent | Description |
|-----------|-------|-------------|
| `comments` | comment-analyzer | Comment accuracy analysis |
| `tests` | pr-test-analyzer | Test coverage quality |
| `errors` | silent-failure-hunter | Error handling review |
| `types` | type-design-analyzer | Type design analysis |
| `code` | code-reviewer | General code review |
| `simplify` | code-simplifier | Code simplification |
| `all` | All applicable | Full review |

### Smart Matching

The system automatically determines which agents apply based on changes:
- **Always applicable**: code-reviewer
- **Test files changed**: pr-test-analyzer
- **Comments/docs changed**: comment-analyzer
- **Error handling changed**: silent-failure-hunter
- **Types added/modified**: type-design-analyzer
- **After passing review**: code-simplifier

### Output Format

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

## 5. Recommended Workflow

### Before Committing
```
1. Write code
2. Run: /pr-review-toolkit:review-pr code errors
3. Fix critical issues
4. Commit
```

### Before Creating PR
```
1. Stage all changes
2. Run: /pr-review-toolkit:review-pr all
3. Address all critical and important issues
4. Run targeted reviews to verify
5. Create PR
```

### After PR Feedback
```
1. Make requested changes
2. Run targeted reviews based on feedback
3. Verify issues are resolved
4. Push updates
```

### After Review Passes (Final Polish)
```
/pr-review-toolkit:review-pr simplify
```

---

## 6. Complete Source Code

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

1. Determine Review Scope — check git status, parse arguments
2. Available aspects: comments, tests, errors, types, code, simplify, all
3. Identify changed files via `git diff --name-only`
4. Determine applicable reviews based on changes
5. Launch review agents (sequential or parallel)
6. Aggregate results into Critical / Important / Suggestions / Strengths
7. Provide prioritized action plan
```

### 6.3 agents/code-reviewer.md (Summary)

```
Model: opus | Color: green
Role: Project guidelines compliance, bug detection, code quality
Scoring: 0-100, only reports ≥80
Reviews unstaged changes from `git diff` by default
```

### 6.4 agents/code-simplifier.md (Summary)

```
Model: opus
Role: Simplify code while preserving all functionality
Key: No nested ternaries, clarity over brevity, project standards
Triggered after code is written or after passing review
```

### 6.5 agents/comment-analyzer.md (Summary)

```
Model: inherit | Color: green
Role: Verify comment accuracy, completeness, long-term value
Advisory only — identifies issues, does not modify code
Output: Critical → Improvements → Removals → Positive Findings
```

### 6.6 agents/pr-test-analyzer.md (Summary)

```
Model: inherit | Color: cyan
Role: Test coverage quality analysis (behavioral, not line coverage)
Rating: 1-10 per gap (10 = critical)
Focus: Real bugs, not academic completeness
```

### 6.7 agents/silent-failure-hunter.md (Summary)

```
Model: inherit | Color: yellow
Role: Zero-tolerance silent failure detection
Checks: Logging quality, user feedback, catch specificity, fallback behavior
Severity: CRITICAL → HIGH → MEDIUM
Empty catch blocks = absolutely forbidden
```

### 6.8 agents/type-design-analyzer.md (Summary)

```
Model: inherit | Color: pink
Role: Type design quality analysis
4 dimensions rated 1-10: Encapsulation, Invariant Expression, Usefulness, Enforcement
Core: "Make illegal states unrepresentable"
```

---

## 7. Summary

The PR Review Toolkit represents the **modular** direction for AI code review:

| Design Decision | Rationale |
|----------------|-----------|
| 6 independent agents | Each focuses on one dimension; depth > breadth |
| Flexible composition | Use as needed, don't have to run everything |
| Natural language triggers | No commands to memorize, just ask |
| Mixed models | Opus for complex reasoning, inherit for general analysis |
| Structured output | Unified severity levels and action items |
| Recommended workflow | Full coverage from writing to merge |

**Complementary with `/code-review`:**
- `/code-review` is best for **one-shot full PR review** with auto-posted comments
- `PR Review Toolkit` is best for **continuous use during development** — targeted review at every stage from writing → testing → committing → PR

---

*Based on the official Anthropic [claude-code](https://github.com/anthropics/claude-code) repository's plugins/pr-review-toolkit directory.*
*Date: 2026-02-22*
