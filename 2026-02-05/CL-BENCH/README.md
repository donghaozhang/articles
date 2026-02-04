# CL-BENCH: 开启大模型下半场的基准

**Author:** Lei Li (@_TobiasLee)  
**Source:** https://x.com/_tobiaslee/status/2019057509118742815  
**Date:** Feb 5, 2026

---

## 论文信息

**CL-BENCH: A BENCHMARK FOR CONTEXT LEARNING**

**作者团队：**
- Shihan Dou, Ming Zhang, Zhangyue Yin, Chenhao Huang, Yujlong Shen, Junzhe Wang
- Jiayi Chen, Yuchen Ni, Junjie Ye, Cheng Zhang, Huaibing Xie, Jianglu Hu
- Shaolei Wang, Weichao Wang, Yanling Xiao, Yiting Liu, Zenan Xu, Zhen Guo
- Pluto Zhou, Tao Gui, Zuxuan Wu, Xipeng Qiu, Qi Zhang, Xuanjing Huang
- Yu-Gang Jiang, Di Wang, **Shunyu Yao** (Last Author)

**机构:** Hunyuan Team (Tencent) + Fudan University

---

## 核心观点

这两年我们习惯用各种榜单回答"模型更聪明了吗"：数学、代码、考试、推理。

但 CL-BENCH 把一个更贴近真实落地、却长期被忽视的问题拎出来：

> **大模型能否在推理时（inference time）从复杂上下文中获得新知识，并据此完成任务。**

作者把这种能力命名为 **Context Learning（上下文学习）**，并用一个工程落地导向的 benchmark 去测它。

---

## 1) CL-Bench 在测什么：从 Prompt Reasoning 到 Context Learning

**论文的核心主张一句话：**

现有大模型擅长"基于参数知识的提示词推理"，但真实任务往往需要"从上下文中学习并应用新知识"。

评估模型能力不应只看 pre-training 的记忆/泛化，更要看 inference time 能否根据 context 做 adaptation。

更关键的是：当 parameter knowledge 与 context knowledge 冲突时，模型必须**忠于 context**——否则在企业私有文档、内部规范、局部规则系统里会持续"自信地答错"。

**CL-BENCH 的设计就是刻意制造这种"上下文优先"的压力：**
- 解题信息完整地放在 context 里
- 其中不少内容不存在于预训练语料，甚至可能与常识冲突
- 逼迫模型把"上下文当作当前世界的真值系统"

---

## 2) 数据集形态：500 个复杂上下文 + 1899 个任务 + 31607 条 rubrics

CL-BENCH 不追求"海量"，而是追求**高密度 + 高人工成本**：

| 指标 | 数量 |
|------|------|
| Complex contexts | 500 |
| Tasks | 1,899 |
| Verification rubrics | 31,607 |
| 平均每题 rubric | 16.6 条 |
| 每个 context 平均 | 63.2 条 |
| 上下文长度（平均） | ~10.4K tokens |
| 上下文长度（最长） | 65K tokens |
| 多轮/串联任务 | 51.1% |

任务类型明显偏"真实世界"：读 SDK/手册写代码或伪代码、按流程执行、按规则系统判定。它更像"把文档当作临时学到的知识体系来用"，而不是"读一段文章做阅读理解"。

---

## 3) 这套 benchmark 最值得肯定的三点

### 3.1 从 pre-training 走向 inference-time adaptation，且冲突时忠于 context

contexts 经常是：
- 全新虚构的法律体系/规则系统
- 修改过的现实知识（故意与常识冲突）
- 小众/长尾/新近资料（不太可能大规模进入预训练）

这正对齐我们做 **context engineering / RAG / agent memory** 的常见失败点：看到了但没学进去；学了但不会用；或被参数知识带跑偏。

### 3.2 构建方式尽可能避免 contamination（虚构文档等）

论文强调 contamination-free，主要通过：
- 专家虚构
- 对已有知识做修改变体
- 引入长尾/新兴材料

还有一个强 sanity check：**去掉 context 后，GPT-5.1 解题率掉到 0.9%**（随机 1000 题）。这基本证明它测的不是"模型本来就会"，而是"模型能不能从给定上下文里学"。

### 3.3 评估依靠 rubrics：专家给出丰富打分标准

rubric-driven verification 很工程化：
- 每条 rubric 是二元（yes/no）
- all-or-nothing：必须全通过才算 solved
- rubrics 覆盖事实、计算、程序正确性、完整性、格式合规等维度

