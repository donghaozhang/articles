# Measuring AI Agent Autonomy in Practice

**来源**: https://www.anthropic.com/research/measuring-agent-autonomy
**作者**: Miles McCain, Thomas Millar, Saffron Huang 等 (Anthropic)
**日期**: 2026-02-18
**保存日期**: 2026-02-21

---

## 核心发现

1. **Claude Code 自主运行时间越来越长** — 最长会话从 25 分钟增长到 45+ 分钟（3个月内翻倍）
2. **老用户更多自动批准，但也更频繁打断** — 新用户 20% 全自动，老用户超 40%
3. **Claude 主动暂停比人类打断更频繁** — 复杂任务中，Claude 询问澄清的频率是人类打断的两倍
4. **Agent 已进入高风险领域，但规模不大** — 软件工程占 50%，医疗、金融、网络安全有新兴用例

---

## 原文

AI agents are here, and already they're being deployed across contexts that vary widely in consequence, from email triage to cyber espionage. Understanding this spectrum is critical for deploying AI safely, yet we know surprisingly little about how people actually use agents in the real world.

We analyzed millions of human-agent interactions across both Claude Code and our public API using our privacy-preserving tool, to ask: How much autonomy do people grant agents? How does that change as people gain experience? Which domains are agents operating in? And are the actions taken by agents risky?

### Claude Code is working autonomously for longer

How long do agents actually run without human involvement? In Claude Code, we can measure this directly by tracking how much time has elapsed between when Claude starts working and when it stops.

Most Claude Code turns are short. The median turn lasts around 45 seconds. The more revealing signal is in the tail. Between October 2025 and January 2026, the 99.9th percentile turn duration nearly doubled, from under 25 minutes to over 45 minutes.

Notably, this increase is smooth across model releases. If autonomy were purely a function of model capability, we would expect sharp jumps with each new launch. The relative steadiness suggests power users building trust over time, applying Claude to increasingly ambitious tasks, and the product itself improving.

From August to December, Claude Code's success rate on internal users' most challenging tasks doubled, at the same time that the average number of human interventions per session decreased from 5.4 to 3.3.

Both measurements point to a significant **deployment overhang**, where the autonomy models are capable of handling exceeds what they exercise in practice.

### Experienced users auto-approve more frequently, but interrupt more often

Newer users (<50 sessions) employ full auto-approve roughly 20% of the time; by 750 sessions, this increases to over 40%.

Both interruptions and auto-approvals increase with experience. This reflects a shift in oversight strategy: new users approve each action before it's taken; experienced users let Claude work autonomously, stepping in when something goes wrong.

On our public API: 87% of tool calls on minimal-complexity tasks have human involvement, compared to only 67% for high-complexity tasks.

Effective oversight doesn't require approving every action—but being in a position to intervene when it matters.

### Claude Code pauses for clarification more often than humans interrupt it

As task complexity increases, Claude Code asks for clarification more often—and more frequently than humans choose to interrupt it.

**Why Claude stops itself:**
- Present user with a choice between approaches (35%)
- Gather diagnostic information or test results (21%)
- Clarify vague or incomplete requests (13%)
- Request missing credentials/access (12%)
- Get approval before taking action (11%)

**Why humans interrupt Claude:**
- Provide missing technical context or corrections (32%)
- Claude was slow, hanging, or excessive (17%)
- Received enough help to proceed independently (7%)
- Want to take the next step themselves (7%)
- Change requirements mid-task (5%)

### Agents are used in risky domains, but not yet at scale

80% of tool calls come from agents with at least one safeguard. 73% appear to have a human in the loop. Only 0.8% of actions appear irreversible.

Software engineering accounts for nearly 50% of tool calls on the public API. Beyond coding: business intelligence, customer service, sales, finance, and e-commerce.

### Recommendations

- **Model/product developers**: Invest in post-deployment monitoring infrastructure
- **Model developers**: Train models to recognize their own uncertainty
- **Product developers**: Design for user oversight — visibility + simple intervention mechanisms
- **Policymakers**: Don't mandate specific interaction patterns; focus on whether humans can effectively monitor and intervene
