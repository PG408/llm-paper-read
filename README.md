# LLM Paper Read

> 仓库是根据 [@TheAhmadOsman: The Top 26 Essential Papers (+5 Bonus Resources)
for Mastering LLMs and Transformers](https://x.com/TheAhmadOsman/status/2016893734986616915) 利用 [ljg-xray-paper skill](https://github.com/lijigang/ljg-skill-xray-paper/tree/master) 生成的论文阅读笔记（`notes` 目录下的 org 文件）。  
> 本文档对上述笔记进行综述化整理：给出研究与工程主线、关键拐点，并提供可跳转索引；同时用简短 TL;DR 总结反复出现的约束条件。

## PLAIN TALK

- 这组笔记聚焦于大语言模型的能力来源与系统性约束：能力提升的关键因素、可复现的工程路径、以及常见失效模式。
- 主线可理解为：**架构范式** → **预训练/提示** → **规模与算力分配** → **工程与效率** → **稀疏扩容** → **对齐** → **推理/工具/检索** → **可解释与诊断**。
- 阅读顺序：先看 `LANDSCAPE` 的时间线找拐点，再按 `THEMES` 的分主题脉络深入，最后用 `INDEX` 快速跳转到对应笔记。

## HOW TO USE

- 看主线时间线与关键拐点：从 `LANDSCAPE` 入手。
- 按主题系统梳理与串联：读 `THEMES`，再跳转到 `notes/` 的具体论文笔记。
- 快速定位单篇与回看 takeaways：用 `INDEX` 表格按标签检索与跳转。
- 约束 TL;DR（常反复出现）：系统吞吐/显存/通信是硬门槛；数据质量与评测设计决定可解释的增益归因；固定预算下训练配比约束显著；对齐引入目标与分布权衡；检索/工具/多步执行把问题转为端到端可靠性治理（相关论文见 `INDEX`）。

## LANDSCAPE

<pre>
Pre 2017    RNN/LSTM + seq2seq attention era
      |     <i>预 Transformer：主要依赖循环网络，训练并行度受限，长程依赖建模困难。</i>
      |
      v
    2017-01 Sparsely-Gated MoE layer &lt;== [TURN]
      |     <i>稀疏专家：在每 token 仅激活少数专家的条件下显著扩展容量，将容量与单位 token 计算部分解耦。</i>
      |     - sparse capacity: activate a few experts per token
      |       <i>稀疏计算：每个 token 仅通过少数专家，从而在近似固定计算量下容纳更多参数。</i>
      |     - limitation: routing/balance/comms can dominate
      |       <i>代价：路由塌缩、负载不均衡与通信开销可能显著削弱收益。</i>
      v
    2017-06 Transformer (self-attention only) &lt;== [TURN]
      |     <i>注意力统一范式：高度并行、长程依赖路径更短，成为后续一切的底座。</i>
      |     - parallel training; short path for long-range dependencies
      |       <i>优势：训练并行化程度更高，长程信息传递路径更短。</i>
      |     - limitation: attention cost grows with sequence length (~ n*n)
      |       <i>瓶颈：注意力计算与内存开销随序列长度近似二次增长（~ n*n）。</i>
      v
    2018-10 BERT (bidirectional encoder pretrain) &lt;== [TURN]
      |     <i>双向预训练：通过掩码词恢复学习“左右一起看”的表示，统一迁移到理解任务。</i>
      |     - masked token recovery + sentence relation objective
      |       <i>训练信号：完形填空 + 句间关系，强化句内语义与句间衔接。</i>
      |     - limitation: mask-token mismatch; context cap
      |       <i>局限：训练时出现的 mask 在推理时不存在；上下文长度也有硬上限。</i>
      v
    2019-02 GPT-2 (unsupervised multitask)
      |     <i>规模 + 数据治理：纯下一个词预测在更大规模与更高质量语料下呈现跨任务迁移能力。</i>
      |     - scale + cleaner data makes zero-shot more viable
      |       <i>要点：在相同目标下，规模与数据质量的提升显著影响泛化能力。</i>
      v
    2020-01 Scaling laws (Kaplan) &lt;== [TURN]
      |     <i>可预算化的规模研究：用幂律将参数/数据/算力与损失关联，以支持预算规划。</i>
      |     - predictable power laws -&gt; compute budgeting &amp; recipe planning
      |       <i>意义：由经验驱动转向预算驱动的配方设计与消融验证。</i>
      v
    2020-05 GPT-3 (few-shot via prompting) &lt;== [TURN]
      |     <i>提示与上下文学习：通过提示与少量示例执行新任务，少样本能力随规模提升而增强。</i>
      |     - in-context learning becomes a main usage paradigm
      |       <i>形态：在上下文中编码任务指令与示例，而非依赖参数更新。</i>
      |     - limitation: prompt sensitivity; factuality &amp; reasoning brittleness
      |       <i>局限：对提示高度敏感；事实性与推理稳定性仍存在不足。</i>
      v
    2020-05 RAG (retrieve then generate) &lt;== [TURN]
      |     <i>知识外置：先检索证据再生成，以外部语料约束与更新知识来源。</i>
      |     - knowledge moves to a replaceable corpus; evidence-conditioned answers
      |       <i>收益：可通过替换文库更新知识，并以证据提升可追溯性。</i>
      v
    2021-01 Switch Transformer (top-1 MoE at scale)
      |     <i>工程化 MoE：每 token 只选 1 个专家，简化路由以提高稳定性与吞吐。</i>
      |     - simpler routing to stabilize trillion-parameter capacity
      |       <i>要点：用更简单的稀疏策略换来更可控的大规模训练。</i>
      v
    2021-04 RoFormer / RoPE (rotary relative position)
      |     <i>相对位置外推：把位置信息编码到旋转角度，更利于长序列泛化。</i>
      |     - better length extrapolation without a fixed position table
      |       <i>优势：不依赖固定长度的位置表，更适用于长上下文外推。</i>
      v
    2021-12 GLaM (top-2 MoE efficiency)
      |     <i>高效稀疏扩容：top-2 激活在能力与稳定性之间做折中，强调能耗/成本。</i>
      |     - large capacity with lower energy / FLOPs per token
      |       <i>要点：容量显著扩大，但每 token 的有效计算量接近稀疏激活规模。</i>
      v
    2022-01 Chain-of-Thought prompting &lt;== [TURN]
      |     <i>显式化推理轨迹：通过示例促使模型输出中间推理步骤。</i>
      |     - exemplars induce multi-step reasoning traces
      |       <i>机制：提供“题目 -&gt; 推理 -&gt; 答案”的示例格式，促进分步推理。</i>
      |     - limitation: format-dependent; can look right but be wrong
      |       <i>局限：效果对格式依赖较强；推理链的可读性不保证正确性。</i>
      v
    2022-03 InstructGPT / RLHF &lt;== [TURN]
      |     <i>对齐工程化：将指令遵循与安全性纳入可优化目标。</i>
      |     - align outputs to user intent &amp; safety via preference feedback
      |       <i>路径：SFT + 偏好反馈（奖励/偏好目标）塑造助手行为。</i>
      |     - limitation: preference bias; possible capability trade-offs
      |       <i>代价：偏好分布带来新偏差，也可能牺牲部分能力或多样性。</i>
      v
    2022-03 Chinchilla (compute-opt) &lt;== [TURN]
      |     <i>算力最优转向：指出大量大模型存在训练 token 不足；在固定算力下，更小模型配合更多 tokens 通常更具算力效率。</i>
      |     - many big LMs were undertrained; more tokens often wins
      |       <i>结论：在固定算力下，训练得更久（更多数据）常常比更大参数更划算。</i>
      v
    2022-04 PaLM + Pathways (system gate)
      |     <i>系统门槛显性化：规模能否兑现取决于训练效率与稳定性（MFU）。</i>
      |     - training efficiency (MFU) becomes a first-class scaling constraint
      |       <i>含义：同样算力下，能否“吃满”硬件决定了可达的训练规模上限。</i>
      v
    2022-05 FlashAttention (exact, IO-aware) &lt;== [TURN]
      |     <i>缓解注意力的 IO 瓶颈：在保持精确计算的前提下降低显存占用并提升吞吐。</i>
      |     - memory/IO bottleneck -&gt; tiled streaming attention
      |       <i>方法：分块流式计算 softmax，避免显式物化 n×n 注意力矩阵。</i>
      v
    2022-12 Sparse Upcycling (MoE from dense checkpoints)
      |     <i>从 dense 转换为 MoE：复用已有 checkpoint，以较小追加训练预算获取稀疏扩容增益。</i>
      |     - convert dense MLPs into experts; small extra budget for gains
      |       <i>技巧：复制部分 MLP 形成多个专家 + 路由器，再少量续训。</i>
      v
    2023-02 LLaMA (longer training on public data) &lt;== [TURN]
      |     <i>开源基座与长训练：在较低推理成本下接近或超过更大规模基线模型。</i>
      |     - cost-effective open base models
      |       <i>信息：长训练 + 数据治理让“性价比”成为竞争变量。</i>
      v
    2023-05 DPO (preference optimization without RL)
      |     <i>简化偏好优化：避免强化学习堆栈，使训练流程更稳定且更易工程化。</i>
      |     - simpler, more stable alternative to RLHF training stacks
      |       <i>意义：降低对齐门槛，减少不稳定与调参成本。</i>
      v
    2023-06 Textbooks Are All You Need (quality-first code data)
      |     <i>数据质量优先：教材式讲解与练习驱动，提升小模型的代码生成能力。</i>
      |     - textbook + exercises make small models strong at coding
      |       <i>要点：结构化高质量数据在单位算力下更具样本效率。</i>
      v
    2023      ReAct (reasoning + acting agents) &lt;== [TURN]
      |     <i>代理范式：将推理轨迹与外部行动（工具/搜索）交织，降低无依据生成并提升可追溯性。</i>
      |     - tool use + action loop improves faithfulness and debuggability
      |       <i>收益：行动约束生成，轨迹可追踪，错误更易定位。</i>
      v
    2024-01 Mixtral (open MoE tradeoff)
      |     <i>开源 MoE：在大容量条件下保持稀疏计算，推理成本更接近中等规模模型。</i>
      |     - strong capacity/compute tradeoff with open weights
      |       <i>要点：能力接近大模型，但每 token 仅激活部分参数参与计算。</i>
      v
    2024      Scaling Monosemanticity (interpretable features) &lt;== [TURN]
      |     <i>可解释特征提取：用稀疏字典提取可命名、可干预的内部特征。</i>
      |     - sparse dictionaries expose controllable internal features
      |       <i>意义：不仅支持解释，还支持通过特征干预稳定改变输出行为。</i>
      v
    2024-05 Platonic Representation Hypothesis
      |     <i>表示收敛假说：强模型的内部表征跨模态/架构趋于一致。</i>
      |     - stronger models' representations converge across modalities
      |       <i>观点：世界统计结构相同，学得越好越会收敛到“同一套表示”。</i>
      v
    2025-01 DeepSeek-R1 (RL-for-reasoning) &lt;== [TURN]
      |     <i>以强化学习优化推理能力：在可自动判分任务上奖励正确性，减少对人工链式示范的依赖。</i>
      |     - incentivize reasoning via RL on auto-verifiable tasks
      |       <i>动机：通过自监督可验证信号促进推理策略探索。</i>
      v
    2025-05 Qwen3 (/think vs /no_think, thinking budget)
      |     <i>可控推理深度：通过开关与预算约束，在延迟与能力之间进行可控权衡。</i>
      |     - controllable reasoning depth in one model
      |       <i>用途：在同一模型内支持不同推理预算，降低多模型路由复杂度。</i>
      v
    2025-10 Smol Training Playbook (training as decision loop)
      |     <i>训练的决策闭环：infra/ablation/eval/监控构成长期可迭代主线。</i>
      |     - infra + ablation + eval + monitoring become the main storyline
      |       <i>结论：主要瓶颈通常来自系统与流程，而非单一技巧选择。</i>
      v
Pos 2025    LLMs become an end-to-end system problem
      <i>后 2025：模型能力、数据、系统、对齐、工具与评测被绑定成一个整体工程系统。</i>
</pre>

- 关键拐点锚点（笔记）：
  - Transformer 范式： [Attention Is All You Need][n_aiayn]
  - 预训练迁移范式： [BERT][n_bert]
  - few-shot 主范式化： [Language Models are Few-Shot Learners][n_gpt3]
  - 算力最优转向： [Training Compute-Optimal Large Language Models][n_chinchilla]
  - 稀疏扩容路线： [Switch Transformers][n_switch] / [GLaM][n_glam]
  - 对齐产品化： [Training language models to follow instructions with human feedback][n_instructgpt]
  - 代理形态： [ReAct][n_react]
  - 可解释特征： [Scaling Monosemanticity][n_monosemanticity]

## THEMES

### 1) 基座范式与预训练

- **核心痛点**：如何用一个通用模型覆盖大量任务，而不是为每个任务重训一套系统。
- **解题机制**：
  - 用 Transformer 统一序列建模骨架，把“并行训练 + 长依赖路径短”变成默认。
  - 用预训练目标与微调/提示，把知识与能力迁移到下游任务。
  - 通过规模化与更长训练，使零/少样本能力随规模提升而增强（但存在显著任务差异）。
- **创新增量**：
  - 预训练 → 迁移/提示的范式逐步固定，成为后续能力扩展的底座。
  - 开源高质量基座让工程可控性（成本、可部署性）成为研究变量的一部分。
- **批判性边界**：
  - few-shot/zero-shot 对 prompt 较敏感；基准提升不等同于生产稳定性。（[GPT-3][n_gpt3]）
  - 预训练目标与真实输入存在错配（mask token、长上下文、工具交互等）。（[BERT][n_bert]）
  - 同一模型的多模式输出（如思考开关）可能混淆评测信号与产品信号。（[Qwen3][n_qwen3]）
- **推荐阅读顺序**：
  - [Attention Is All You Need][n_aiayn] -> [BERT][n_bert] -> [Language Models are Unsupervised Multitask Learners][n_gpt2] -> [Language Models are Few-Shot Learners][n_gpt3] -> [LLaMA][n_llama] -> [Qwen3 Technical Report][n_qwen3]

### 2) 规模定律与算力最优

- **核心痛点**：算力预算固定时，参数规模、数据量与训练步数如何配置以最大化单位算力收益。  
- **解题机制**：
  - 用可拟合的幂律，把“效果 vs 规模/数据/算力”变成可预测、可规划的问题。
  - 用 compute-opt 重新校准训练配比：许多模型并非参数不足，而是训练 tokens 不足。
  - 把系统效率（能否把峰值算力兑现成有效训练）纳入结论的一部分。
- **创新增量**：
  - 研发从经验驱动的参数扩展转向预算驱动的训练配方设计与消融验证。
  - 解释了为何某些模型在更大规模下会出现“看似跃迁”的现象（但是否可预测仍开放）。
- **批判性边界**：
  - 幂律与 compute-opt 对数据分布与质量敏感，跨语料/跨目标可能漂移。（[Scaling Laws][n_kaplan] / [Chinchilla][n_chinchilla]）
  - 系统能力强弱会影响可复现性：部分结论依赖训练过程的稳定完成与高效率兑现。（[PaLM][n_palm]）
- **推荐阅读顺序**：
  - [Scaling Laws for Neural Language Models][n_kaplan] -> [Training Compute-Optimal Large Language Models][n_chinchilla] -> [PaLM][n_palm]

### 3) 训练工程与效率

- **核心痛点**：在现实系统约束下，同样的算法是否可训练、可复现且具备足够吞吐与成本可控性。  
- **解题机制**：
  - 用可外推的位置编码与更高效的 attention 实现，使长上下文训练具备工程可行性。
  - 用教材-练习式的数据组织，提升小模型在特定能力（如代码）上的单位算力收益。
  - 用 playbook 把训练拆成闭环：目标定义 -> ablation -> 自动评测 -> 监控与基础设施。
- **创新增量**：
  - 在不做近似的前提下显著降低 attention 的内存/IO 瓶颈。（[FlashAttention][n_flashattn]）
  - 将数据质量作为可设计变量，而非仅依赖更大规模抓取。（[Textbooks][n_textbooks]）
  - 总结可复用的故障处理与迭代流程，而非仅给出最终 config。（[Smol][n_smol]）
- **批判性边界**：
  - 工程细节强依赖具体硬件/栈；换平台后结论可能失效或需重做 profile。（[PaLM][n_palm] / [Smol][n_smol]）
  - 长上下文收益高度依赖评测设计；评测噪声可能导致表观提升。（[RoFormer][n_rope] / [FlashAttention][n_flashattn]）
- **推荐阅读顺序**：
  - [RoFormer][n_rope] -> [FlashAttention][n_flashattn] -> [Textbooks Are All You Need][n_textbooks] -> [Smol Training Playbook][n_smol]

### 4) 稀疏化与 MoE 扩容

- **核心痛点**：想要更大“容量”（更多参数/知识），但不想让每个 token 的计算量等比例上涨。  
- **解题机制**：
  - 用路由器把 token 分发到少数专家（top-1/top-2），并用负载均衡避免专家塌缩。
  - 用更简单的路由与工程技巧，让 MoE 的稳定性/通信开销可控。
  - 用“稀疏 upcycling”把已有 dense checkpoint 以较低追加成本改造成 MoE，再通过追加训练获得增益。
- **创新增量**：
  - 把“万亿参数级容量”变成可训练、可评测的现实路线。（[Switch][n_switch] / [GLaM][n_glam]）
  - 以更可复用的开源形式提供强 MoE 权重与明确的成本-性能权衡。（[Mixtral][n_mixtral]）
- **批判性边界**：
  - 隐性成本常被低估：路由带来的通信/编译/负载不均衡，可能显著削弱理论收益。（[Sparsely-Gated MoE][n_moe] / [Switch][n_switch]）
  - 稀疏扩容不自动带来能力：数据、训练配比与稳定性仍是核心约束。（[GLaM][n_glam] / [Chinchilla][n_chinchilla]）
  - upcycling 的有效性依赖原 checkpoint 质量与“专家可分解性”，存在适用条件。（[Sparse Upcycling][n_sparse_upcycling]）
- **推荐阅读顺序**：
  - [Sparsely-Gated MoE][n_moe] -> [Switch Transformers][n_switch] -> [GLaM][n_glam] -> [Sparse Upcycling][n_sparse_upcycling] -> [Mixtral of Experts][n_mixtral]

### 5) 对齐：指令与偏好优化

- **核心痛点**：预训练目标主要优化文本续写分布，而产品场景强调指令遵循、可控性与降低无依据生成。  
- **解题机制**：
  - 用人类示范做监督微调（SFT），将输出分布调整至更符合指令遵循的分布。
  - 用偏好数据训练奖励/偏好目标，使回答质量在偏好空间中可优化。
  - 用更直接的目标（如 DPO）减少 RL 复杂度与不稳定性。
- **创新增量**：
  - 将对齐从研究概念落到可规模化流程，显著提升指令遵循与安全性。（[InstructGPT][n_instructgpt]）
  - 把偏好优化变得更轻量，降低工程门槛。（[DPO][n_dpo]）
- **批判性边界**：
  - 偏好数据的分布决定了模型行为边界；奖励/偏好目标容易引入新偏差。（[InstructGPT][n_instructgpt] / [DPO][n_dpo]）
  - 对齐常伴随能力折损、风格同质化或“过度迎合”，需要单独评测与治理。（[InstructGPT][n_instructgpt]）
- **推荐阅读顺序**：
  - [Training language models to follow instructions with human feedback][n_instructgpt] -> [Direct Preference Optimization][n_dpo]

### 6) 能力外延：推理/代理/检索/可解释

- **核心痛点**：在语言生成之外，提升检索、推理、工具调用能力，并增强系统可诊断性。  
- **解题机制**：
  - 用 CoT 将中间推理过程显式化，以提升多步推理任务表现（并对提示较敏感）。
  - 用 ReAct 将推理轨迹与外部行动/工具调用结合，以降低无依据生成并提升可追溯性。
  - 用 RAG 把知识外置到可替换的文库，用证据约束生成。
  - 用强化学习直接奖励“推理正确”，减少昂贵的人工链式示范。（[DeepSeek-R1][n_deepseek_r1]）
  - 用稀疏字典/特征开关做可解释与可干预诊断。（[Monosemanticity][n_monosemanticity]）
- **创新增量**：
  - 把“工具/检索/代理”纳入模型能力边界的一部分，而非只看语言困惑度。（[RAG][n_rag] / [ReAct][n_react]）
  - 把“可解释”从只读分析推进到可控干预的方向。（[Monosemanticity][n_monosemanticity]）
- **批判性边界**：
  - 检索质量与工具可靠性是系统短板：检索误差与行动失败可能产生级联放大效应。（[RAG][n_rag] / [ReAct][n_react]）
  - 推理链路与 RL 容易过拟合评测格式；需要更强的分布外验证。（[CoT][n_cot] / [DeepSeek-R1][n_deepseek_r1]）
  - “表示收敛/特征可解释”仍是局部结论，跨模型/跨层/跨任务的稳定性需谨慎。（[Platonic Rep][n_platonic] / [Monosemanticity][n_monosemanticity]）
- **推荐阅读顺序**：
  - [Retrieval-Augmented Generation][n_rag] -> [Chain-of-Thought Prompting][n_cot] -> [ReAct][n_react] -> [DeepSeek-R1][n_deepseek_r1] -> [Scaling Monosemanticity][n_monosemanticity] -> [The Platonic Representation Hypothesis][n_platonic]

## INDEX

| 标签 | 标题（含 venue/年份信息） | 一句话 takeaway | 笔记 | 原文 |
| --- | --- | --- | --- | --- |
| 基座/预训练 | Attention Is All You Need (NeurIPS 2017) | 用纯自注意力替代 RNN/CNN，提速提质，奠定 Transformer 基座。 | [note][n_aiayn] | [source][s_aiayn] |
| 基座/预训练 | BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding (NAACL-HLT 2019) | 掩码词预测+句间关系预训练，统一 encoder 微调并刷新理解基准。 | [note][n_bert] | [source][s_bert] |
| 基座/预训练 | Language Models are Unsupervised Multitask Learners (OpenAI Technical Report, 2019) | 扩大 LM 规模+更干净数据，纯下一个词预测也能免微调跨任务。 | [note][n_gpt2] | [source][s_gpt2] |
| 基座/预训练 | Language Models are Few-Shot Learners (arXiv 2020) | 把模型做大后，靠提示+少量示例零微调做新任务，暴露推理与事实短板。 | [note][n_gpt3] | [source][s_gpt3] |
| 基座/预训练 | LLaMA: Open and Efficient Foundation Language Models (arXiv 2023) | 只用公开数据长训练小模型，用更低推理成本逼近/超过更大闭源模型。 | [note][n_llama] | [source][s_llama] |
| 基座/预训练 | Qwen3 Technical Report (arXiv 2025) | 同一模型内置 /think 与 /no_think 开关，用思考预算在延迟与能力间折中。 | [note][n_qwen3] | [source][s_qwen3] |
| 规模/算力 | Scaling Laws for Neural Language Models (arXiv 2020) | 损失随参数/数据/算力呈稳定幂律，用公式做预算分配且不必训到收敛。 | [note][n_kaplan] | [source][s_kaplan] |
| 规模/算力 | Training Compute-Optimal Large Language Models (arXiv 2022) | 指出大量大模型训练 tokens 不足；在固定算力下，更小模型配合更多 tokens 通常更具算力效率。 | [note][n_chinchilla] | [source][s_chinchilla] |
| 规模/算力 | PaLM: Scaling Language Modeling with Pathways (arXiv 2022) | 用 Pathways 高效训稳 540B PaLM，并在部分任务上观察到规模跃迁。 | [note][n_palm] | [source][s_palm] |
| 训练/效率 | RoFormer: Enhanced Transformer with Rotary Position Embedding (arXiv 2021) | 把位置信息变成 Q/K 旋转角度，使注意力天然编码相对位置并外推更长。 | [note][n_rope] | [source][s_rope] |
| 训练/效率 | FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness (NeurIPS 2022) | 分块流式算 softmax，不落 n×n 注意力矩阵，显存近线性且结果不变。 | [note][n_flashattn] | [source][s_flashattn] |
| 训练/效率 | Textbooks Are All You Need (arXiv 2023) | 采用教材式高质量示例与练习数据，使小模型在代码基准上达到竞争水平。 | [note][n_textbooks] | [source][s_textbooks] |
| 训练/效率 | The Smol Training Playbook: The Secrets to Building World-Class LLMs (Hugging Face, 2025) | 总结从零训练小模型的关键风险点与工程闭环：目标设定、消融、自动评测与基础设施。 | [note][n_smol] | [source][s_smol] |
| MoE/稀疏 | Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer (arXiv 2017) | 用门控路由只激活少数专家，在近似不增算力下扩大参数容量并提效。 | [note][n_moe] | [source][s_moe] |
| MoE/稀疏 | Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity (JMLR) | 每 token 只选 1 个专家，简化 MoE 训练与通信，实现万亿参数稀疏扩容。 | [note][n_switch] | [source][s_switch] |
| MoE/稀疏 | GLaM: Efficient Scaling of Language Models with Mixture-of-Experts (ICML 2022) | 以 MoE 在较低能耗与 FLOPs/token 下扩展容量，并在多任务 few-shot 上取得优势。 | [note][n_glam] | [source][s_glam] |
| MoE/稀疏 | Sparse Upcycling: Training Mixture-of-Experts from Dense Checkpoints (arXiv 2022) | 从密集模型复制 MLP 成多专家再续训，小预算把 dense 变 MoE 提升性能。 | [note][n_sparse_upcycling] | [source][s_sparse_upcycling] |
| MoE/稀疏 | Mixtral of Experts (arXiv 2024) | 8x7B 每 token 激活 2 专家，以较低推理成本逼近 70B 级效果并开源。 | [note][n_mixtral] | [source][s_mixtral] |
| 对齐/偏好 | Training language models to follow instructions with human feedback (arXiv 2022) | 用人类示范+偏好反馈做 SFT+RLHF，让模型更听指令且更安全。 | [note][n_instructgpt] | [source][s_instructgpt] |
| 对齐/偏好 | Direct Preference Optimization: Your Language Model is Secretly a Reward Model (arXiv 2023) | 将偏好对转为直接优化目标，避免 RL 堆栈，使训练流程更简洁且更稳定。 | [note][n_dpo] | [source][s_dpo] |
| 能力外延 | Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks (arXiv 2020) | 把检索与生成端到端结合，让模型先找证据再回答，可替换文库更新知识。 | [note][n_rag] | [source][s_rag] |
| 能力外延 | Chain-of-Thought Prompting Elicits Reasoning in Large Language Models (arXiv 2022) | 在提示里给出“思考过程”示例，可显著提升多步推理题的正确率。 | [note][n_cot] | [source][s_cot] |
| 能力外延 | ReAct: Synergizing Reasoning and Acting in Language Models (ICLR 2023) | 将推理轨迹与可执行行动结合，降低无依据生成，在复杂任务中提升稳健性与可追溯性。 | [note][n_react] | [source][s_react] |
| 能力外延 | DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning (arXiv 2025) | 用可自动判分任务做强化学习奖励，逼出推理能力，减少昂贵人工链路。 | [note][n_deepseek_r1] | [source][s_deepseek_r1] |
| 能力外延 | Scaling Monosemanticity: Extracting Interpretable Features from Claude 3 Sonnet (Transformer Circuits, 2024) | 用稀疏字典把中间激活拆成可命名特征，并可调控特征稳定改变输出。 | [note][n_monosemanticity] | [source][s_monosemanticity] |
| 能力外延 | The Platonic Representation Hypothesis (arXiv 2024) | 模型越强表示越对齐，跨模态/架构收敛到同一“世界统计”表征。 | [note][n_platonic] | [source][s_platonic] |

<!-- Note links -->
[n_aiayn]: notes/20260214T133121--paper-attention-is-all-you-need__read.org
[n_bert]: notes/20260214T172116--paper-bert-pre-training-deep-bidirectional-transformers__read.org
[n_gpt3]: notes/20260214T195154--paper-language-models-few-shot-learners__read.org
[n_gpt2]: notes/20260215T120935--paper-language-models-unsupervised-multitask__read.org
[n_kaplan]: notes/20260215T145719--paper-scaling-laws-neural-language-models__read.org
[n_chinchilla]: notes/20260215T153458--paper-training-compute-optimal-large-language-models__read.org
[n_llama]: notes/20260215T185459--paper-llama-open-efficient-foundation-language__read.org
[n_rope]: notes/20260215T191140--paper-roformer-rotary-position-embedding__read.org
[n_flashattn]: notes/20260215T202729--paper-flashattention-io-aware-exact-attention__read.org
[n_rag]: notes/20260215T204618--paper-retrieval-augmented-generation-knowledge-intensive__read.org
[n_instructgpt]: notes/20260216T143500--paper-training-language-models-follow-instructions__read.org
[n_dpo]: notes/20260216T164916--paper-direct-preference-optimization-reward-model__read.org
[n_cot]: notes/20260218T145507--paper-chain-of-thought-prompting-elicits-reasoning__read.org
[n_react]: notes/20260218T163256--paper-react-synergizing-reasoning-acting__read.org.org
[n_deepseek_r1]: notes/20260218T164201--paper-deepseek-r1-incentivizing-reasoning-capability-llms__read.org
[n_qwen3]: notes/20260218T190603--paper-qwen3-technical-report__read.org
[n_moe]: notes/20260220T134823--paper-sparsely-gated-mixture-of-experts__read.org
[n_switch]: notes/20260220T141812--paper-switch-transformers-trillion-sparsity__read.org
[n_mixtral]: notes/20260220T205018--paper-mixtral-of-experts__read.org
[n_sparse_upcycling]: notes/20260220T210836--paper-sparse-upcycling-moe-dense-checkpoints__read.org
[n_platonic]: notes/20260221T135812--paper-platonic-representation-hypothesis__read.org
[n_textbooks]: notes/20260221T165951--paper-textbooks-all-you-need__read.org
[n_monosemanticity]: notes/20260221T173528--paper-scaling-monosemanticity-interpretable-features-sonnet__read.org
[n_palm]: notes/20260221T180518--paper-palm-scaling-language-modeling-pathways__read.org
[n_glam]: notes/20260221T194542--paper-glam-efficient-scaling-moe__read.org
[n_smol]: notes/20260221T195840--paper-smol-training-playbook-world-class-llms__read.org

<!-- Source links -->
[s_aiayn]: https://arxiv.org/html/1706.03762v7#S5
[s_bert]: https://ar5iv.labs.arxiv.org/html/1810.04805
[s_gpt3]: https://ar5iv.labs.arxiv.org/html/2005.14165
[s_gpt2]: https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf
[s_kaplan]: https://ar5iv.labs.arxiv.org/html/2001.08361
[s_chinchilla]: https://ar5iv.labs.arxiv.org/html/2203.15556
[s_llama]: https://ar5iv.labs.arxiv.org/html/2302.13971
[s_rope]: https://ar5iv.labs.arxiv.org/html/2104.09864
[s_flashattn]: https://ar5iv.labs.arxiv.org/html/2205.14135
[s_rag]: https://ar5iv.labs.arxiv.org/html/2005.11401
[s_instructgpt]: https://ar5iv.labs.arxiv.org/html/2203.02155
[s_dpo]: https://arxiv.org/html/2305.18290v3
[s_cot]: https://ar5iv.labs.arxiv.org/html/2201.11903
[s_react]: https://arxiv.org/pdf/2210.03629
[s_deepseek_r1]: https://ar5iv.labs.arxiv.org/html/2501.12948
[s_qwen3]: https://arxiv.org/abs/2505.09388
[s_moe]: https://ar5iv.labs.arxiv.org/html/1701.06538
[s_switch]: https://ar5iv.labs.arxiv.org/html/2101.03961
[s_mixtral]: https://ar5iv.labs.arxiv.org/html/2401.04088
[s_sparse_upcycling]: https://ar5iv.labs.arxiv.org/html/2212.05055
[s_platonic]: https://arxiv.org/html/2405.07987
[s_textbooks]: https://ar5iv.labs.arxiv.org/html/2306.11644
[s_monosemanticity]: https://transformer-circuits.pub/2024/scaling-monosemanticity/
[s_palm]: https://ar5iv.labs.arxiv.org/html/2204.02311
[s_glam]: https://ar5iv.labs.arxiv.org/html/2112.06905
[s_smol]: https://huggingface.co/spaces/HuggingFaceTB/smol-training-playbook