价值在于把"看起来差不多"变成"到底哪里没做到"。

---

## 4) 最值得关注的结果：前沿模型平均只解对 17.2%

### 4.1 Overall：Context Learning 远没到可用

评测 10 个前沿模型（thinking / high reasoning effort 档）：

| 模型 | 解题率 |
|------|--------|
| 平均 | 17.2% |
| GPT-5.1 (High) | 23.7% |
| Claude Opus 4.5 Thinking | ~21% |
| 大多数模型 | 13%–18% |

**结论很"扎心但有用"：长上下文 + 强推理 ≠ 自动获得"从上下文学习并正确应用"能力。**

### 4.2 类别差异：归纳型（empirical discovery & simulation）最难

四大类里最难的是 Empirical Discovery & Simulation：
- 平均约 11.8%
- simulation environment 往往最低（多数模型 < 11%）

这类任务要求从实验/观测数据里归纳规律再预测/解释，比"给定规则直接套用"的 deductive 更难。

### 4.3 子类差异很大：同域不同知识形态差异显著

法律域对比很启发：
- **Legal & Regulatory**（规则检索/条款应用）：整体更高（很多模型 30%+，GPT-5.1 可到 40%+）
- **Legal Advisory**（场景化裁量/综合判断）：显著更低

即便同一领域，"知识呈现形态"（条款 vs 裁量）会显著改变 context learning 难度。

### 4.4 失败模式：最大问题不是"算错"，而是"不忠于上下文 + 不按要求输出"

主要错误类型：
| 错误类型 | 比例 |
|----------|------|
| Context Ignored（忽视上下文） | 55%+ |
| Context Misused（误用上下文） | 60%+ |
| Format Error（格式/约束不满足） | 35%–40% |

这几乎一一对应真实部署事故：
- 看了文档但没用（attention/selection 失败）
- 引错规则/条件绑定错（binding/grounding 失败）
- 内容大致对，但漏字段/顺序不对/没走流程而不可用（compliance 失败）

---

## 5) 两个"反直觉但重要"的观察

### 5.1 推理强度提高，收益并不大

以 GPT-5.1 为例：high vs low reasoning effort 只提升约 **+2.5%**（21.2% → 23.7%）。

说明"多想一会儿"不是银弹。Context learning 更像是"读取—内化—约束执行"的系统性短板，而不仅是多推几步。

### 5.2 长上下文仍然是硬瓶颈：越长越崩

按上下文长度分桶，几乎所有模型一致趋势：
- 0–4K tokens：约 25%–35%
- 32K+ tokens：常见只有 5%–10%

**能放进窗口 ≠ 能学会并稳定使用。**

---

## 6) 这篇 benchmark 对社区的真正价值

CL-BENCH 不只是"又一个榜单"，而是把研究问题重新对齐到真实世界：

- 真实系统越来越多能力来自 context engineering（RAG、记忆、工具、工作流），前提是模型真的能从 context 里学并执行
- 它尽量把"检索失败"和"学不会/用不好"解耦：解题信息保证都在 context 内
- rubric-based 验证让失败可定位，不是主观打分

**从这个角度看，它在强调：不要再用"模型会答题"替代"模型会工作"。**

---

## 7) 未来怎么走：从"更长上下文"到"更强上下文学习"

后续大概率三条线并行：

### 7.1 训练目标显式转向 Context Learning
用"上下文包含新知识且可能与参数知识冲突"的数据训练；强化"context 优先"的对齐与约束执行

### 7.2 引入更结构化的内化与执行机制
- 通过更显式的记忆结构与可验证中间表征
- 分层摘要、规则抽取成可执行约束
- read–extract–plan–execute–verify 的多遍阅读/迭代对齐

### 7.3 把 rubrics / verifier 变成训练信号
CL-BENCH 证明细粒度 rubrics 在评估上有效，下一步自然是用作 RL / 自训练反馈

---

## 落地视角一句话

> **未来的"强模型"不只是更会推理，而是更像一个入职当天读完内部手册就能开始干活的同事——能学、能守规矩、冲突时以当前上下文为准。**

---

## 相关链接

- 原文推文: https://x.com/_tobiaslee/status/2019057509118742815
- 作者: @_TobiasLee (Lei Li)
- 机构: Tencent Hunyuan Team, Fudan University

---

*整理于 2026-02-05*
