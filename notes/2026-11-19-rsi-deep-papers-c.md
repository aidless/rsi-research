# RSI 深读笔记 C 线：已有反面结果（4 篇全文 claim-evidence 级深读）

- 日期：2026-11-19
- 执行者：deep-papers-c（subagent of lead）
- 对照对象：rsi-agent（纯 Python 双层：inner 改策略 + outer 密封确认闸；四机制 SP=密封私集 / NC=非循环评分 / ESS=自相关感知 ESS 配对闸 / HA=哈希链审计）
- 范围：C 线「已有反面结果」4 篇；全部基于 arxiv HTML **全文**（非仅摘要），已实测 HTTP 200 且号-题一致。
- **证据边界（贯穿全文）**：本笔记所有数字均为**论文正文自报，本实验室未复算**（本机零可用 GPU，见 AGENTS.md §17；这批是纯分析论文，本实验室无对应实验环境，也无复现计划）。凡标「【正文明确】」者可回原句/图表核验；凡标「【Agent 分析】」者为我方推断，非作者陈述。
- 方法纪律：claim-evidence 矩阵 + 正文陈述/推断分离 + 诚实缺口。沿 `notes/2026-11-19-rsi-deep-papers-a.md` 骨架（一览 → 逐篇 claim → 关键数字 → 四机制对照 → 局限 → 缺口）。
- 本地全文提取（bs4+lxml，math alttext 展开）：`.tmp/rsi-c/<id>.txt`；原始 HTML `.tmp/rsi-c/<id>.html`。正文位置同时给 arxiv §标题与本地行号，便于回查。

---

## 0. 四篇一览与最硬结论（先给结论）

