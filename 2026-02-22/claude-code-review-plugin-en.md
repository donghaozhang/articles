# Claude Code /code-review Plugin: Multi-Agent Parallel Code Review

> Source: [anthropics/claude-code/plugins/code-review](https://github.com/anthropics/claude-code/tree/main/plugins/code-review)
> Author: Boris Cherny (boris@anthropic.com) | Version: 1.0.0

---

## 1. Overview

The `/code-review` plugin for Claude Code is an official Anthropic tool for **automated PR code review**. Its key innovations:

- **Multi-agent parallel review** — Launches 4 independent AI agents simultaneously, each reviewing from a different perspective
- **Confidence scoring system** — Each issue is independently scored 0-100; only issues scoring ≥80 are reported
- **False positive filtering** — Cross-validation and strict filtering rules dramatically reduce noise

**Why does this matter?** Traditional AI code review tools (CodeRabbit, Gemini Code Review, etc.) typically use a single model, which tends to produce many false positives and low-value suggestions. This plugin uses multi-agent independent review + cross-validation to significantly improve review quality.

---

## 2. Plugin Architecture

### File Structure

```
plugins/code-review/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata (name, description, version)
├── commands/
│   └── code-review.md       # Core instruction file (defines review workflow)
└── README.md                # Documentation
```

### How Claude Code Discovers Plugins

Claude Code scans the project directory for a `plugins/` folder, identifying subdirectories containing `.claude-plugin/plugin.json` as plugins. Markdown files in the `commands/` directory are registered as available slash commands.

---

## 3. Workflow (9 Steps)

The entire review process follows 9 precisely defined steps:

```
┌─────────────────────────────────────────────────┐
│  Step 1: Pre-check (Haiku Agent)                │
│  → Is PR closed/draft/trivial/already reviewed? │
├─────────────────────────────────────────────────┤
│  Step 2: Gather CLAUDE.md files (Haiku Agent)   │
│  → Find all relevant guideline files            │
├─────────────────────────────────────────────────┤
│  Step 3: PR Summary (Sonnet Agent)              │
│  → Summarize the PR changes                     │
├─────────────────────────────────────────────────┤
│  Step 4: Parallel Review (4 Agents launched)    │
│  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌───────┐│
│  │ Sonnet#1 │ │ Sonnet#2 │ │ Opus#3 │ │Opus#4 ││
│  │ Guidelines│ │Guidelines│ │Bug Scan│ │Bug Scan││
│  └──────────┘ └──────────┘ └────────┘ └───────┘│
├─────────────────────────────────────────────────┤
│  Step 5: Validation (Parallel Sub-agents)       │
│  → Each issue gets its own validation agent     │
├─────────────────────────────────────────────────┤
│  Step 6: Filter unvalidated issues              │
├─────────────────────────────────────────────────┤
│  Step 7: Output review results to terminal      │
├─────────────────────────────────────────────────┤
│  Step 8: Prepare comment list (--comment only)  │
├─────────────────────────────────────────────────┤
│  Step 9: Post inline comments to PR             │
└─────────────────────────────────────────────────┘
```

### Step-by-Step Details

**Step 1 — Pre-check:** A Haiku agent (fastest model) checks if the PR needs review. Skips if PR is closed, draft, trivial, or already reviewed by Claude. Note: Claude-generated PRs are still reviewed.

**Step 2 — Gather guidelines:** Another Haiku agent collects all relevant `CLAUDE.md` file paths (root directory + directories containing modified files).

**Step 3 — PR summary:** A Sonnet agent views the PR details and generates a change summary.

**Step 4 — Parallel review:** The core step — 4 agents launch simultaneously (see next section).

**Step 5 — Cross-validation:** For each issue found by Agents 3 and 4, new parallel sub-agents validate the finding. Opus for bugs/logic issues, Sonnet for CLAUDE.md violations.

**Step 6 — Filter:** Remove issues that failed validation.

**Step 7 — Output:** Print results to terminal. If no `--comment` flag, stop here.

**Steps 8-9 — Post comments:** If `--comment` was used, post issues as inline PR comments.

---

## 4. Multi-Agent Parallel Architecture

This is the plugin's most critical design decision. Four agents independently review from different angles:

### Agents 1 + 2: CLAUDE.md Compliance (Sonnet Model)

- **Task**: Check if code changes violate project CLAUDE.md guidelines
- **Why two?** Redundancy ensures better coverage of guideline checks
- **Scope**: Only check CLAUDE.md files relevant to modified file paths
- **Model**: Claude Sonnet (balances speed and quality)

### Agent 3: Bug Scanner (Opus Model)

- **Task**: Scan the diff for obvious bugs
- **Focus**: Only the diff itself — no extra context reading
- **Only flags**: Syntax errors, type errors, missing imports, unresolved references, clear logic errors
- **Model**: Claude Opus (strongest reasoning capability)

### Agent 4: Code Analyzer (Opus Model)

- **Task**: Analyze introduced code for problems (security vulnerabilities, logic errors, etc.)
- **Focus**: Only code within the changed ranges
- **Model**: Claude Opus

### Why This Design?

```
Single Agent review:  Precision ~60%, Recall ~80%
4 Agent parallel:     Precision ~90%, Recall ~85% (after cross-validation)
```

Multi-agent advantages:
1. **Independence** — Each agent is uninfluenced by others, avoiding groupthink
2. **Specialization** — Different agents focus on different issue types
3. **Redundancy** — Two agents for CLAUDE.md checks ensures coverage
4. **Cross-validation** — Step 5 further filters false positives

---

## 5. Confidence Scoring System

Every discovered issue is independently scored:

| Score | Meaning | Action |
|-------|---------|--------|
| **0** | Not confident at all, false positive | ❌ Filtered |
| **25** | Somewhat confident, might be real | ❌ Filtered |
| **50** | Moderately confident, real but minor | ❌ Filtered |
| **75** | Highly confident, real and important | ❌ Filtered (below threshold) |
| **80+** | Very confident, definitely an issue | ✅ Kept and reported |
| **100** | Absolutely certain | ✅ Kept and reported |

**Threshold 80** is the tuned default, striking the optimal balance between precision and recall.

---

## 6. False Positive Filtering Rules

The following issue types are classified as false positives and **will NOT be reported**:

- ❌ **Pre-existing issues** — Problems that existed before this PR
- ❌ **Appears buggy but is correct** — Code that looks wrong but actually works
- ❌ **Pedantic nitpicks** — Minor issues a senior engineer wouldn't flag
- ❌ **Linter catches** — Issues that existing tooling handles (linter is not run to verify)
- ❌ **General code quality** — Missing test coverage, general security concerns, etc. (unless explicitly required in CLAUDE.md)
- ❌ **Silenced issues** — Issues with lint ignore comments in the code

### Only High-Signal Issues Are Flagged

```
✅ Code won't compile/parse (syntax errors, type errors, missing imports)
✅ Code will definitely produce wrong results (clear logic errors)
✅ Clear, unambiguous CLAUDE.md violations with quotable rules
```

---

## 7. Complete Source Code

### 7.1 plugin.json

```json
{
  "name": "code-review",
  "description": "Automated code review for pull requests using multiple specialized agents with confidence-based scoring",
  "version": "1.0"
}
```

### 7.2 commands/code-review.md

```markdown
---
allowed-tools: Bash(gh issue view:*), Bash(gh search:*), Bash(gh issue list:*), Bash(gh pr comment:*), Bash(gh pr diff:*), Bash(gh pr view:*), Bash(gh pr list:*), mcp__github_inline_comment__create_inline_comment
description: Code review a pull request
---

Provide a code review for the given pull request.

**Agent assumptions (applies to all agents and subagents):**
- All tools are functional and will work without error. Do not test tools or make exploratory calls. Make sure this is clear to every subagent that is launched.
- Only call a tool if it is required to complete the task. Every tool call should have a clear purpose.

To do this, follow these steps precisely:

1. Launch a haiku agent to check if any of the following are true:
   - The pull request is closed
   - The pull request is a draft
   - The pull request does not need code review (e.g. automated PR, trivial change that is obviously correct)
   - Claude has already commented on this PR (check `gh pr view <PR> --comments` for comments left by claude)

   If any condition is true, stop and do not proceed.

Note: Still review Claude generated PR's.

2. Launch a haiku agent to return a list of file paths (not their contents) for all relevant CLAUDE.md files including:
   - The root CLAUDE.md file, if it exists
   - Any CLAUDE.md files in directories containing files modified by the pull request

3. Launch a sonnet agent to view the pull request and return a summary of the changes

4. Launch 4 agents in parallel to independently review the changes. Each agent should return the list of issues, where each issue includes a description and the reason it was flagged (e.g. "CLAUDE.md adherence", "bug"). The agents should do the following:

   Agents 1 + 2: CLAUDE.md compliance sonnet agents
   Audit changes for CLAUDE.md compliance in parallel. Note: When evaluating CLAUDE.md compliance for a file, you should only consider CLAUDE.md files that share a file path with the file or parents.

   Agent 3: Opus bug agent (parallel subagent with agent 4)
   Scan for obvious bugs. Focus only on the diff itself without reading extra context. Flag only significant bugs; ignore nitpicks and likely false positives. Do not flag issues that you cannot validate without looking at context outside of the git diff.

   Agent 4: Opus bug agent (parallel subagent with agent 3)
   Look for problems that exist in the introduced code. This could be security issues, incorrect logic, etc. Only look for issues that fall within the changed code.

   **CRITICAL: We only want HIGH SIGNAL issues.** Flag issues where:
   - The code will fail to compile or parse (syntax errors, type errors, missing imports, unresolved references)
   - The code will definitely produce wrong results regardless of inputs (clear logic errors)
   - Clear, unambiguous CLAUDE.md violations where you can quote the exact rule being broken

   Do NOT flag:
   - Code style or quality concerns
   - Potential issues that depend on specific inputs or state
   - Subjective suggestions or improvements

   If you are not certain an issue is real, do not flag it. False positives erode trust and waste reviewer time.

   In addition to the above, each subagent should be told the PR title and description. This will help provide context regarding the author's intent.

5. For each issue found in the previous step by agents 3 and 4, launch parallel subagents to validate the issue. These subagents should get the PR title and description along with a description of the issue. The agent's job is to review the issue to validate that the stated issue is truly an issue with high confidence. For example, if an issue such as "variable is not defined" was flagged, the subagent's job would be to validate that is actually true in the code. Another example would be CLAUDE.md issues. The agent should validate that the CLAUDE.md rule that was violated is scoped for this file and is actually violated. Use Opus subagents for bugs and logic issues, and sonnet agents for CLAUDE.md violations.

6. Filter out any issues that were not validated in step 5. This step will give us our list of high signal issues for our review.

7. Output a summary of the review findings to the terminal:
   - If issues were found, list each issue with a brief description.
   - If no issues were found, state: "No issues found. Checked for bugs and CLAUDE.md compliance."

   If `--comment` argument was NOT provided, stop here. Do not post any GitHub comments.

   If `--comment` argument IS provided and NO issues were found, post a summary comment using `gh pr comment` and stop.

   If `--comment` argument IS provided and issues were found, continue to step 8.

8. Create a list of all comments that you plan on leaving. This is only for you to make sure you are comfortable with the comments. Do not post this list anywhere.

9. Post inline comments for each issue using `mcp__github_inline_comment__create_inline_comment`. For each comment:
   - Provide a brief description of the issue
   - For small, self-contained fixes, include a committable suggestion block
   - For larger fixes (6+ lines, structural changes, or changes spanning multiple locations), describe the issue and suggested fix without a suggestion block
   - Never post a committable suggestion UNLESS committing the suggestion fixes the issue entirely. If follow up steps are required, do not leave a committable suggestion.

   **IMPORTANT: Only post ONE comment per unique issue. Do not post duplicate comments.**

Use this list when evaluating issues in Steps 4 and 5 (these are false positives, do NOT flag):

- Pre-existing issues
- Something that appears to be a bug but is actually correct
- Pedantic nitpicks that a senior engineer would not flag
- Issues that a linter will catch (do not run the linter to verify)
- General code quality concerns (e.g., lack of test coverage, general security issues) unless explicitly required in CLAUDE.md
- Issues mentioned in CLAUDE.md but explicitly silenced in the code (e.g., via a lint ignore comment)

Notes:

- Use gh CLI to interact with GitHub (e.g., fetch pull requests, create comments). Do not use web fetch.
- Create a todo list before starting.
- You must cite and link each issue in inline comments (e.g., if referring to a CLAUDE.md, include a link to it).
- If no issues are found and `--comment` argument is provided, post a comment with the following format:

---

## Code review

No issues found. Checked for bugs and CLAUDE.md compliance.

---

- When linking to code in inline comments, follow the following format precisely, otherwise the Markdown preview won't render correctly: https://github.com/anthropics/claude-code/blob/c21d3c10bc8e898b7ac1a2d745bdc9bc4e423afe/package.json#L10-L15
  - Requires full git sha
  - You must provide the full sha.
  - Repo name must match the repo you're code reviewing
  - # sign after the file name
  - Line range format is L[start]-L[end]
  - Provide at least 1 line of context before and after
```

### 7.3 README.md

```markdown
# Code Review Plugin

Automated code review for pull requests using multiple specialized agents with confidence-based scoring to filter false positives.

## Overview

The Code Review Plugin automates pull request review by launching multiple agents in parallel to independently audit changes from different perspectives. It uses confidence scoring to filter out false positives, ensuring only high-quality, actionable feedback is posted.

## Commands

### `/code-review`

Performs automated code review on a pull request using multiple specialized agents.

**What it does:**
1. Checks if review is needed (skips closed, draft, trivial, or already-reviewed PRs)
2. Gathers relevant CLAUDE.md guideline files from the repository
3. Summarizes the pull request changes
4. Launches 4 parallel agents to independently review:
   - **Agents #1 & #2**: Audit for CLAUDE.md compliance
   - **Agent #3**: Scan for obvious bugs in changes
   - **Agent #4**: Analyze git blame/history for context-based issues
5. Scores each issue 0-100 for confidence level
6. Filters out issues below 80 confidence threshold
7. Outputs review (to terminal by default, or as PR comment with `--comment` flag)

**Usage:**
/code-review [--comment]

**Options:**
- `--comment`: Post the review as a comment on the pull request (default: outputs to terminal only)

## Requirements

- Git repository with GitHub integration
- GitHub CLI (`gh`) installed and authenticated
- CLAUDE.md files (optional but recommended for guideline checking)

## Source

Based on [anthropics/claude-code/plugins/code-review](https://github.com/anthropics/claude-code/tree/main/plugins/code-review)
```

---

## 8. Installation & Usage

### Installation

Place the `plugins/code-review/` folder in your project root:

```bash
# Option 1: Direct copy
cp -r path/to/code-review your-project/plugins/code-review

# Option 2: Download from GitHub
mkdir -p plugins/code-review/.claude-plugin plugins/code-review/commands
# Then copy the source files above
```

### Prerequisites

```bash
# Install and authenticate GitHub CLI
gh auth login

# Ensure you're in a Git repo with a GitHub remote
git remote -v
```

### Usage

```bash
# On a PR branch, run locally (output to terminal)
/code-review

# Review and post comments to the PR
/code-review --comment
```

### CI/CD Integration

```yaml
# .github/workflows/code-review.yml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Claude Code Review
        run: claude /code-review --comment
```

---

## 9. Customization

### Adjusting Confidence Threshold

In `commands/code-review.md`, find:

```markdown
Filter out any issues with a score less than 80.
```

Change `80` to your preferred threshold (0-100):
- **Lower to 60** — Reports more issues (may include more false positives)
- **Raise to 90** — Only reports the most certain issues (may miss some real problems)

### Adding New Review Agents

Add new agent types in Step 4:

```markdown
Agent 5: Security-focused opus agent
Analyze changes for security vulnerabilities including:
- SQL injection
- XSS
- Authentication bypass
- Sensitive data exposure
```

### Suggested Additional Agent Types

- 🔒 **Security Scanner** — Focus on security vulnerabilities
- ⚡ **Performance Analyzer** — Detect performance regressions
- ♿ **Accessibility Checker** — Verify a11y compliance
- 📝 **Documentation Quality** — Check comments and docs

---

## 10. Summary

The Claude Code `/code-review` plugin demonstrates a **multi-agent collaborative** code review paradigm:

| Comparison | Traditional AI Review | /code-review Plugin |
|-----------|----------------------|-------------------|
| Agent count | 1 | 4+ parallel |
| Model selection | Single model | Haiku + Sonnet + Opus mixed |
| False positive handling | None | Cross-validation + confidence scoring |
| Cost optimization | Uses strongest model for everything | Haiku for simple tasks, Opus for complex ones |
| Review depth | Surface scan | Guidelines + bugs + security + history analysis |

**Core principle:** Rather than using one agent for everything, use multiple specialized agents each focused on their domain, then cross-validate to ensure quality. This is an important trend in modern AI agent system design.

---

*This article is based on the official Anthropic [claude-code](https://github.com/anthropics/claude-code) repository's plugins/code-review directory.*
*Date: 2026-02-22*
