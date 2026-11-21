# RSI 深读笔记 A 线：递归自我改进本体（5 篇全文 claim-evidence）

- 日期：2026-11-19
- 执行者：deep-papers-a（subagent of lead）
- 对照对象：rsi-agent（纯 Python 双层：inner 改策略 + outer 密封确认闸；四机制 SP=密封私集 / NC=非循环评分 / ESS=自相关感知 ESS 配对闸 / HA=哈希链审计）
- 范围：A 线「递归自我改进本体」5 篇；全部基于arxiv HTML **全文**（非仅 abs 摘要），已实测 HTTP 200 且号-题一致。
- **证据边界（贯穿全文）**：本笔记所有数字均为**论文正文自报，本实验室未复算**（我们没有这些论文的实验环境；本机零可用 GPU，见 AGENTS.md §17）。凡标「Agent 分析」者为我方推断，非作者陈述。凡标「正文明确」者可回到原句/图表核验。
- 方法纪律：claim-evidence 矩阵 + 「作者陈述 vs Agent 分析」分离 + 诚实缺口（取自 nature-paper-card 纪律，不生搬其 16 节模板）。沿 `notes/2026-11-19-rsi-papers.md` 骨架（实测表 → 逐条判断 → 证据等级 → 缺口）。

## 0. 五篇一览与最硬结论（先给结论）