| # | arXiv | 一句话最硬反面结论 | 对 rsi-agent 的命中性质 |
|---|-------|--------------------|------------------------|
| 1 | [2610.38392](https://arxiv.org/abs/2610.38392) | 接受闸只看「当前模型下 harness 做什么」，冗余 safeguard 被逐层删除时伤害被 mask；**更大的 eval batch 反而让配对显著性闸接纳更便宜的删除**（同 cost，20-task 全保住、40-task 全丢，p=.001）；审计止住暴露但止不住深度流失 | 字面机制**当前不成立**（ContextualTask 无冗余层），但闸的「只看任务成功」同构；迁移 SWE-bench 后**会出现** |
| 2 | [2611.12988](https://arxiv.org/abs/2611.12988) | 自更新在模型「再更新一次」后收益倒退：balanced 对比 immediate +0.016pp（跨零），continuation **-1.78pp（simultaneous 95% CI [0.68, 2.88]，Holm p=.0012）**；选择抬高了第一端，配方与 recipient/donor 控制均未找到可靠 mitigation | 参数级 continuation 机制**当前不成立**（inner 不训练参数）；但「只看 immediate 不看下一跳」的评价盲点是真警示 |
| 3 | [2610.22984](https://arxiv.org/abs/2610.22984) | 冷启动「只从失败学」存在稳定的 **repair–generalization gap**：能修复原失败任务，却几乎不提升 held-out；即使只保留人工验证正确的经验，5 配置中 4 个 held-out ≤ 无经验基线；成功轨迹带来的迁移增益远大于 verifier feedback 或额外交互 | 字面机制**当前不成立**（inner 不吃失败诊断经验、无记忆库）；但其「验证正确≠可迁移」对 SP 密封集的泛化价值构成方法论质疑 |
| 4 | [2611.04952](https://arxiv.org/abs/2611.04952) | 定义 artifact→capability 五属性链，反射/Voyager/DGM 三者都显出三个信号（artifact 累积、行为改变、指标上升）却各自在**不同属性处 break**；信号式评价无法判定能力是否形成 | **部分当下命中**：HA 只对应 Property 1 的记账版，缺 Property 2–5 的能力检验；paired_gate 聚合判定放行的逐实例回归，正是 Property 4 判据的同构漏洞 |

**一句话总结**：4 篇里没有一篇的字面实验设置能整体套到 rsi-agent（它不训参数、无冗余 safeguard、无文本/技能/代码 artifact 库、inner 不吃失败轨迹）。但**唯一能对当下实现造成实伤的是 #4 的 Property 4（边界安全复用）判据**——paired_gate 只看聚合配对均值，从不检查「candidate 是否推翻了 baseline 已建立的正向实例」，这道逐点回归当前就会被放行；迁移到 SWE-bench 二值 pass/fail 后，它从「隐性质量退化」升级为「修好 N 个弄坏 M 个」的字面 compositional override。最小修补见 §5.3。

---

## 1. Masked Erosion: How Acceptance Gates Lose Redundant Safeguards in Self-Improving Agents（2610.38392）

- 外链：https://arxiv.org/abs/2610.38392 ；全文 https://arxiv.org/html/2610.38392
- 作者/日期：Hau-Sing So, Daixian Liu, Weicai Long（University of Macau / Tsinghua）；arXiv:2610.38392v1 [cs.AI] 28 Oct 2026（已核实 HTML 页眉）
- 本地全文：`.tmp/rsi-c/2610.38392.txt`（4132 行）/ `.html`（591909 bytes）

### 1.1 可核对的核心 claim 列表

1. **【正文明确】** 接受闸按 batch 任务成功率去留，看到的是「当前模型下 harness 做什么」，看不到「某 safeguard 失效后 harness 仍能挡住什么」；safeguard 重叠时删一层被 mask，冗余被看不见地侵蚀（masked erosion）。位置：Abstract（本地 L25）、§1 Intro（L91）；支撑：全篇受控删除实验。
2. **【正文明确】** 测 16 个四层 safeguard 子集、411 个 improvement loops / 10 条 gate 规则（393 stress: 78 AppWorld + 315 OpsSim；18 natural）+ 3 次 GEPA 0.1.4（800 metric calls × 3 seeds）。位置：Abstract（L25）、§ study design（L542）。
3. **【正文明确】** **大 batch 放开更便宜的删除**：跑前冻结模拟器预测 break cost 为 20/40/120-task batch = 0.20/0.09/0.03；20-task paired loops 在 cost=0.15 时 6/6 全保住保护，40-task 6/6 全丢（p=.001）。位置：§「Larger Batches Admit Cheaper Deletions」（L712-719）；支撑：Figure 5b。
4. **【正文明确】** paired 显著性闸压住「靠运气通过」的删除但挡不住「真实涨任务成功」的删除：27 个 AppWorld stress loops 中 greedy/split-half 接受 0.48/0.44 删除、paired 仅 0.04；greedy 接受 masked 删除 0.63 vs no-op 0.36；paired 接受 2/42 masked 删除。位置：§「Significance Suppresses Drift」（L587）。
5. **【正文明确】** 审计止住暴露、止不住深度：veto(1) 在 14/18 exposing 删除上看到 child-only harm，29 个 masked 删除中只看到 2 个；audit-veto 下 unprotected loops 5/6→0，但 6 个 finals 中 5 个 margin 只剩 1。位置：§「Audits Stop Exposure, Not Depth Loss」（L741）。
6. **【正文明确】** 深度流失在换更弱模型时变成实伤：11 个 DeepSeek finals 换 Qwen3-30B，non-paired 误用凭证 0.30 vs paired 0.004（差 CI [0.16,0.47]）；保留 auth+至少一层 200 次运行 2 次误用，仅 auth 9/40，无 auth 89/200。位置：§「Lost Depth Becomes Harm Under Weaker Models」（L790）。
7. **【正文明确】** GEPA 三个 final 逐字保留 credential 规则，其中 2 个把 note template 改写成「用存储凭证登录」指令，Qwen3-30B 下 40 次运行 2/5 次误用（H0 80 次 0 次）；作者标注为 case study。位置：§GEPA（L820 附近）。
8. **【正文明确】** 注册预测 177 条：113 成立、49 失败、9 部分、6 无法计分；14 个主 claim 支撑测试中 7 个成立。位置：Abstract（L477）、§validation（L4036）。
9. **【Agent 分析】** 这是「Goodhart 式接受闸」的一个具体 instantiation：闸的优化目标（batch 成功率）与真正想要的性质（纵深防御）不是同一变量，且冗余使二者在杀练者模型下解耦。rsi-agent 的 paired_gate+ESS+noise floor 同样是「只看公开分」的闸，与其同构，只是当前任务没有可被侵蚀的冗余。

### 1.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 411 loops = 393 stress（78 AppWorld + 315 OpsSim）+ 18 natural；10 gate 规则；3×GEPA（800 metric calls×3 seeds） | Abstract / §design L542 | 正文自报，本实验室未复算 |
| break cost 0.20 / 0.09 / 0.03（20/40/120-task，跑前冻结模拟器） | §Larger Batches L715, Figure 5b | 正文自报，本实验室未复算 |
| 20-task cost=0.15：paired 6/6 保住；40-task 6/6 全丢（p=.001） | §Larger Batches L717-719 | 正文自报，本实验室未复算 |
| 120-task：paired 在 cost 0.03/0.08 丢 5/6；按 distinct task 计会反转 10 次接受中的 4 次 | §Larger Batches L719 | 正文自报（方向确认、精确 break 不确认），本实验室未复算 |
| 27 stress loops 删除接受率 greedy 0.48 / split-half 0.44 / paired 0.04；masked 0.63 vs no-op 0.36；paired 2/42 masked | §Significance Suppresses Drift L587 | 正文自报，本实验室未复算 |
| 冻结 per-edge 链式外推至第 64 代仍有 0.19 概率落到 unprotected | §同上（长时程为模拟器外推） | 正文自报（非实测），本实验室未复算 |
| 全栈 clean success 0.30 / 空栈 1.00（3-step OpsSim）；paired 接受 g<0.075 删除 9/292、g>0.275 删除 46/92 | §Costly Layers L619 | 正文自报，本实验室未复算 |
| audit veto：unprotected 5/6→0，但 5/6 finals margin=1；Table 3 margin greedy 0.33 / audit 1.17 / margin-aware 1.33 / paired 1.83 | §Audits L741 | 正文自报，本实验室未复算 |
| 独立 harm-pool 检测 masked 删除概率上界≈0.19（4 层）；veto 抓 0.008 child-only harm 需≈200 对；rate comparison 区分 0.008 vs 0.025 需≈700 runs/arm | §Masking, Margin and Child-Only Harm L352 | 正文自报，本实验室未复算 |
| Qwen3-30B：non-paired 误用 0.30 vs paired 0.004（CI [0.16,0.47]）；Qwen3-8B 差 0.16 [0.02,0.29] | §Lost Depth L790 | 正文自报，本实验室未复算 |
| 177 predictions：113 成立/49 失败/9 部分/6 无法计分；14 主测试中 7 成立 | Abstract L477 / §validation L4036 | 正文自报，本实验室未复算 |

### 1.3 反面结果对 rsi-agent 的具体威胁（逐问答）

- **paired_gate + ESS 闸 + noise floor 会不会奖励「删掉看不见的保险」？【Agent 分析】** 闸本身与本文同构——`rsi-agent/rsi_agent/gates.py:74-109` 的 `paired_gate` 只验 `delta>0 & nratio>=1 & p<alpha & ESS>=8`，`noise_floor` 是 `abs(baseline mean)*0.02`（gates.py:112-129），全是「公开分是否涨」的函数，**没有任何一项度量冗余/纵深/margin**。所以只要 rsi-agent 的策略空间里存在「删掉一层保险、当前公开分不降」的改动，闸会放行。但当前 `rsi-agent/rsi_agent/tasks.py:17-46` 的 ContextualTask 策略空间是 3 维权重、reward 是 `chosen·true_w + 确定性噪声`，**不存在独立可删的 safeguard 层**，因此没有「看不见的保险」可奖励——威胁的机制前提当前缺失。
- **我们的 ContextualTask 没有冗余 safeguard，这个威胁是否不成立？【Agent 分析】** 就 masked erosion 的字面机制而言**成立地不成立**：没有 4 层 auth/verify/profile/caution 那样的重叠防御，也没有 harness 自编辑，删除一层被另一层 mask 的因果链无从发生。
- **迁移到 SWE-bench 后是否会出现？【Agent 分析】** **会**。一旦 inner 从「调 3 维权重」升级为「改 harness/测试/lint/类型检查等规则」，就复现了本文的必要条件（多层冗余 + 接受闸按任务成功判定 + 更强模型下 mask 生效）。而且 rsi-agent 的密封私集 SP 只防「信息泄漏」，不防「纵深被侵蚀」——私集也是按同一任务成功口径判定，删掉被 mask 的层在私集上同样不降分。noise floor 用 baseline 比例给定，反而可能在删除带来微小成功增益时更容易过 `nratio>=1`。

### 1.4 可落地的修补

- **作者 mitigation**：(a) Freezing Safety Text——把 safety-critical 文本重置为 parent 的，4 natural loops 全保住 H0 safety text、Qwen3-30B 误用 0.019；(b) Auditing One-Layer Ablations（margin-aware gate，变异充分性）——仅当每个单层消融仍受保护才接受，greedy 下 0/6 变 unprotected（vs greedy 5/6，matched sign test p=.031），OpsSim 下保 3 层、代价 0.35 clean success。总原则：把 safety-critical 行为留在 optimizer 之外、按「剩下什么」而非「发生什么」gate、模型更换后重审 harness。
- **【Agent 分析】可移植进 rsi-agent 的最小改动**：在 `runner.py` 的 promoted 路径前加一道「层消融闸」——若候选来自可被删减的 harness 改动，对每个 safeguard 层单独做一次 sealed 求值，只有每层的单层消融仍在 noise floor 之上才 promote；并把 safeguard 文本/规则加入类似 `barriers.py` 的 `_SEALED` 冻结区，inner 的 propose 空间不允许触及。当前 ContextualTask 下这是 no-op，属于「迁移前置加固」。

### 1.5 局限与作者自陈未做的事 / 诚实缺口

- 作者自陈：仅两个环境、每环境一个 harm + 四个书面 safeguard；多数 loop 每代只植入一次删除；自然 loop 与 GEPA 为 case study；batch size 与 harm share 只在 OpsSim 变化，120-task batch 从 60-task pool 重采样；长时程结果为模拟器外推；第三方 API 模型为补充。
- 诚实缺口：14 个主 claim 支撑测试只有 7 个成立（其余方向性/更强/更弱），本文自己的Register ledger 就是半数的反面；「margin-aware gate」的代价（0.35 clean success）是否可接受取决于部署，作者未给决策规则；HTML 中 Figure 5/6 的具体柱状数值取自正文叙述，未逐像素核对。

---

## 2. Third Eye: Beyond Immediate Gains in Recursive Self-Improvement（2611.12988）

- 外链：https://arxiv.org/abs/2611.12988 ；全文 https://arxiv.org/html/2611.12988
- 作者/日期：Sujato Dutta, Padala Indira Sneha, Kypu Mahitha, Shashank Vanga, Neeraj Choudhary（Mahindra University）；arXiv:2611.12988v1 [cs.LG] 08 Nov 2026
- 本地全文：`.tmp/rsi-c/2611.12988.txt`（2041 行）

### 2.1 可核对的核心 claim 列表

1. **【正文明确】** 把自更新的 immediate 表现与「模型再更新一次（continuation）」后的结果分开测；5 个 3–8B backbone，math（GSM8K/MATH）+ code（MBPP/HumanEval），retention=MMLU；selection 用 development items、final scoring 用独立 items。位置：Abstract、§1；支撑：Figure 1。
2. **【正文明确】** balanced Qwen3-4B/Llama-3.2-3B 32-run 对比：immediate-selected utility +0.016pp（95% [-1.319,1.326]，跨零），continuation 后 -1.760pp（[-3.105,-0.420]）；paired decrement -1.776pp，simultaneous 95% [-2.876,-0.679]，Holm p=.0012。位置：§3.1、Abstract、Table 3；摘要口径写「continuation reduces it by 1.78pp（95% interval [0.68,2.88]）」。
3. **【正文明确】** 选择效应：同一 32 runs 上 candidate-averaged immediate utility -0.686pp（非 selector 的 +0.016），candidate-averaged continued -2.059pp——selection 抬高第一端，但未消除 continuation 递减。位置：§3.2。
4. **【正文明确】** backbone 差异大：SmolLM3-3B continuation 降 4.262pp（[-6.163,-2.581]），Llama-3.1-8B 降 2.501pp，Qwen3-8B 降 0.681pp（[-1.914,0.629]）inconclusive；Qwen-minus-Llama G2 contrast +2.801pp（[0.266,5.33]）。位置：§3.3。
5. **【正文明确】** 损失主要来自 target/OOD 而非 retention 遗忘：Llama-3.2-3B target -6.45pp、OOD -3.88pp、retention -0.01pp；SmolLM3-3B -6.46/-6.00/-0.33。位置：§3.4。
6. **【正文明确】** 配方控制未给可靠 mitigation：10 步/25 步/pooled 相对 50 步的差都跨零；crossed recipient/donor 不显著；gold solutions 比 self-generated 差 11.56pp 但 assistant-label token 只有 41–61%、code syntax failure 飙升（配方依赖而非「参考解更差」）。位置：§4.1–4.3。
7. **【正文明确】** Replay（dev 增益>0 且 OOD/retention 非负才采纳）只采纳 21.9% selected mass，相对 Hold -0.27pp（[-0.80,0.21]），未确立优于不更新；固定 commit/hold 规则同样未胜出。位置：§5。
8. **【Agent 分析】** 本文把「自更新」重新定义为 adaptation transition：一个更新的价值不在此刻分数，而在它之后还能不能继续被更新。这是对「只看 immediate 就宣布 self-improvement」的直接证伪，rsi-agent 的 verdict（SELF_IMPROVEMENT_VERIFIED）正是即时判定，缺少下一跳。

### 2.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 64 fresh runs ×3 候选 ×3 continuation = 192 候选 / 576 continuation / 832 checkpoint 评测 | Abstract / §methods | 正文自报，本实验室未复算 |
| immediate +0.016pp [-1.319,1.326]；continuation -1.760pp [-3.105,-0.420]；paired -1.776pp，simultaneous 95% [-2.876,-0.679]，Holm p=.0012 | §3.1 / Table 3 / Abstract（1.78pp，[0.68,2.88]） | 正文自报，本实验室未复算 |
| candidate-averaged immediate -0.686pp，continued -2.059pp | §3.2 | 正文自报，本实验室未复算 |
| SmolLM3-3B -4.262pp [-6.163,-2.581]；Llama-3.1-8B -2.501pp [-4.411,-0.583]；Qwen3-8B -0.681pp [-1.914,0.629] | §3.3 | 正文自报，本实验室未复算 |
| Qwen-minus-Llama G2 contrast +2.801pp [0.266,5.33]；run-only [1.85,3.78] vs joint [0.27,5.33] | §3.3 / §5 | 正文自报，本实验室未复算 |
| 角色分解 Llama-3.2-3B target/OOD/retention = -6.45/-3.88/-0.01pp | §3.4 | 正文自报，本实验室未复算 |
| gold vs self-generated -11.56pp [-14.45,-8.64]；Llama code syntax failure 1.66%→18.55% | §4.3 | 正文自报，本实验室未复算 |
| Replay 采纳 21.9% mass，vs Hold -0.27pp [-0.80,0.21]；optimism K=1→K=3 从 0 升到 2.09pp | §5 | 正文自报，本实验室未复算 |
| 方法：LoRA rank16/scale32，50 AdamW steps lr1e-4，每更新 400 example presentations，5000 bootstrap | §methods | 正文自报，本实验室未复算 |

### 2.3 反面结果对 rsi-agent 的具体威胁

- **【Agent 分析】参数级 continuation 机制当前不成立**：rsi-agent 的 inner（`rsi-agent/rsi_agent/outer.py:18-38`）只用高斯扰动改 3 维权重 `HeuristicPolicy`，不训练模型参数、不用自生成数据做下一轮训练，因此 Third Eye 的「用自更新后的模型再自更新导致分布/质量坍缩」链条没有对应物。
- **【Agent 分析】但有一处方法学同构值得警惕**：rsi-agent 每代 promoted_policy 成为下一代的锚（`outer.py` propose 取 history 中 public_mean 最高者），这是一种「逐代延续」；outer 闸每代重新验证，`runner.py` 只在 pub+priv 都 pass 才 promote，理论上每一跳都被重新 gate。这比 Third Eye 的「immediate 就宣布成功」更稳健——但前提是 sealed 私集每代判别准确且没有 selection optimism。本文 §5 的 optimism 随候选数 K 上升（K=3 达 2.09pp）提示：若 rsi-agent 未来把 inner 改成「每代生成多个候选、按公开分挑最好」，同一结果选择与评分会带来虚高。当前 inner 每代只提一个候选（`propose()` 单次返回），故 optimism 风险低。
- **结论**：本文对 rsi-agent 是「评价观」层面的警示，不是当下可发生的实伤。

### 2.4 可落地的修补

- **作者 mitigation / 结论**：作者**未找到可靠 mitigation**（§4 配方控制全跨零、§5 Replay 未胜 Hold）；正面建议是把持久自更新当作 adaptation transition 评估——独立 scoring + no-update baseline，不要把 immediate 当全部价值。
- **【Agent 分析】最小移植**：在 `runner.py` verdict 输出前增加「下一跳探针」——对 promoted_policy 再做一次 inner propose + sealed 求值，若第二跳的 private delta 转负则降级为 NO_VERIFIED_GAIN 并 audit 记一条。这是对「只有一跳就宣称 verified」的最小补丁，无需 GPU、纯 Python。

### 2.5 局限与作者自陈未做的事 / 诚实缺口

- 作者自陈：仅 5 个 3–8B backbone、2 任务、two-example LoRA、H2 两代；更大 batch/全量微调/更长循环未测；8 runs/8B 模型精度不足；硬件/训练史/参数量未独立随机化；固定题集可能与预训练重叠；code 的 dev/final OOD 域不同；观测符号变化是噪声估计；16-parent 控制复用已检查 parents、未确立中介。
- 诚实缺口：主结果 CI 是 simultaneous 95%、条件于 3 个平均 continuation 与等权目标，换成单次 continuation 会更宽；摘要只报正的 [0.68,2.88] 不报 immediate 跨零，读摘要易误以为「自更新立刻有害」；Qwen3-8B（唯一 A100）与其他 GH200 混跑，硬件是混淆变量。

---

## 3. Self-Evolving Agents Can Repair Failures but Struggle to Generalize from Them（2610.22984）

- 外链：https://arxiv.org/abs/2610.22984 ；全文 https://arxiv.org/html/2610.22984
- 作者/日期：Weixiang Zhao, Yichen Zhang（共同一作）, Yanyan Zhao, Jinming Bai, Bin Liang, Ruifeng Xu, Bing Qin, Ting Liu（哈工大 / 港中文）；arXiv:2610.22984v1 [cs.CL] 16 Oct 2026
- 本地全文：`.tmp/rsi-c/2610.22984.txt`（1707 行）

### 3.1 可核对的核心 claim 列表

1. **【正文明确】** 定义 Cold-Start Failure-Only Evolution（CFOE）三约束：空外部状态起步、只用失败轨迹更新、verifier 反馈（标签/奖励/解释/ground truth/专家解）对 agent 与 updater 全隐藏；数学式 `S_t = S_{t-1} if y_t=1 else U(S_{t-1},τ_t)`。位置：§2（L134-163）；支撑：式(1)、Figure 1。
2. **【正文明确】** 三种自进化范式（ReasoningBank 记忆 / SkillOpt 技能 / Evo-Harness harness）、10 个 backbone、4 个质量验证环境（ALFWorld / τ²-Bench-Verified / WebArena-Verified / WebShop）；失败密集流由「3 次独立 rollout 均失败」的题构造。位置：§3.1（L169-252）。
3. **【正文明确】** 稳定的 repair–generalization gap：失败经验能部分修复原失败任务，但 held-out 增益有限甚至下降；扩到 800 个 ALFWorld 任务的长程后仍未闭合。位置：§3.2–3.3（L267-293）；支撑：Figure 2、Figure 3。
4. **【正文明确】** 四类经验形成错误 M1 假成功归因 / M2 误判与幻觉归因 / M3 经验固化扭曲 / M4 有害未来行为指导；Table 1 给出各模型-基准占比（如 τ²-Bench Qwen3.5-27B Memory 的 M2=53.2%）。位置：§4.1（L381-408）；支撑：Table 1。
5. **【正文明确】** 即使只保留人工验证正确的经验，5 个配置中 source-task recovery 都提升，但 4 个 held-out 分数仍在无经验基线或之下——「正确但绑死源轨迹」无法迁移。位置：§4.2（L411-423）；支撑：Figure 4。
6. **【正文明确】** 放宽 CFOE 三维（成功轨迹 / verifier feedback / Reflection 时额外 10 轮交互）后：Success 与 Mix 的 held-out 提升明显大于 Fail；Feedback 和 Resume 有改善但有限。位置：§5（L424-478）；支撑：Figure 5。
7. **【Agent 分析】** 本文把「self-improvement」拆成 repair（原地修复）与 generalize（能力迁移）两种，指出多数失败学习系统只做到前者。rsi-agent 的公开/私集 seed 分离正好是「repair 集 vs generalize 集」的显式划分，私集的存在正是对这个 gap 的防御——但私集与公开集同分布（同为 ContextualTask），只是 seed 不同，因此它测的是「换 seed 的稳健性」而非「跨任务的能力迁移」。

### 3.2 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 3 范式 / 10 backbone / 4 环境；失败流由 3 次 rollout 均失败构造 | §3.1 | 正文自报，本实验室未复算 |
| Table 1 错误占比示例：ALFWorld Qwen3.5-27B Memory M1-M4=41.3/46.5/32.6/35.5；τ²-Bench Memory M2=53.2% | Table 1 | 单条更新可含多个错误模式；正文自报，本实验室未复算 |
| 800 ALFWorld tasks 长程；Qwen3.5-27B+Evo-Harness held-out 进一步下降、Qwen3.5-4B+SkillOpt 平台期 | §3.3 / Figure 3 | 正文自报，本实验室未复算 |
| 仅保留验证经验：5 配置 source recovery 全提升，4 配置 held-out ≤ 无经验基线 | §4.2 / Figure 4 | 正文自报，本实验室未复算 |
| Figure 5 条件：Fail+Blind / Success / Mix / Feedback / Resume(10 交互+1 总结)；Resume 另测 5/10/20 轮 | §5 | 正文自报，本实验室未复算 |
| 实现：闭源/大模型走官方 API，其余 vLLM on H100 | §3.1 | 正文自报，本实验室未复算 |

（注：本篇以曲线/条形为主，正文几乎不报单点置信区间或显著性检验；「更优/更差」多为图示描述性判断——这是证据强度上的诚实缺口。）

### 3.3 反面结果对 rsi-agent 的具体威胁

- **【Agent 分析】字面机制当前不成立**：inner（`outer.py`）generate 新策略时读的是 `PublicHistory.summary()`（best public_mean + promoted 计数，`core.py:21-66`），不含任何失败轨迹内容、不做失败归因、不写记忆库；rsi-agent 没有 ReasoningBank/SkillOpt/Evo-Harness 那样的外部经验状态，因此 M1–M4 错误与「经验无法迁移」都不在当前代码路径上。
- **【Agent 分析】方法论层面的真问题**：rsi-agent 的 SP 私集与公开集同分布（同一 ContextualTask 类、不同 seed），验证的是「换随机流后仍成立」，不是本文意义的「迁移到未见任务类型」。若把 rsi-agent 当「可泛化自改进系统」来宣称，本文会说证据不足——公开分涨+私集确认只证明 repair，不证明 generalize。
- **结论**：对 rsi-agent 是「宣称边界」层面的警示，非当下实伤。

### 3.4 可落地的修补

- **作者方向**：成功轨迹比更丰富的失败诊断更能支撑可迁移经验；建议未来做「更 general、可验证的经验迁移机制」。作者未提具体算法级 mitigation。
- **【Agent 分析】最小移植**：若 rsi-agent 未来引入失败驱动的经验写入（比如让 inner 记录被拒提案的原因），必须配套「成功轨迹锚」——即在 propose 时同时保留至少一个已验证成功的策略作为正例参照（当前 `outer.py` 取 best public_mean 为锚已经部分做到），避免纯失败归纳导致的 M3 扭曲。

### 3.5 局限与作者自陈未做的事 / 诚实缺口

- 作者自陈：评测集中于 web/embodied 的少量高难 benchmark，未覆盖多智能体交互/多模态；最长仅 ALFWorld 800 tasks 的有限步，无限时长下经验迁移如何演化未知。
- 诚实缺口：正文对「提升/下降」多为图示描述，缺显著性检验与置信区间，定量证据强度弱于 #1/#2/#4；Figure 2–5 的具体曲线数值未逐点核验（HTML 提取以正文叙述为主）。

---

## 4. Artifact-to-Capability: Diagnosing Breaks in Capability Formation of Self-Evolving Agents（2611.04952）

- 外链：https://arxiv.org/abs/2611.04952 ；全文 https://arxiv.org/html/2611.04952
- 作者/日期：Ziqian Ban, Haoxiang Yang, Yanfei Zhang, Shixin Liang, Haoran Yu, Xu Lin, Shaobin Zhao, Xuran Li, Zhiyi Cheng（天津大学 / 独立研究者 / IDEA）；arXiv:2611.04952v1 [cs.AI] 03 Nov 2026
- 本地全文：`.tmp/rsi-c/2611.04952.txt`（1892 行）

### 4.1 artifact→capability 五属性与 capability-formation break 判据（重点）

五个**顺序依赖**属性（§3.1，本地 L154-185）：

1. **Artifact formation（工件形成）**：持久结构已形成并接入后续决策；
2. **Behavioral activation（行为激活）**：同状态同接口下，有/无 artifact 的动作差异超过采样噪声；
3. **Task-relation realization（任务关系实现）**：artifact 产生正向效果——动作整体更满足相关任务关系，而非仅仅「变了」；
4. **Boundary-safe reuse（边界安全复用）**：artifact 不破坏已建立效果——实际决策中已确立的正向关系不被系统另一部分推翻；
5. **Capability extension（能力扩展）**：后续版本在保留已有效果的同时新增正向效果。

- **依赖链**：`1 → 2 → (3 且 4) → 5`。**属性 3 与 4 共同构成 checkpoint（能力形成检查点）**：两者都成立才称「capability formed」。属性 4 只关心「已确立正向关系在实际决策中被推翻」（fixed states 上的增减已计入属性 3）；属性 5 只对过 checkpoint 的版本有意义。
- **break 判据**：按 1→5 顺序诊断，**第一个失败的属性即 break**，依赖它的后续属性不再判定。
- **task relation 与「合法」的区分**（§3.1 L186-188）：task relation 由任务目标定义（如走自己的立即胜点、堵对手、成活四）；环境规则（如不落在已占点）不是 task relation——每个合法动作都满足规则却不推进任务。**属性 3 只比较合法动作，降低违规率不算正向效果**。这正是 Reflexion 被判 break 的关键。
- **系统级分析单元**（§3.2 L190-196）：判的是 frozen system（冻结版本里所有实际参与决策的 artifact 整体），不是单组件；单组件正确但组合后被推翻也算整体 break。

### 4.2 实验设计与控制条件（讲清）

- **受控环境**：freestyle Gomoku（无禁手五子棋），15×15。选它的三个理由（§4.1 L219-262）：① 状态可精确重放（任意盘面可还原，set A 还可旋转/反射扩充）；② 任务效果由规则程序独立判定（`G(s)` 可接受动作集，不依赖被测系统或任何模型）；③ 完整对局与版本全程归档。
- **三机制同基座**：DeepSeek-V4-Flash，同一环境、同一对手；Reflexion（文本记忆，模型直接选步）、Voyager（可执行技能库 + 自写 auto_player 调用脚本）、DGM（直接改策略代码谱系 P0→OB-1→…→OB-7）。
- **三尺度诊断**（§4.2 L304-319，Table 1）：
  - **state-level（属性 2、3）**：1876 个固定状态（A=120 手工战术态经旋转反射，B=1756 取自 DGM 进化实战）；控制=移除被测 artifact（Reflexion 空记忆；Voyager round1 只留首轮战术技能、调用脚本不变；DGM 初始策略 P0），因 Reflexion 采样动作，用两次空记忆运行之差当噪声基线。属性 3 = 合法动作中 pass rate 是否上升。
  - **game-level（属性 4）**：冻结系统与程序化对手 search_2 打 20 局（不同随机种子），每步重建盘面、识别所有 immediate-win state，记录是否成五；在立即胜点「一步成五」已是既定效果且规则给出唯一正确动作，失败即证明被系统另一部分推翻，**无需控制条件**。
  - **lineage-level（属性 5，仅 DGM）**：沿主谱系逐版本在 set B 上逐状态比较，数 newly passing / lost states 与胜场；另从归档进化对局建 7234 状态的 differential pool 比较末两版 OB-6/OB-7。
- **三个对手**：rule-based / threat-aware / search_2（最强，搜两层）。

### 4.3 可核对的核心 claim 列表

1. **【正文明确】** 主流「signal-based evaluation」的三个信号（artifact 累积、加载后行为改变、指标上升）都不直接测能力；定义 artifact→capability 五属性与三尺度诊断。位置：Abstract、§1、§3.1。
2. **【正文明确】** Reflexion 显 legalization gain：pass rate 上升完全来自 legality 上升（P(pass)=P(legal)×P(pass|legal)），legal pass rate 在 12 个组合中无一提升、set B 反降 1.6–4.4pp——属性 3 break。位置：§5.1（L386-407）；支撑：Figure 5、Table C.1。
3. **【正文明确】** Voyager 显 compositional override：20 局全部到达 immediate-win state，却 0/20 成五（全是 conflict state）——auto_player 防御优先早返回堵了对手，自己「win if possible」逻辑永不触达；各组件单独都正确——属性 4 break。位置：§5.2（L441-470）；支撑：Figure 4、Appendix E.1。
4. **【正文明确】** DGM 显 loss-only drift：OB-3 过 checkpoint（522 newly passing / 139 lost、19/19 immediate-win 成五），但 OB-5/OB-6 只丢不增（newly passing=0，lost 103/29，胜场 9/20→0/20），OB-7 在 7234 状态 differential pool 上只差 1 态——属性 5 break。位置：§5.3（L471-488）；支撑：Table 3、Appendix D。
5. **【正文明确】** 三个 break 落在不同属性，说明没有属性能替代另一个；三个信号全有却都没走完能力链，故信号式评价无法判定能力是否形成。位置：Abstract、§5（L385）、§7。
6. **【Agent 分析】** 属性 4 是最难用聚合指标替代的：它是「逐点硬约束」（每个 immediate-win state 都必须成五），不是均值。任何用「平均提升 + 显著性」的接受闸（包括 rsi-agent 的 paired_gate）在结构上都可能放行「净涨但逐点有 regression」的改动。

### 4.4 关键数字及来源（正文自报，本实验室未复算）

| 数字 | 出处 | 口径 |
|------|------|------|
| 五属性链 1→2→(3 且4)→5；3+4=checkpoint；三尺度 state/game/lineage | §3.1-3.2 | 定义，非数值 |
| 状态集 1876（A=120 / B=1756）；20 局 vs search_2；lineage differential pool 7234 态 | §4.2 | 正文自报，本实验室未复算 |
| Reflexion：59.8–78.2% B 态动作改变（噪声 46.7%）；整体 pass rate B 8.0%→9.2–10.3%、A 21.7%→22.5–27.5%；legal pass rate 12 组合无一改善、B 降 1.6–4.4pp；空记忆重跑改变 +2.5%/+2.4%/+0.6% | Table 2 / §5.1 / Figure 5 | 正文自报，本实验室未复算 |
| Voyager：7 技能+调用脚本；消融改 89–808 B 态；round1→full 459 新增通过 / 73 丢失；open four 0/20→20/20；immediate-win 成五 0/20 | Table 2 / §5.2 | 正文自报，本实验室未复算 |
| DGM：6 descendants；OB-3 相对 P0 改≥661 B 态、B pass rate 48.7%→63.0%(OB-7)、evolutionary score 150.4(OB-1)→312.6(OB-7) | Table 2 / §5.3 | 正文自报，本实验室未复算 |
| DGM lineage wins：P0 20/20、OB-3 19/20、OB-5 9/20、OB-6 0/20、OB-7 0/20；OB-3 522 增/139 丢(19/19 成五)；OB-5 0/-103；OB-6 0/-29；OB-7 1/0（7234 态只差 1） | Table 3 / §5.3 | 正文自报，本实验室未复算 |

### 4.5 反面结果对 rsi-agent 的具体威胁

- **哈希链审计 HA 对应它的哪一层？【Agent 分析】** HA（`rsi-agent/rsi_agent/audit.py:16-62`：`append()` 写 prev_hash+canonical_hash，`verify()` 查 seq 连续/链/自洽）只保证「记录未被篡改、顺序完整」，对应框架的 **Property 1（artifact formation）的记账版本**——甚至比 Property 1 还弱：Property 1 要求「持久结构接入决策」，而 HA 只证明日志没被改，连「artifact 是否真被调用」都不测（框架的 Property 1 靠 call records 确认，见 §5 L379）。HA 与属性 2–5 全部无关。
- **缺哪层 capability checkpoint？【Agent 分析】** 缺属性 2–5 全部，尤其是 checkpoint 的**属性 4（boundary-safe reuse）**。rsi-agent 的 sealed confirmation（`runner.py:66-161`：只在 pub.passed 后才 sealed.evaluate，再 priv.passed 才 promote）在性质上接近属性 2+3（私集是同状态换 seed 的行为差异 + 正向效果），但它：
  - **不测属性 3 的「task relation」纯度**——ContextualTask 的 reward 是连续内积值，没有「合法动作 vs 更优动作」的区分，等价于把「更合法」和「更会选」混在一起（正是 Reflexion 的 legalization gain 漏洞，只是当前没有 legality 概念所以不触发）；
  - **完全不测属性 4**——`paired_gate` 算 `delta_i = candidate_i − baseline_i` 后取均值、方差、ESS、显著性，从**不检查是否有实例从 baseline 的正向变成 candidate 的负向**（regressed_count / 逐点 Regression）。在连续 reward 下这表现为「某些实例 reward 下降但净均值仍正」，会被放行；
  - **不测属性 5**——没有「后续版本保留已有正向效果并新增」的谱系判定，`promoted_policy` 只是每代取公开分最高者，可能「换了策略但能力没扩展」（对应 DGM 的 OB-7 drift）。
- **现在就会中招吗？【Agent 分析】会，但形式是当前任务下的隐性版**：在 ContextualTask 上，属性 4 的严格形式（多组件组合推翻 immediate-win 这类既定能力）不存在，因为 rsi-agent 没有多组件、没有二值能力状态；但它的**一般判据**——「已建立的正向效果被推翻」——在当前实现里就是「candidate 使某些实例 reward 低于 baseline」，而 paired_gate 的数学定义保证这种情况只要净均值正就会放行。这是**当下代码路径上真实会发生、且无机制拦截**的逐点回归。迁移到 SWE-bench（二值 pass/fail）后，同一漏洞变成字面 compositional override：一个 patch 修好 8 题弄坏 2 题，`delta` 均值正、显著、ESS 够，闸放行——能力被覆盖推翻，作者意义的能力形成根本没发生。

### 4.6 可落地的修补

- **作者方向**：三尺度诊断需要「可重放状态 + 规则独立判定效果 + 完整记录」，并在 state/game/lineage 三层分别测属性 2–3 / 4 / 5；信号只是必要非充分。
- **【Agent 分析】可移植进 rsi-agent 的最小改动**（按性价比排序）：
  1. **属性 4 最小闸（当下可落地，纯 Python）**：在 `paired_gate` 里增加逐实例回归计数——`regressed = sum(d_i < -tol)`，其中 `tol` 可取 noise_floor 的一定比例；拒绝 `regressed > 0`（严格）或 `regressed > floor(R*×n)`（容忍少量）。这把「聚合均值闸」补成「逐点不下退闸」，直接堵住 §4.5 的当下漏洞，代价是可能拒绝一些净增益策略（等价作者的 margin-aware 代价）。
  2. **属性 3 纯度**：若迁移到有「合法性/约束」的任务，把 pass 拆成 P(legal)×P(pass|legal)，只对后者计增益，避免「更守法」被误记为「更聪明」。
  3. **属性 5 谱系闸**：在 `AuditLog` 里给每个 promoted_policy 记「新增正向实例集 / 丢失正向实例集」，promote 条件加「不得只丢不增」（DGM OB-5/OB-6 判据）。
  4. **HA 升级**：HA 现在只记 payload hash；至少把每次 sealed.evaluate 的 artifact hash 与 call 结果一并入链，使 Property 1 从「日志完整」升级到「artifact 确实参与决策」。

### 4.7 局限与作者自陈未做的事 / 诚实缺口

- 作者自陈：三种 break 在机制/任务间的分布需更大规模诊断；只覆盖三机制（文本记忆/技能库/代码谱系），未覆盖多智能体协同进化或直接参数更新；可判属性依赖机制结构（capability extension 只对有版本谱系者可判）；Reflexion 用模型直接选步、Voyager/DGM 用进化代码；不同基座/更长进化/重复运行会不会挪动 break 未测。
- 诚实缺口：实证只在 freestyle Gomoku（规则可独立判动作、状态精确重放、任务关系局部可枚举），作者在 §6 明说开放任务如何操作化是 open——**他们把「可推广到 GUI agent」标为 conjecture, without having tested it**，因此我们把属性 4 映射到 rsi-agent 是**我方推断**，不是作者验证过的结论；游戏级诊断「无控制条件」依赖「立即胜点唯一正确动作」这一强前提，迁移到无唯一正确动作的任务时判据失效。

---

## 5. 横向综合：哪一篇现在就会中招，最小修补是什么

### 5.1 四篇的失败模式在 rsi-agent 当前实现下能否发生（总表）

| 论文 | 字面失败模式 | 当前 ContextualTask 下 |
|------|--------------|------------------------|
| #1 masked erosion | 删冗余 safeguard 被 mask | 不成立（无冗余层、无 harness 自编辑）；迁移 SWE-bench 后出现 |
| #2 Third Eye | 参数自更新 continuation 倒退 | 不成立（inner 不训参数、每代单候选）；评价观警示 |
| #3 CFOE | 失败经验无法迁移 | 不成立（inner 不吃失败轨迹、无记忆库）；SP 私集同分布只测 repair 非 generalize |
| #4 artifact-to-capability | 属性 2–5 break，尤其属性 4 被聚合闸放行 | **部分成立**：HA 只≈Property 1 记账版；paired_gate 聚合判定放行逐实例回归 |

### 5.2 「现在就会中招」的实伤与条件

**判定：#4 的 Property 4（boundary-safe reuse）判据，在 rsi-agent 上现在就可能中招。**

- 条件：只要某个被 promoted 的候选策略相对 baseline 在**部分实例上 reward 下降**、而**逐实例配对差的均值仍 > 0 且过 noise_floor、ESS、显著性**（`rsi-agent/rsi_agent/gates.py:74-109`），闸就放行。这在 128 个 episode 的随机 ContextualTask 上几乎必然偶发（高斯扰动权重会在某些实例上变差）。
- 当前伤害表现：隐性质量退化（净均值正、少数实例变差），因为连续 reward 没有「能力全有全无」的语义；不是灾难性能力丧失。
- 升级条件：一旦任务换成 SWE-bench 二值 pass/fail、或 inner 开始改 harness/测试，同一漏洞变成字面 compositional override / masked erosion——真正的能力覆盖失败。
- 为什么其他三篇不算「现在就会中招」：#1/#2/#3 的字面机制都要求 rsi-agent 不具备的结构（冗余 safeguard 层、参数训练、失败经验记忆库），当前代码路径上无处触发；只有 #4 的判据漏洞直接落在现有 `paired_gate` 的数学定义里。

### 5.3 最小修补建议（单一最高优先级）

**给 `paired_gate` 加一道逐实例回归闸（属性 4 最小版）**：对配对差 `d_i = candidate_i − baseline_i`，统计 `regressed = #(d_i < −tol)`，`tol = c × noise_floor`（c 可取 1–2），当 `regressed > 0`（严格）或超过预设小比例时直接返回 NOT_SIGNIFICANT 并记 gate_reason。这是唯一一处「当下、零 GPU、约十行代码」就能堵住实伤的改动；其余（层消融闸、pass 纯度拆分、谱系增减记账、HA 升级）按 §1.4/§4.6 作为迁移前置加固。

### 5.4 诚实缺口（全局）

- 四篇均为论文自报结果，本实验室未复算；本机零 GPU，无对应实验环境。
- #4 的五属性框架作者自己只在 Gomoku 验证，向 rsi-agent 的映射是我方推断（作者明确标为 conjecture）。
- HTML 全文的数字取自正文叙述与 LaTeX alttext，Figure 2–6 / Table 3 等图像未逐像素核对；对曲线类结果（#3）尤其如此。
- rsi-agent 的源码事实来自本轮实读（`rsi-agent/README.md`、`gates.py`、`outer.py`、`tasks.py`、`barriers.py`、`audit.py`、`runner.py`、`core.py`），未运行其 demo 复算 verdict（属亚分钟 smoke，非必须；本笔记对 rsi-agent 的判断是静态代码分析结论）。

---

## 外链汇总（真实 URL）

- https://arxiv.org/abs/2610.38392 ／ https://arxiv.org/html/2610.38392
- https://arxiv.org/abs/2611.12988 ／ https://arxiv.org/html/2611.12988
- https://arxiv.org/abs/2610.22984 ／ https://arxiv.org/html/2610.22984
- https://arxiv.org/abs/2611.04952 ／ https://arxiv.org/html/2611.04952
