# SWE-Universe: Scaling Training Environments for Coding Agents

**Author:** Binyuan Hui (@huybery)  
**Affiliation:** Alibaba Qwen Team  
**Source:** https://x.com/huybery/status/2019072773818360270  
**Paper:** https://huggingface.co/papers/2602.02361  
**Date:** Feb 5, 2026  
**Views:** 21.8K

---

## 核心问题

> How to scale training environments for coding agents? 
> 
> **Let the agent build their own!** 🙌

---

## SWE-Universe 介绍

**SWE-Universe** 🪐 是一个可扩展的框架，能够：

- 将 **GitHub PRs** 转化为真实世界的 SWE 环境
- 支持 **多语言** (Multilingual)
- 生成 **可验证** (Verifiable) 的环境
- Agent 像人类专家一样配置每个环境
- 额外的可靠性保障措施

---

## 数据集规模对比

| Dataset | Instances | Type |
|---------|-----------|------|
| SWE-Bench | 2,294 | Python-only |
| SWE-Gym | 2,438 | Python-only |
| Multi-SWE-RL | 4,723 | Multilingual |
| SWE-rebench | 21,000 | - |
| DeepSeek-V3.2 | 24,667 | - |
| CWM | 35,000 | - |
| MiMo-V2-Flash | 90,000 | - |
| **SWE-Universe (Ours)** | **807,693** | **Multilingual** |

> 📊 SWE-Universe 比最大的竞争对手大 **9倍**！

---

## 验证与应用

已在以下场景验证：

1. **Mid-training** for Qwen3-Coder-Next
2. **Reinforcement Learning (RL)** for Qwen3-Coder-Next

未来方向：
- 进一步推进 environment synthesis
- 实现 **agent self-improvement** 的路径

---

## 相关发布：Qwen3-Coder-Next

**@Alibaba_Qwen · Feb 4:**

> 🚀 Introducing **Qwen3-Coder-Next**, an open-weight LM built for coding agents & local development.
>
> What's new:
> - 🤖 Scaling agentic training: 800K verifiable tasks + executable envs
> - 📈 Efficiency–Performance Tradeoff: achieves strong results on SWE-Bench Pro with 80B total params

---

## 社区反响

> "A dataset of these tasks, that is open-source and easy to augment through agents just running on compute machines could lead to AGI at least in coding. And it's gonna be in like 1 month from now, distributed across computers." - @Aero96193997

> "What if agents could learn from each other's environments instead of building their own from scratch? This approach could speed up the training process." - @bygregorr

> "This is amazing, any thoughts on whether this could work in domains outside of code (still verifiable, but ood for current models)" - @TheodoreGalanos

---

## Key Takeaways

1. **规模是关键**: 807K 可验证环境，远超现有数据集
2. **多语言支持**: 不仅限于 Python
3. **Agent 自主配置**: 让 agent 自己构建训练环境
4. **Self-improvement 路径**: 指向 coding agent 的自我提升
5. **开放权重**: Qwen3-Coder-Next 是开放模型

---

## Links

- Paper: https://huggingface.co/papers/2602.02361
- Tweet: https://x.com/huybery/status/2019072773818360270
- Qwen: https://twitter.com/Alibaba_Qwen
- Author: https://twitter.com/huybery

---

## Hashtags

#SWEUniverse #Qwen #CodingAgents #SWEBench #OpenSource #Alibaba #AGI

---

*整理于 2026-02-05*
