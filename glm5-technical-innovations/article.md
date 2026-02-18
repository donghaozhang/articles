# GLM-5 核心技术创新揭秘

在 GLM-5 发布之后，我们首次公开其核心构建方法。主要技术创新包括：

## DSA 架构采用
在保持长上下文理解能力不受损的前提下，大幅降低训练和推理成本。

## 异步强化学习基础设施（Asynchronous RL Infrastructure）
通过将生成过程与训练过程解耦，显著提升后训练阶段的效率。

## 面向智能体的强化学习算法（Agent RL Algorithms）
使模型能够更有效地从复杂、长时序（long-horizon）的交互中学习。

---

凭借上述创新，GLM-5 在开源模型中达到了当前最先进（SOTA）的性能表现，尤其在真实世界的软件工程任务中展现出非常强劲的能力。

---

## Original (English)

After the launch of GLM-5, we're pulling back the curtain on how it was built. Key innovations include:

- **DSA Adoption**: Significantly reduces training and inference costs while preserving long-context fidelity
- **Asynchronous RL Infrastructure**: Drastically improves post-training efficiency by decoupling generation from training
- **Agent RL Algorithms**: Enables the model to learn from complex, long-horizon interactions more effectively

Through these innovations, GLM-5 achieves SOTA performance among open-source models, with particularly strong results in real-world software engineering tasks.