| # | arXiv | 一句话最硬结论 | 对 rsi-agent 四机制的启示强度 |
|---|-------|----------------|------------------------------|
| 1 | [2611.23138](https://arxiv.org/abs/2611.23138) | decision 级质量标注+mask 让「训练模型去训练模型」在 PostTrainBench 36.56→48.24，比同 token 预算的 step-level 高 5.03 点：局部决策质量比最终成功更值得监督 | 间接（NC） |
| 2 | [2611.21056](https://arxiv.org/abs/2611.21056) | 编码 agent 自改进全程不看下游任务分（0 candidate eval），靠机制整合+确定性有效性闸也达 60.0→85.6%，transfer 更强、快 8.81×；去掉闸崩塌到 52.0% | **直接**（NC / SP） |
| 3 | [2611.08946](https://arxiv.org/abs/2611.08946) | RSI 增益主要在 harness 层（7 对配对 6 对提升）；dev/val/test 三分离 + test 只用一次 + δ=2σ 接受闸 + isolation/revert 是干净范式 | **直接**（SP / ESS） |
| 4 | [2611.10719](https://arxiv.org/abs/2611.10719) | 前沿预训练闭不了端到端 RSI 环，但拆成 debugging/headroom/experience-to-metric 三类小环 + 共享 context infrastructure 保存 negative result，仍能在唯一一次 run 内自改进 | 间接（HA / SP） |
| 5 | [2611.13524](https://arxiv.org/abs/2611.13524) | RSI 自改进过程的 token「税」（verification 占 74.8–93.6%）常超过任务执行成本；简单 bounded-note 基线匹配准确率而总 token 省 65.6–71.0% | **直接**（HA / ESS） |

**有直接可执行启示的 3 篇**：#2 ELITE（NC/SP：确定性有效性闸替代下游评分、迁移集当密封验证）、#3 Flex-RSI（SP/ESS：三分离密封集、2σ 噪声接受闸、配对 run、越界 revert）、#5 RSI Tax（HA/ESS：把每次更新的成本与「后续是否被用」记账审计）。#1 对 NC 提供「最终成功≠局部决策质量」的判据，#4 对 HA 提供「negative result 也要入账」的组织记忆范式。

---

## 1. RSI-Prime: Training Models to Train Models

- 外链：https://arxiv.org/abs/2611.23138 ；全文 https://arxiv.org/html/2611.23138
- 本地全文提取：`/tmp/rsiA/2611.23138.txt`（315 行）

### 1.1 可核对的核心 claim 列表

1. **【正文明确】** 提出 RSI-Prime 数据集，包含 350 条完整模型训练实验轨迹，覆盖 7 类任务，用于训练"会训练模型的模型"。位置：§3 数据集构建 / Abstract；支撑：Table 1（数据集统计）。
2. **【正文明确】** 轨迹过滤用 PostTrainBench 的四个 trajectory judge，并要求相对提升 `g=(s_post−s_base)/(s_instruct−s_base)>0.5` 才纳入。位置：§3.3 式(1)；支撑：式(1)。
3. **【正文明确】** "最终训练成功不等于每个局部决策质量高"——用 GLM-5.3 对每条 decision 的 Analysis/Choice/Execution 三维独立评估+多数投票。位置：§3.5；支撑：Appendix A.4 人类一致性（κ=0.831/0.799/0.866）。
4. **【正文明确】** decision 级 mask：仅当某 message 关联的全部 decision 都判 reasonable/uncertain 才纳入监督，否则整条 mask；保留 75.48% tokens 却优于保留 89.15% 的 tool-failure masking。位置：§3.6 式(2) z_a；支撑：Table 4、Figure 4。
5. **【正文明确】** Decision SFT 把 PostTrainBench Overall 从 36.56 提到 48.24（+31.95%），Full SFT 只到 45.61。位置：§4 主结果；支撑：Table 3。
6. **【正文明确】** Decision 相对 step-level（token 预算相近：75.48% vs 76.63%）高 +5.03 点。位置：§4；支撑：Table 4。
7. **【正文明确】** 行为层面 Decision 的 training protocol 通过率 96.11%，高于 Full 87.25%/step 90.16%/tool-failure 91.65%。位置：§4 Figure 4 分析；支撑：Figure 4。
8. **【Agent 推断】** 该工作把"自我改进的监督粒度"从 outcome 下移到 decision，本质是一种非循环评分（NC）：不靠最终分数给局部反馈，而靠独立 judge 对中间决策打分——与 rsi-agent NC 机制同构，但用 learned judge 而非确定性门。

### 1.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 350 trajectories / 7 categories / 385 segments / 90,738 decision records | §3, Table 1 | 正文自报，本实验室未复算 |
| Evaluation and diagnosis 38.43%、data preparation 27.09% | §3 | 正文自报，本实验室未复算 |
| 54.95% records 跨多 message；57.01% message 含多 decision | §3 | 说明 decision 级标注的必要性；正文自报，本实验室未复算 |
| 质量 92.42% high / 7.58% problematic；implementation optimization 16.89%、data source selection 12.80% problematic | §3.5 | 正文自报，本实验室未复算 |
| mask 保留 76.25% messages / 75.48% tokens（68,027/89,211 msgs；33,477,528/44,353,402 tokens） | §3.6 | 正文自报，本实验室未复算 |
| Overall 36.56→45.61(Full,+24.75%)→48.24(Decision,+31.95%)；Decision vs Full +2.63 | Table 3 | 正文自报，本实验室未复算 |
| AIME2025 +12.23 / HealthBench +26.63 / Arena Hard Writing +6.45 / HumanEval +4.88（Decision vs Full） | Table 3 | 正文自报，本实验室未复算 |
| 距 SOTA Fable5.1+Claude Code(53.68) 低 5.44 点 | Table 3 | 正文自报，本实验室未复算 |
| 监督策略：Full 45.61 / Tool-failure mask 45.12(89.15% tok) / Step-level 43.21(76.63% tok) / Decision 48.24(75.48% tok) | Table 4 | 正文自报，本实验室未复算 |
| pass rate：Decision 96.11% / Full 87.25% / step 90.16% / tool-failure 91.65%；排除 evaluation 类后 95.63% vs 87.40% | Figure 4 | 正文自报，本实验室未复算 |
| PostTrainBench v1.2 权重：AIME2025 24.48% / Arena Hard Writing 9.76% / GPQA Main 24.27% / GSM8K 10.11% / HealthBench 19.90% / HumanEval 11.47% | §4 | 正文自报，本实验室未复算 |
| 训练：GLM-5.3 SFT 2 epochs, batch 32, lr 1e-4, cosine, warmup 0.05；target Qwen3-4B-Base, 1×H100, 10h | §4 | 正文自报，本实验室未复算 |

### 1.3 方法机制可迁移点（对照 rsi-agent 四机制）

- 更新目标：**权重 θ**（训练 agent 自身模型参数，使其更会训练目标模型；不是运行时自改 harness）。【Agent 分析】rsi-agent 不训练权重，故此点**不适用**于直接移植；但"决策级监督"思想可借用。
- 反馈信号：trajectory filter 用 PostTrainBench judge + 相对提升阈值；decision 质量用 GLM-5.3 三维独立评估+多数投票，强调最终成功≠局部质量。
- 接受闸：decision 三维全 reasonable/uncertain 才 pass；message 级其关联全部 decision pass 才纳入，否则整条 mask（式2）。
- held-out / 密封集：PostTrainBench 作独立评估（target 从 Qwen3-4B-Base 初始化），但**没有明确的 agent 训练/评估分离密封集**；decision 质量用 source evidence 而非 final score。
- 四机制对照（均为 Agent 分析）：
  - **SP 密封私集**：正文无"agent 看不到的密封私集"设计；评估集 PostTrainBench 是公开的。**不适用**（该工作不是 harness 自改进循环）。
  - **NC 非循环评分**：**可借鉴**——"最终成功与局部决策质量解耦、用独立 judge 评中间决策"正是 NC 的判据来源；但作者用 learned judge（GLM-5.3），rsi-agent 用确定性非循环评分，可借鉴粒度思想而不借鉴 judge 本身。
  - **ESS 自相关感知配对闸**：**不适用**（无自相关/ESS 概念）。
  - **HA 哈希链审计**：**不适用**（无审计链）。

### 1.4 局限与作者自陈未做的事

- 作者自陈（§4）：**"The training corpus contains HumanEval-related content, resulting in training–evaluation content overlap for this task."**（训练-评估内容污染）。
- Appendix A.4：人审 400 records / 2 annotators + 第 3 人裁决，acceptance 395/398/390，κ=0.831/0.799/0.866；明确 **"does not estimate the recall of decision identification"**（只测精度不测召回）。
- §6 Conclusion 极短，**无专门 Limitations 节**（诚实缺口）。

### 1.5 诚实缺口

- 无专门 Limitations 节；训练-评估污染是正文里散落自陈的。
- 人审只估 acceptance/精度，不估 decision 识别的召回——意味着"被 mask 掉的 decision 里有多少其实质量高"未知。
- HTML 提取以文本为主，Figure 4 的具体柱状数值是从正文叙述读到，未逐柱核对图像像素。
- 权重级 SFT（1×H100, 10h）本实验室未复现，亦无复现计划（零 GPU）。

---

## 2. Evaluation-Free Recursive Self-Improvement for Coding Agents（ELITE）

- 外链：https://arxiv.org/abs/2611.21056 ；全文 https://arxiv.org/html/2611.21056
- 本地全文提取：`/tmp/rsiA/2611.21056.txt`（574 行）

### 2.1 可核对的核心 claim 列表

1. **【正文明确】** 提出 Explore-then-Integrate（ELITE）：从同一 parent 并行生成多个聚焦改进 hypothesis，各自独立实现，再检查各 child 的机制（互补/冗余/冲突/过复杂/不完整），把兼容机制重实现为一个 coherent descendant。位置：§1 Intro / Abstract；支撑：Figure 1。
2. **【正文明确】** 演化循环内**不使用任何下游任务结果或任务求解 trajectory** 来引导；只用确定性的结构/操作有效性检查（G_exp 查可编译/导入/断言/接口/修改范围，G_acc 加 tool-call 协议与主流控 smoke tests）。位置：§3.1 / Abstract；支撑：式(5)、Table 4（w/o gates 消融）。
3. **【正文明确】** 在 60-task SWE-bench Pro 上 60.0%→85.6%，DGM 86.7%/HGM 88.3%，但 ELITE 用 0 candidate task evaluations，wall-clock 快 8.81×/3.20×。位置：§4 Table 1 / Table 2；支撑：Table 1、Table 2。
4. **【正文明确】** 无进一步适配的跨基准迁移：Verified 73.3%、Polyglot 69.3%，高于 DGM/HGM（后者在迁移集上甚至退步）。位置：§4.2；支撑：Table 1、Figure 2。
5. **【正文明确】** 去掉两道确定性闸会崩塌：Full 76.1% → w/o deterministic gates 52.0%。位置：§4.2 / Table 4；支撑：Table 4。
6. **【正文明确】** Evaluation-based Select-one 在目标集 Pro 更高（91.7%）但迁移更差（Verified 70.0/Polyglot 62.0），作者据此论证机制整合是对"评估选择"的竞争性替代。位置：§4.2（L216）；支撑：Table 4（注意该基线单次 run）。
7. **【正文明确】** 探索宽度 K 有边际递减：K=7 平均 76.1 最优，K=9 Pro 最高 88.9 但迁移回落。位置：§4；支撑：Table 5。
8. **【Agent 推断】** ELITE 的"确定性有效性闸"与 rsi-agent 的 NC 机制同源（都不用被改进者自己的下游评分做接受判断）；"迁移集从不参与演化"等价于把 Verified/Polyglot 当密封验证集（SP）。

### 2.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| DeepSeek-V4-Flash：Initial 60.0/64.0/64.0(Avg 62.7)；DGM 86.7/63.3/62.7(70.9)；HGM 88.3/65.3/64.7(72.8)；Elite 85.6/73.3/69.3(Avg 76.1,+13.4) | Table 1 | Pro/Verified/Polyglot；正文自报，本实验室未复算 |
| Qwen-3.7-Plus：Initial 33.3/48.0/50.0；DGM 50.0/54.0/50.0；HGM 48.3/54.0/58.0；Elite 45.6/56.7/61.3(Avg 54.5,+10.7) | Table 1 | 正文自报，本实验室未复算 |
| 资源 DeepSeek：DGM 810 task-eval/0.58 self-mod·h⁻¹/2900M tok/43.15h/$45；HGM 800/3.77/2100M/15.66h/$30；Elite 0/16.35/80M/4.90h/$8 | Table 2 | 正文自报，本实验室未复算 |
| 资源 Qwen：DGM 820/0.58/2500M/48.33h/$125；HGM 800/2.99/1800M/19.70h/$90；Elite 0/10.89/70M/7.35h/$20 | Table 2 | 正文自报，本实验室未复算 |
| mini-swe-agent+Claude-opus-5.5：10 rounds / 6 accepted；Pro 71.7→73.3(c1)→80.0(c3)→83.3(c6 final)，Verified 70→78，Polyglot 72→78 | Table 3 / Figure 3 | 正文自报，本实验室未复算 |
| 消融：Full 76.1；Select-one(LLM) 71.8；SIFT-style 67.7；Evaluation-based Select-one 74.6(Pro 91.7)；Single K=1 69.9；Direct joint 72.1；w/o hypothesis 71.0；w/o deterministic gates 52.0 | Table 4 | 除 Full 与 no-gates 为 3 runs，其余单次；正文自报，本实验室未复算 |
| 宽度 K∈{3,5,7,9}：Pro 82.8/85.0/85.6/88.9；Verified 67.3/70.7/73.3/70.0；Polyglot 61.3/68.7/69.3/68.0；Avg 70.5/74.8/76.1/75.6 | Table 5 | 正文自报，本实验室未复算 |
| 3 runs Mean±SD：Pro 85.6±1.0 / Verified 73.3±1.2 / Polyglot 69.3±2.3（runs: 86.7/72/68；85/74/68；85/74/72） | Table 11 | 正文自报，本实验室未复算 |
| Figure 3：7 children 六个 76.7–83.3%/一个 3.3%；integrated descendant 81.7%（胜 5/7 但低于最高 83.3%）；T=10 有 5 次 accepted；8/10 代 integrated descendant 被 G_acc 拒；integration 用了 41 eligible proposals 中 25 个的 primary mechanisms | Figure 3 | 正文自报，本实验室未复算 |

### 2.3 方法机制可迁移点（对照 rsi-agent 四机制）

- 更新目标：**harness/scaffold Σ（源码仓库），LLM θ 固定**。位置：§3.1 𝓐=(θ,Σ)，𝓐_t=(θ,Σ_t)。【Agent 分析】与 rsi-agent 的 inner 改策略同层（非权重）。
- 反馈信号：**implementation-level evidence**（同父探索出的可执行 children 的机制），明确不用下游任务结果；G_exp 查可编译/导入/断言/接口/修改范围，G_acc 再加 tool-call 协议与主流控 smoke tests。
- 接受闸：两道确定性 gate；G_acc 失败保留 parent（式 5）；integrator 按 5 维（complementary/redundant/conflicting/overly complex/incomplete）组合机制，把决策单元从"整个 child"下移到"机制"。
- held-out / 密封集：演化只用 Pro 60-task subset 且 ELITE 不看其分；Verified/Polyglot 纯 transfer 评估、从不参与演化（§5.1, Figure 2）——相当于把迁移基准当密封验证。
- 四机制对照（均为 Agent 分析）：
  - **SP 密封私集**：**可借鉴**——Verified/Polyglot 全程不进演化循环、只在终点评估，正是"密封集不参与决策"的正面范例；rsi-agent 的 SP 私集可照此固化"演化期零访问、终点一次性评估"。
  - **NC 非循环评分**：**可借鉴（强）**——0 candidate task evaluation + 确定性结构/操作有效性闸，是 NC 的极致实现：接受判断完全不来自被改进 agent 的同一任务分布评分。rsi-agent 的 NC 可借鉴"用结构性有效性而非任务分做第一道门"。
  - **ESS 自相关感知配对闸**：**不适用**（ELITE 无自相关/方差判据；K=7 多分支+整合是另一类多样性机制，非 ESS）。
  - **HA 哈希链审计**：**不适用**（无审计链；但 G_acc 拒绝 8/10 代可视为门控日志的雏形，可启发 HA 记录门判定）。

### 2.4 局限与作者自陈未做的事

- §6 Limitations：仅 coding agents + 选定 benchmark 子集；依赖 implementation-level reasoning 与 validity checks；accepted updates 之间下游性能会波动（"两次中间 accepted 后暂时下滑"），长期演化行为未研究；当前 single lineage，无 archive。
- A.7 Evaluation scope / Future work：DeepSeek 每方法 3 runs、Qwen baselines 与**多数 ablation 单次**；benchmark 用子集；future work = archive-based search（保留历史 agent、选 parent、多 lineage）。

### 2.5 诚实缺口

- Figure 3(b) 的 post-hoc 86.7% 与 Table 1 主线 85.6% 口径不一致（不同 run / post-hoc）；正文称 "post-hoc downstream outcomes measured after conclusion, never exposed during evolution"。
- 多数消融单次 run（正文明确），故 Table 4 中小差距不宜当显著结论。
- "metric-free 演化"并非真无反馈：G_exp/G_acc 本身是人的先验结构检查，作者未量化这些检查的"设计成本"。
- HTML 中 Figure 1/2/3 为图像，本文数值多来自图注与正文叙述，未逐图核对像素。

---

## 3. Flex-RSI: Benchmarking Recursive Self-Improvement of Frontier Agents

- 外链：https://arxiv.org/abs/2611.08946 ；全文 https://arxiv.org/html/2611.08946 ；leaderboard https://flex-rsi.github.io
- 本地全文提取：`/tmp/rsiA/2611.08946.txt`（1762 行）

### 3.1 可核对的核心 claim 列表

1. **【正文明确】** 提出 Flex-RSI benchmark：24 tasks / 6 areas（Action/Reasoning/Creation/Reconstruction/Perception/Infrastructure），18 models，固定预算下测 frontier agent 的递归自改进。位置：§1 / §4.1 / Table 1；支撑：Table 1。
2. **【正文明确】** harness H=(E,I)，模型权重固定：executor E 改 task solution 并用 dev split 实验；improver I 改 harness（E 的 prompts/tools/workflow + 自己的 instructions/lessons），是"产生下一 harness 的操作者，同时也是其参数"。位置：§3.2（L127）；支撑：式(1)(2)。
3. **【正文明确】** 三不相交 split：dev（带 label，agent 可用）、validation（withheld，决定保留哪个 candidate）、test（withheld，run 结束只用一次）；candidate 仅在 validation 超过 accepted solution 达 noise margin δ 才保留。位置：§3.1 / §3.3 / Appendix B.4；支撑：式(2)。
4. **【正文明确】** 177 runs 中 150 个（85%）提高 validation score，median +10.4 点；闭源 85 runs 中 94% 提高（median +14.9），开源 92 runs 中 76%（median +8.5）。位置：§4.3；支撑：Figure 3(a)(b)。
5. **【正文明确】** 改进主要发生在 harness 层，模型权重从不改变、多数任务不训练。位置：§6 Conclusion；支撑：全文设计 + Table 8。
6. **【正文明确】** 配对 run（除 improver 外全同）：7 对中 6 对自改进 harness 的 test 更高（+2.0 到 +40.0），最大增益恰在原 harness 几乎无进展处。位置：§5.4；支撑：Table 8。
7. **【正文明确】** 反馈粒度影响开源模型：只记 success/failure 的 4 个任务上，开源 median gain 掉到 1.6 vs 闭源 16.9；量表分任务上开源 9.9 vs 闭源 28.4。位置：§5.2；支撑：Table 7。
8. **【正文明确】** 严格物理稳定性/精确正确性检查任务上闭源-开源差距最大；有容差或 judge 评分任务上差距最小。位置：§5.3；支撑：Table 7。
9. **【正文明确】** 单任务 run median 49.7M tokens；相对 token 使用与 score 的 rank correlation = −0.05（多花钱不买高分）。位置：§4.5；支撑：Figure 5（正文叙述）。
10. **【Agent 推断】** 该 benchmark 是 rsi-agent SP 机制的教科书式实现：三层 split 中 validation 与 test 都对 agent 密封，且 test 一次性使用；δ=2σ 的 noise margin 是 ESS 思想（把评估噪声纳入接受判据）的一种固定阈值特例。

### 3.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 24 tasks / 6 areas / 18 models（11 闭源 + 7 开源权重） | §4.1, Table 1 | 正文自报，本实验室未复算 |
| 主榜：Claude Opus 5.5 = 91.6（唯一跑全 20 任务，11 任务 test 第一）；Gemini4 Argon 89.0（仅 9 任务）；GPT-6.1 Sol 77.0；GPT-6 Astra 65.5；最佳开源 Qwen3.8-Flash-Next 49.1 | §4.2 | rank 口径=每任务击败其他模型占比；正文自报，本实验室未复算 |
| 177 runs 中 150（85%）提高 val，median +10.4；闭源 94%/median +14.9，开源 76%/median +8.5 | §4.3, Figure 3 | 正文自报，本实验室未复算 |
| 单模型 mean gain：Opus 26.7 / Argon 25.7 → Gemma-4-31B-it 6.8 | §4.3 | 正文自报，本实验室未复算 |
| 90% gain 到达轮次：Perception/Reasoning 第 3 轮、Creation 第 4 轮、Action 第 6 轮、Reconstruction 第 8 轮 | §4.4 | 正文自报，本实验室未复算 |
| 单任务 run median 49.7M tokens（中间 80%：8.6–93.4M）；token 使用 vs score rank corr = −0.05 | §4.5 | 正文自报，本实验室未复算 |
| starter 强/弱对比：单答案任务 starter median 78.2、最大 gain median 14.8、结束差 25.0；自建 solution 量表任务 starter 19.4、最大 gain 38.8、起点差 0.9→结束差 38.8 | §5.1, Table 7 | 正文自报，本实验室未复算 |
| 反馈粒度：9 量表任务开源 median gain 9.9 vs 闭源 28.4；4 成败任务开源 1.6 vs 闭源 16.9 | §5.2, Table 7 | 正文自报，本实验室未复算 |
| 严格检查差距：RoboCasa 50.0 / RoboLab 40.3 / RoboDojo 37.3 / ARC-AGI-2 49.2 / ARC-AGI-3 43.3 / Scene from Video 36.4 / Scene from Photo 26.5；宽松任务 Video Quality 0.1 / Mass 0.8 / Multi-turn 1.0 / Video Generation 2.5 / Image Editing 4.2 | §5.3, Table 7 | 数字为闭源-开源最佳差距；正文自报，本实验室未复算 |
| Scene from Photo：开源 Appearance 81.8 但 Objects ≤16.5 vs 闭源 62.5；Scene from Video 开源 Physics=0.0（四模型全）vs Opus 70.7 / Astra 37.7 | §5.3 | 正文自报，本实验室未复算 |
| 配对 run：7 对中 6 对 harness 自改进 test 更高（+2.0 到 +40.0）；DeepSeek Open Drawer 0.0→40.0；Qwen Turn On Sink Faucet +20.0、Active Search +38.3；无 improver run 6/7 对更早结束 | §5.4, Table 8 | 正文自报，本实验室未复算 |
| 弱模型用强模型 harness：Open Drawer DeepSeek 从 Argon executor 起 20.0、Argon improver 继续编辑 60.0、自己从零 40.0、Argon 80.0；Active Search Qwen3.8-27B 从 Argon harness 49.2 vs 从零 69.0 | §5.4, Table 9 | 正文自报，本实验室未复算 |
| 接受闸：δ 初始 = starter validation score 标准差的 2×；improver 可改 δ；未测 σ 的任务设 0 | Appendix B.4 | 正文自报，本实验室未复算 |
| 预算：wall-clock/GPU hours/tokens/dollars/validation eval 数/round cap；连续 5 轮无 candidate 也无 harness 改动则停（LIBERO-MAX/PRO 例外） | Appendix B.5 | 正文自报，本实验室未复算 |

### 3.3 方法机制可迁移点（对照 rsi-agent 四机制）

- 更新目标：**双层**——solution（executor E 改）+ harness（improver I 改 prompts/tools/workflow/lessons/acceptance rule），权重固定。与 rsi-agent inner 改策略 + outer 确认闸同构。
- 反馈信号：dev split（带 label，agent 可见，用于实验）+ validation split（withheld，决定接受）+ test（withheld，终点一次）；improver 收 executor transcript/实验/notes/所有轮分数，每 4 轮自审哪些 harness 改动后跟 accepted vs stall。
- 接受闸：**held-out validation 分数超过 accepted solution 达 δ 才保留**（式 2）；δ 初始 2σ，improver 可改；不完整/越界改动 revert。
- held-out / 密封集：明确 dev/validation/test 三分离，test 只用一次；agent code 在 sandbox 执行、无权访问 validation/test。
- 四机制对照（均为 Agent 分析）：
  - **SP 密封私集**：**可借鉴（强，正面范式）**——三分离 + test 一次性 + agent 无访问权限 + 越界 revert（Appendix B.6）+ evaluator/data splits 不可改，直接可作为 rsi-agent SP 私集的工程清单。尤其"每 phase 后越界改动 revert 并记为 violation"可补强 rsi-agent 的私集隔离。
  - **NC 非循环评分**：**可借鉴**——candidate 只在 withheld validation 上评分、dev 分只供实验不作为接受依据，避免"用自己见过的数据给自己打分"的循环。rsi-agent NC 可借鉴 dev/接受判据分离。
  - **ESS 自相关感知配对闸**：**可借鉴（部分）**——δ=2σ 噪声裕度是"把评估噪声纳入接受判据"的固定阈值版，但未处理**自相关**（连续轮次共享样本造成的有效样本量膨胀）；rsi-agent 的 ESS 配对闸比它更严，可视为对 Flex-RSI 的升级。另 Table 8 的 paired run（除 improver 外全同）本身就是配对设计思想。
  - **HA 哈希链审计**：**不适用**（无哈希链）；但"记为 violation""所有 model calls/experiments/dev eval 都经 benchmark 路由、计费并记录"是可审计日志思想的近亲，可供 HA 借鉴记录格式。

### 3.4 局限与作者自陈未做的事

- §6 Conclusion 明确：改进主要在 harness 层，权重从不改变、多数任务不训练；部分任务的 solution 可能在 dev split 上训练/微调 helper models；**Flex-RSI 不测改自身参数的模型，weight-level self-improvement 留 future work**。
- 每模型每任务**只跑一次**（重复则报最新）——无重复统计。
- 主榜 20 tasks 要求 ≥6 models；4 个任务只有 2 models（Table 12），排名口径在不同任务上覆盖模型数不一。
- 预算里 "evaluator 在 val/test 上的 model calls 记账但不计费"——成本口径可能低估。

### 3.5 诚实缺口

- Table 8 配对 run 的 +2.0 到 +40.0 是 7 对的个别差值，未给显著性检验/方差；第 7 对（Opus Turn On Sink Faucet）无差异。
- Table 9 "强模型 harness 迁移到弱模型"在 4 任务中只有 2 个提升，且都未接近强模型——作者未深挖为何另 2 个失效。
- §4.2 中 ARC-AGI 数字口径混乱（"Opus 99.8?" 实为 Astra 在 ARC-AGI-1；ARC-AGI-2/3 分属不同模型），需回原文/leaderboard 核对，本笔记不据此下结论。
- 本实验室未复算任何 benchmark；Figure 5/3 数值来自正文叙述，未核对图像。

---

## 4. No Second Run: Does RSI Stop at Frontier Pre-training?

- 外链：https://arxiv.org/abs/2611.10719 ；全文 https://arxiv.org/html/2611.10719 ；开源 https://github.com/GAIR-NLP/801-30B
- 本地全文提取：`/tmp/rsiA/2611.10719.txt`（563 行）

### 4.1 可核对的核心 claim 列表

1. **【正文明确】** RSI 的三个前提（从已有/base/toy 模型起、每次试验 ≤GPU-hours、每次返回可验分）在前沿预训练三条全破：run 需数十万 GPU-hours、月计不可复；headroom 不能从单一分读出；要改进的是生产系统与组织（6 条数据 pipeline / >30 阶段 / 1053 数据子集 / 130+ infra 代码版本 / 106 超参字段）。位置：§1；支撑：全文。
2. **【正文明确】** 端到端 propose→run→measure→update 环无法闭合，故把预训练循环拆成三类可闭合小环：debugging loops（异常驱动，measure=异常是否消失）、headroom loop（修 measure，信号=post-training headroom）、experience-to-metric loops（为凭经验的步骤建显式 eval）。位置：§2 / §9 Conclusion；支撑：§3–§5。
3. **【正文明确】** 共享 context infrastructure 整合跨团队证据（per-layer loss / per-module grad norms / expert load-balance stats + 超参变更/数据混合/infra diffs/checkpoint evals），记录失败及原因；"递归 lives here"。位置：§2 / §9；支撑：§3 案例。
4. **【正文明确】** 五个 cross-stack 诊断（load balancing / optimizer config / data quality / weight decay / data loading）约一个月完成；22 benchmark mean 在 1T budget 从 47.78→51.96→56.83。位置：§1 / §3.3；支撑：Figure（正文叙述）。
5. **【正文明确】** internal metric 比下游 eval 更早预警：3T budget 处 layer15 MaxVio revised pilot >14 vs main ~5，下游 eval gain 同期递减。位置：§3.1；支撑：Figure（正文叙述）。
6. **【正文明确】** headroom loop 揭示 false stagnation（base 分平缓但 SFT 两 epoch +1.49/+2.39）与 ranking disagreement（23 checkpoint 配对，两 epoch ρ=0.796、一 epoch ρ=0.438）。位置：§4；支撑：Figure（正文叙述）。
7. **【正文明确】** 产物 801-30B-A3B：base AVG 73.13（七 base 中第二），同 SFT 下 AVG5=28.70（超 Qwen3.5 by 8.16）、AVG13=81.34（第二），轻量 SFT 后四 agentic benchmark AVG 58.43 最高。位置：§8.2–§8.3；支撑：Table 7/8/9。
8. **【正文明确】** "模型是 byproduct，account（过程记录）才是贡献"；"no second run, but the one we have can still learn from its own progress"。位置：§9 Conclusion。
9. **【Agent 推断】** "记录失败及原因的 context infrastructure"是 HA 审计链的组织级类比：两者都把 negative result 与改动绑定留存，供后续迭代避免重踩。

### 4.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 801-30B-A3B：30B 总/3B active MoE，11.8T tokens，3 个月，新团队 | §1/§7 | 正文自报，本实验室未复算 |
| 22 benchmark mean 1T budget：47.78→51.96→56.83；3T 处 main 59.94 vs revised pilot 56.48 | §1/§2 | 正文自报，本实验室未复算 |
| 五 cross-stack 诊断约一月完成（无 infra 通常数月） | §3.3 | 正文自报，本实验室未复算 |
| infra fix：LB 系数 1e-2→1e-4、lr 2.7e-3→5e-4、router 优化器 Muon→AdamW；3T 处 layer15 MaxVio revised>14 vs main~5 | §3.1 | 多因素同改，不隔离单一贡献；正文自报，本实验室未复算 |
| Tiny-SFT 6.33B token mix：Math/Science/Code 各~2.00B(31.6%)、instruction following 0.331B(5.2%) | §4 | 正文自报，本实验室未复算 |
| ranking disagreement：23 checkpoint 配对，Stage1-1 13 个两 epoch ρ=0.796、Stage1-2/3 10 个一 epoch ρ=0.438 | §4 | 正文自报，本实验室未复算 |
| agentic data：260K raw PR→220K executable tasks(>84%)→九 harness→~2M trajectories→1.1M filtered(>100B tokens,~91K/trajectory)→共 134B agentic tokens | §6 | 正文自报，本实验室未复算 |
| attention-balanced sampling：256 GPU/CP=4/64 DP/窗口 8 microbatch，2.26× 吞吐（1126→2543 TGS） | §7.1 | 正文自报，本实验室未复算 |
| optimizer allocation（第二 pilot）：Overall_Avg all-AdamW 45.60 / Muon-expert 47.89 / Muon-expert+attention 48.28；local vs global 48.52 vs 48.28 | §7.2 | 每配置单次 run；正文自报，本实验室未复算 |
| Table 7 base：801 AVG 73.13（Qwen3.5 76.67、Nemotron-3.5-Lightning 73.07）；GSM8K 92.80/HumanEval 78.81 最高；MMLU-Pro 68.38 | §8.2 | 正文自报，本实验室未复算 |
| Table 8 SFT：AVG5 28.70（Qwen3.5 20.54，+8.16；AA-LCR 61.00/AA-OmniScience 31.30/SimpleQA-verified 34.50）；AVG13 81.34（Qwen3.5 83.76），Math500 98.40/GSM8K 95.75/HumanEval 95.12 | §8.3 | 正文自报，本实验室未复算 |
| Table 9 agentic：KernelBench 77.60 最高、四 benchmark AVG 58.43 最高（vs 完全 post-trained，声明为 headroom reference 非 like-for-like） | §8.3.2 | 正文自报，本实验室未复算 |
| learning aptitude Figure 25：BFCL step55 801=31.32<Qwen40.04，斜率 8.62 vs3.33(~2.6×)，step3485 48.65 vs46.26；SWE-bench Pro step30 26.95 vs0，斜率 9.15 vs5.03，step1922 43.37 vs8.76 | §8.4 | 正文自报，本实验室未复算 |

### 4.3 方法机制可迁移点（对照 rsi-agent 四机制）

- 更新目标：**数据 pipeline / infra / recipe / 评估度量**（组织与生产系统），权重更新只在唯一一次预训练 run 内发生，不是 agent 可反复改的对象。【Agent 分析】与 rsi-agent 的 harness/策略层类比但尺度不同。
- 反馈信号：异常驱动（MaxVio、输出模式异常）+ headroom 度量（tiny-SFT / learning aptitude / max-over-protocols）+ 显式化后的经验度量（classifier/coverage/verified yield）。
- 接受闸：按环而定——debugging 看异常是否消失；headroom 看 post-training headroom；数据修复需人批准；**没有单一统一接受闸**。
- held-out / 密封集：无明确密封私集；靠 internal metric（per-layer loss、MaxVio）与下游 eval 的时序对比来判断，且作者承认数据与 recipe 一起改、涨幅归因于组合。
- 四机制对照（均为 Agent 分析）：
  - **SP 密封私集**：**不适用**（前沿预训练无"私集评分"概念；评估集公开且多协议）。
  - **NC 非循环评分**：**可借鉴（弱）**——"internal metric 比下游 eval 更早预警""measure 信号必须对齐真目标，否则自证错误/坍缩/frame lock-in"支持 NC 的动机；但作者未实现非循环评分闸。
  - **ESS 自相关感知配对闸**：**不适用**（无配对/方差判据；每配置单次 run）。
  - **HA 哈希链审计**：**可借鉴（概念层）**——context infrastructure 把 negative result + 原因 + infra/data 改动绑定留存，正是 HA「不可抵赖审计」的组织级原型；rsi-agent HA 可借鉴"每次门判定与失败原因都要入链"的记录粒度。

### 4.4 局限与作者自陈未做的事

- **无专门 Limitations 节**（诚实缺口）；散落自陈：Tiny-SFT **未多种子**，相邻 checkpoint 小差异需谨慎；多处"多因素同改、不隔离单一贡献"（LB+lr+optimizer 同改；数据与 recipe 同改）；每配置**单次 run**；base 与 post-SFT 用**不同 benchmark 集**，只比趋势不比绝对值。
- Table 9 对比的是完全 post-trained 模型，作者明确声明 "headroom reference rather than like-for-like comparison"。
- 两 pilot 用于暴露并消除问题，**不是**用于外推分数——作者明确限制结论范围。

### 4.5 诚实缺口

- "递归 lives in context infrastructure"是叙事性主张，没有量化"保存 negative result 具体避免了多少次重踩"的消融。
- 1T→11.8T 的涨幅无法归因单一修复（作者自陈），故"三类环导致提升"是相关性而非因果。
- 多个 Figure（MaxVio、learning aptitude）数值来自正文叙述，未核对图像。
- 本实验室零 GPU，11.8T 预训练与本实验室无关，未复算。

---

## 5. The RSI Tax: When Recursive Self-Improvement Is Not Worth the Cost

- 外链：https://arxiv.org/abs/2611.13524 ；全文 https://arxiv.org/html/2611.13524
- 本地全文提取：`/tmp/rsiA/2611.13524.txt`（374 行）

### 5.1 可核对的核心 claim 列表

1. **【正文明确】** 定义 RSI Tax = 自改进过程的 token 成本（含尝试更新但最终保留旧状态的 token），end-to-end cost = 更新 token + 任务执行 token；在持续任务流上先记录每个任务结果，再用其反馈更新。位置：§1 / Abstract / §3.1；支撑：Figure 1。
2. **【正文明确】** 六方法 × 两模型 × (105+89) = 2,328 scored method-task pairs；bounded-note baseline（SRN）四 setting 中三个准确率最高、第四个与 EvoSkill 并列，total token 每 setting 都低于 Meta-Harness/EvoSkill/RRSI。位置：§1 / §2.2 / §5.1；支撑：Table 2、Figure 2。
3. **【正文明确】** 相对 Meta-Harness/EvoSkill/RRSI，SRN 平均省 65.6% / 71.0% / 68.2% total tokens；相对 Static SRN +4.7pp 准确率而 total token 仅 +3.6%。位置：§5.1；支撑：Table 2、Figure 1(c)(d)。
4. **【正文明确】** verification 占 RSI Tax：Meta-Harness 74.8% / EvoSkill 78.1% / RRSI 93.6%（四 setting 聚合）。位置：§5.2；支撑：Table 12(a)、Figure 3(a)。
5. **【正文明确】** 大量 tax 关联到后续未用的 state/artifact：Meta-Harness 52.78% / EvoSkill 37.91% / ReasoningBank 63.57% / RRSI 23.70%；SRN 全部关联到后续使用的 note。位置：§5.2；支撑：Table 13(b)、Figure 3(b)。
6. **【正文明确】** 省略 RSI Tax 会翻转成本比较：DeepSeek terminal 上 Meta-Harness task-execution 184.13M < SRN 214.37M，但 578.45M tax 使 total 762.57M vs SRN 214.65M，且准确率更低（77.5% vs 78.7%）。位置：§5.1（L143）；支撑：Table 11。
7. **【正文明确】** SRN update calls 占其 total token <0.4%；note 限 2048 tokens，每 4 任务更新，有效非空 revision 直接采纳。位置：§4 / §5.1；支撑：Table 2/8、式(7)。
8. **【Agent 推断】** 该文把"自改进的开销"变成一等公民指标，本质上要求审计链记录每次更新的 token 与后续使用情况——与 rsi-agent HA 的目标一致，可据其 Verify/Explore/Write 三分法扩展 HA 的日志字段。

### 5.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 2,328 = 6×2×(105+89) scored method-task pairs | §2.2 | 正文自报，本实验室未复算 |
| 六方法：Static / ReasoningBank / EvoSkill / Meta-Harness / RRSI / SRN；两模型 DeepSeek V4.1 Flash / GPT-6 Luna | §1 | 正文自报，本实验室未复算 |
| Table 2 准确率&总 token（M，四 setting = SWE-Chain-Evo×{DS,Luna} + Terminal-Bench2.1×{DS,Luna}）：Static 43.8/487.92、38.1/179.69、69.7/185.55、66.3/78.86；SRN 48.6/492.56、41.0/157.07、78.7/214.65、68.5/87.08；Meta-Harness 45.7/976.61、40.0/495.81、77.5/762.57、65.2/317.99；EvoSkill 41.0/1166.51、40.0/612.39、71.9/898.72、68.5/358.85；RRSI 44.8/1092.94、39.0/574.07、74.2/605.75、61.8/454.30 | Table 2 | 正文自报，本实验室未复算 |
| Table 9 RSI tax（M）：ReasoningBank 4.78/5.96/4.53/7.39；Meta-Harness 109.06/177.35/314.16/329.15；EvoSkill 144.35/213.21/393.10/358.57；RRSI 125.79/213.02/223.23/346.68；SRN 0.04/0.25/0.13/0.36 | Table 9 | 正文自报，本实验室未复算 |
| 成分聚合（M,%）：Meta-Harness 1648.86 = Verify 1233.89(74.83%)/Explore 296.99(18.01%)/Write 91.74(5.56%)/Other 26.24；EvoSkill 2103.05 = 1643.17(78.13%)/455.29(21.65%)/4.59(0.22%)；RRSI 1770.50 = 1656.86(93.58%)/45.22/68.41；ReasoningBank 48.98 = Explore 24.61(50.25%)/Write 24.37(49.75%)；SRN 1.21 = Write 100% | Table 12(a) | 正文自报，本实验室未复算 |
| 后续未用份额：ReasoningBank 63.57% / Meta-Harness 52.78% / EvoSkill 37.91% / RRSI 23.70% / SRN 0% | Table 13(b) | 描述性归因，非因果；正文自报，本实验室未复算 |
| "recorded input 约占 RSI Tax 98.1%"（Meta-Harness/EvoSkill） | §5.2 | 正文自报，本实验室未复算 |
| EvoSkill 在 DeepSeek terminal 上 497.45M 用于刷新旧 frontier vs 182.26M 于 89 个到达任务 | §5.2 | 正文自报，本实验室未复算 |
| SRN note ≤2048 tokens；每 4 任务更新；support set DeepSeek/SWE 用最近 2 任务、其余 4 任务；轨迹摘录头 20,000 + 尾 10,000 字符 | §4 / A.2 / A.4 | 正文自报，本实验室未复算 |

### 5.3 方法机制可迁移点（对照 rsi-agent 四机制）

- 更新目标：memory/skills（ReasoningBank/EvoSkill）与 harness（Meta-Harness/RRSI/HarnessPro）；SRN 只改一条 ≤2048 token 的 note。固定权重。
- 反馈信号：二元 verification outcome + 标量结果；SRN 用官方标量结果选失败任务改 note；ReasoningBank 用内部 trajectory judge；重跑已完成任务的花销记入 RSI Tax 且不替代原始结果。
- 接受闸：Meta-Harness selector 需严格 accuracy 提升才换部署、平局保留 incumbent；EvoSkill frontier 随窗口滚动重评分；SRN 有效非空 revision 直接采纳、无效不变。
- held-out / 密封集：信息边界明确——可读公开指令/可见轨迹/标量结果，**不可读未来任务/参考解/隐藏测试内容**；chain 边界重置状态；每任务从规定 repo snapshot 起。
- 四机制对照（均为 Agent 分析）：
  - **SP 密封私集**：**可借鉴**——"不可读未来任务/参考解/隐藏测试"是明确的密封信息边界；但本文的 support task 是已完成任务的重评分（非独立私集），需注意与 SP 的差别。
  - **NC 非循环评分**：**可借鉴**——"重跑已完成任务的花销记入 tax 且不替代原始结果"防止用重评分覆盖原始客观结果，是 NC 的重要纪律；符合 rsi-agent NC 精神。
  - **ESS 自相关感知配对闸**：**不适用**（无自相关/方差判据）；但该文证明"反复 verification 是主要浪费源"，从成本侧支持 rsi-agent 用 ESS 配对闸减少不必要的验证轮次。
  - **HA 哈希链审计**：**可借鉴（强）**——Verify/Explore/Write 三分法 + "subsequent use"归因，可直接扩展 rsi-agent HA 的日志字段：每次门判定记 token 成本、是否采纳、更新产物是否被后续任务实际使用。

### 5.4 局限与作者自陈未做的事

- §6 Limitations 原文：只覆盖两个 backbone、七条 repository chain、一个 terminal benchmark 顺序；其他模型/workload/ordering/重复 runs 未测；成本归因是**描述性**的——"use in a later task does not establish an accuracy benefit, absence of use does not establish calls can be removed without changing outcomes"；需要 controlled interventions 才能量化削减每个成分的效果。
- §6 Choosing an update policy：本文比较固定 procedure 与已实现成本，**没有评估"预测成本并选择更新"的策略**。
- §6 Reducing the RSI Tax：是否削减成本同时保持性能"requires controlled evaluation"——作者明确留作 future work。
- A.3：Meta-Harness 配置比原方法的 population-based search 更窄（一次 native search iteration）；EvoSkill 用原始 proposer 但信息边界更严、rolling refresh 是流式实现改动——即比较的不是这些方法的"满血"版。

### 5.5 诚实缺口

- Table 13 的 "subsequent use" 是 token 份额归因，不是因果效应；作者反复声明，本笔记不越界解读。
- SWE-Chain-Evo 是完整 50 chains/493 steps 的**选定子集**（7 chains×15=105）；terminal benchmark 只有一种固定顺序——顺序敏感性未知。
- GPT-6 Luna 配置省略采样参数；两模型 tokenization 不同，"nominal limits 不代表相同有效容量"——成本跨模型比较需谨慎。
- "省 65.6–71.0%" 是四 setting 平均，不是每个 setting 都成立（具体见表 2）——引用时须带口径。

---

## 6. 跨篇综合：对 rsi-agent 四机制的可执行清单（Agent 分析，非作者陈述）

| 机制 | 最直接的论文依据 | 可执行启示 |
|------|------------------|------------|
| **SP 密封私集** | #3 Flex-RSI（dev/val/test 三分离，test 一次，越界 revert） + #2 ELITE（迁移集从不进演化） | 固化"演化期零访问私集、终点一次性评估、越界改动 revert 并记 violation"；把 Val/Test 与 agent 可见 dev 彻底隔离 |
| **NC 非循环评分** | #2 ELITE（0 candidate task eval + 确定性有效性闸） + #1 RSI-Prime（decision 级独立 judge） + #5（重评分不替代原始结果） | 接受判断不取自被改进者自己的同分布任务分；用结构性有效性/独立判据；把"局部决策质量"而非最终分作为监督粒度 |
| **ESS 自相关感知配对闸** | #3 Flex-RSI（δ=2σ 噪声裕度 + paired run） + #5（反复 verification 是主要浪费） | 把固定 2σ 升级为自相关感知 ESS 判据：连续轮次共享样本时抬高接受门槛；用配对设计验证 harness 贡献，避免多付验证税 |
| **HA 哈希链审计** | #5 RSI Tax（Verify/Explore/Write + subsequent use） + #4（context infrastructure 记 negative result 与原因） | 扩展审计字段：每次门判定的 token 成本、是否采纳、产物是否被后续任务使用、失败原因；把 negative result 与改动绑定入链 |

**一句话判据**：四篇（#2/#3/#5 直接，#1/#4 间接）共同印证 rsi-agent 的两层+四机制方向站得住；最该补的是 SP 的"越界 revert 日志"、ESS 的"自相关→抬高门槛"、HA 的"成本+后续使用入账"这三处。

## 7. 全局诚实缺口

- 五篇全部数字为正文自报，**本实验室未复算**（零 GPU、无对应实验环境）。
- 本次基于 arxiv HTML 全文文本提取；Figure 1–5/25 等图像未逐像素核对，个别数字取自正文叙述。若需严谨引用图表数值，应回到原始 PDF/图核对。
- #1 无 Limitations 节 + 训练-评估污染；#4 无 Limitations 节 + 单次 run/多因素同改；#2/#3/#5 有部分单次 run 或描述性归因，已在各节标注。
- 未做任何本地实验、未 git commit/push。
