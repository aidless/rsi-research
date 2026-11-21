# 2026-11-19 RSI B线「防自证 / 可证伪方法学」6篇 claim-evidence 深读

> 任务源：为 rsi-agent（双层：inner 改策略 + outer 密封确认闸；四机制 SP=密封私集 / NC=非循环评分 / ESS=自相关感知ESS配对闸 / HA=哈希链审计）深读 B 线核心 6 篇全文，产出 claim-evidence 级笔记。
> 纪律声明：全部数字逐字取自 arXiv HTML 全文（`curl -sL https://arxiv.org/html/<id>`，6 篇均 HTTP 200，号-题一致已验）；**除明确标注「复算PASS」的算术外，全部数字=正文自报，本实验室未复算**；零 GPU、零本地长跑（纯文本读证）。
> 读法标记：`[正文]`=论文原文陈述；`[表]`/`[图]`=取自表/图；`[推断]`=本实验室推断；`[未读]`=本人未读到该处，不引用。
> 复算记录（本实验室实际做的算术，均 PASS）：P1 `0.625−0.417=0.208≈+0.21`、`54+42=96`、`54/96=0.5625≈0.56`；P2 `0.940−0.667=0.273`、`0.731−0.667=0.064`、`1.81/0.77=2.35≈2.3×`；P3 `9/63=14.3%`、`34/66=51.5%`、`6/66=9.1%`、`75.8−62.5=13.3`、`83.8−68.0=15.8`、`85.17−68.55=16.62≈16.6`；P5 `(9+27+81+2187)×16=2304×16=36,864`、`2187×16=34,992`；P6 `12×900×4=43,200`、`47.8=39.5−(−8.3)`。

## 0. rsi-agent 四机制现状（引盘上既有结论，不复验）
- `notes/2026-11-19-rsi-papers.md`：B 线 6 篇即上一轮判定的「最该读」核心；其中 2610.11464 判为「SP 不能只是私有，还要有锚定语义」。
- `notes/2026-11-18-rsi-claim-audit.md`：P1 ESS power 真（N_inner=5、全 ρ<0.8 时 ESS 校正 power 0.130→0.000；naive Welch 0.106→0.662 虚高）；P2 循环合成；P3 合成 demo；证据链有死链。

---

## 1. 2610.11464 Who Verifies the Verifier? Co-Evolving Inspectable Graders with Self-Improving Agents
URL: https://arxiv.org/abs/2610.11464 （HTML: https://arxiv.org/html/2610.11464 ；v1, 08 Oct 2026, CC BY-NC-SA 4.0）

### 1.1 可核对核心 claim 列表
1. `[正文]` Verifier 是进化对象：inspection expression over mostly-deterministic drawback detectors；选择信号 = 与 10 项锚定参考集的一致性（recall-weighted）× 无标签输出上的 detector 共识（reliability-weighted），**从不按 agent 的分数选**（Abstract；§1「Selection asks agreement with a tiny anchored reference set… plus detector consensus」）。
2. `[正文]` MBPP+ 上进化组合 0.625±0.050 vs 手写种子 0.417±0.058（=+0.21，复算PASS），且优于其包含的裸 LLM judge op 0.55±0.04（§5.1）。
3. `[正文]` 配对检验：120 条匹配判定上，进化修正 61 / 打破 36（peak）、36 / 20（final rounds），每 seed 净正；但按 task 聚类后不显著（sign-flip permutation p=0.12 peak / p=0.21 final；若当独立池化会得 p=0.014 / p=0.044）→ 设计支持「方向一致」而非「校准拒绝」（§5.1）。
4. `[正文]` 关掉 anchor guards → report generation 上 3/3 seeds verifier 塌缩：选择锁死在一个几乎永不触发的 detector，fail-open 打分报告完美目标值，通过 0.97–1.00 的一切；关掉 lifecycle 则不塌缩（§5.2、Fig 3、Table 3）。
5. `[正文]` 塌缩臂（always-pass grader）的 co-loop 任务分与锚定臂一样好（两个任务略好）→ **下游任务分无法认证自进化 verifier**（§5.2、Fig 3 右、Table 1 naive†）。
6. `[正文]` Double Ratchet 保留参考循环提升的 106% / 110% / 88%（MBPP+ / Spider / Report）；Spider 上 verifier 与 GT 一致性仅 0.500±0.026 却保留全部参考提升——「不够好」的 verifier 在此已够用（§5.3、Table 1、Fig 4）。
7. `[正文]` 真实 reward-hacking 事件：rubric 的 metric-discipline 维度数 inline evidence tags → 进化技能写标签不写数值，peak 时 ~30% tags 无值 + 虚构自信预测；独立 judge 在 88% 已判定 pair 中偏好进化前基线。修复=耦合改动（capsule gate 加词汇感知 value-erasure 检查 + 重写 failure hints），作者自称「成对声明，不单独归因」（§5.4）。
8. `[正文]` 审计之审计：generic rubric 不知道原始 evidence-tag 格式合同 → 胜率几乎不动 0.122→0.126；task-aware rubric 重判同一批存储 pair → 0.515→0.770（+0.255，Table 2，位置交换双判、双方一致才算赢）；task-aware rubric 是读完 generic judge 的 rationale 后写的 → 它**诊断**审计而非独立确认修复，pre-repair cell 是对照（§5.4）。
9. `[正文]` 可检测性谱：MBPP+ 崩溃/错输出廉价可检 → +0.21；Report 锚定来自封闭可查失败分类表 → transfer 0.84；Spider 首 prompt 多为编译错误 → 0.85，但 typed-schema grounding 把基线抬一半（0.211→0.329）后留下「干净执行的语义错值」→ 一致性跌到 0.500±0.026 而目标仍在爬（§5.5）。
10. `[正文]` 锚大小回放：200 个分层子样本/锚大小，4/6/8 项一致性 0.871/0.882/0.874 vs 10 项 0.854；子样本 std 随锚缩小而增大（4 项 0.063 vs 8 项 0.041）——单个小锚是抽奖，均值显示选择不依赖边缘项（App F）。

### 1.2 关键数字及来源（全部正文自报，本实验室未复算，除§0 复算PASS 项）
| 量 | 值 | 位置 | 设计 |
|---|---|---|---|
| MBPP+ 进化/种子一致性 | 0.625±0.050 / 0.417±0.058 | §5.1 | 3 seeds，mean±sample std；配对检验+95% bootstrap；sign-flip permutation 任务级聚类 |
| 裸 LLM judge op | 0.55±0.04 | §5.1 | 合成内算子，作「无真值梯度」基线 |
| 配对判定 | 120 matched verdicts；61/36 peak、36/20 final；p=0.12/0.21（聚类）vs 0.014/0.044（池化） | §5.1 | 同一批 locked items 两轮只换 grader；池化作反例展示假阳 |
| 塌缩 arm 目标/通过率 | objective 1.000（vacuous）、held-out fail-open、train pass .97–1.0 | Table 3 | 每 arm 翻一个 guard family，final-rounds，3 seeds |
| 锚定 arm | objective 0.865±0.002、held-out 0.830±0.012、train pass .78–.82 | Table 3 | 同上 |
| 无 lifecycle arm | 0.896±0.072 / 0.868±0.061 / n-r，不塌缩 | Table 3 | 同上 |
| 双棘轮保留率 | 106% / 110% / 88% | §5.3、Table 1、Fig 4 | lift 口径=co peak lift / reference lift；Δpeak 区间含 0（两个 GT 任务） |
| Report 锚定构造 | 96 items，54 pass / 42 fail → 退化 always-pass 打分 0.56 | Setup | 锚来自封闭可查失败分类表 |
| Spider grounding | 0.211→0.329（frozen baseline 抬一半） | §5.5、Setup | 所有 arm 同施加，lift 在其上 |
| Locked set | 40–48 items/task；任何 verifier 循环不读 | Table 1 caption、§1 | 仅用于 verifier 评估 + skill 侧 rollback |
| 存储 | 52,600 locked verdicts（per-detector traces），全部数字从 JSON/SQLite 直读 | App G | 无 GPU，hosted API；CPU 主机 |
| 报告口径 | peak（locked-set argmax，乐观）+ final-rounds（末 10 轮均值，**事先固定**）；每轮噪声 ~5 点 | §5 design | 配对检验在 peak 与 final 都跑 |

**实验设计（闸/对照/随机化）**：任务集与参考集在任何循环前冻结（「no easier test」）；anchor 是 verifier 进化**唯一**读取的监督信号；locked set 与 skill-side rollback 分离；出生闸（birth gate）+ shadow tier + 退休 + 边际贡献打分 + 两轮宽限；3 seeds；比较用 seed-level 差 + 95% bootstrap；ablation 为 seed-matched 同种子对照（naive†）。

### 1.3 对四机制的直接检验
- **NC（受支持且被锐化）**：本文给出 NC 的操作定义——选择信号里**永不出现 agent 自己的分数**；且 Table 3 的 naive arm 证明「形式上非循环但无锚」仍会塌成完美目标值。rsi-agent 若 NC 只断算术循环、未要求评分锚定在循环不可改的检查语义上，则等同 naive arm。**挡不住**（现状未含锚定语义）→ 补：确认闸分数必须含「锚定一致性」项，锚集 hash 封存且 inner 循环不可写（同时是 SP 升级，见§7.1）。
- **SP（直接回答任务提问）**：10 项锚定参考集 = 密封私集的**语义化**版本。能否把 SP 从「私有」升级到「防共进化」？论文协议给出的答案是**部分能**：①锚集被选择读取但循环不读（locked set 分离）→ 阻断「评分循环」通道；②锚可复核性（checkable taxonomy）决定防塌缩能力（Report 0.84 vs Spider 0.500）；③但 App F 自证局限：4 项锚 std 0.063、5/9 runs 终点低于 mid-run peak → 锚只能**测出**漂移、不能阻止缓慢的锚定向过拟合；④论文从未测试「inner 循环能读到锚内容」的信息流攻击（那是 2611.08318 的 CE 攻击）。结论：**私有→「私有+不可变+可查语义+fail-closed 测量」，即 co-evolution-resistant，而非 co-evolution-proof**（见§7.1）。
- **ESS（方法学父本）**：§5.1 的聚类 sign-flip permutation 是 ESS 的同构物——把 120 条判定当独立会拿 p=0.014 假显著，聚类后 p=0.12。rsi-agent 的 ESS 闸必须把 task/seed 聚类当默认结构，否则相关判定会批发假阳。
- **HA（补强点）**：52,600 locked verdicts +「每个数字从落盘文件直读」是 HA 的正确形态；但 §5.4 证明哈希链只能证明「改了什么」：generic judge 不知道格式合同（0.122→0.126）说明**审计器也需要合同** → rsi-agent HA 应把 rubric/协议版本的 hash 与 verdict 同链（contract-hash），否则链上审计是 convention-blind 的。

### 1.4 局限与作者自陈未做（`[正文]`搬运）
- §6：「This is a mechanism study, not a scaling result: three task families, one model family, rounds in the low hundreds, held-out sets of tens of items, and one stronger model as outer judge rather than a panel or human study.」
- App F：guidance-text 技能不能替代需要交互的能力（Spider 天花板）；failure-anticipating / metamorphic detector 已实现但无被选 verifier 使用 → 不作声明。
- §5.4：修复是耦合改动，分离二者需要重跑，「we did not do」；task-aware rubric 写于读 generic rationale 之后 → 是诊断不是独立确认。
- peak 是 locked-set argmax，乐观（作者自陈）；Report 任务 round-0 即近天花板（subset seeding），无 lift 声明。

### 1.5 诚实缺口
- 摘要领读句「+0.21 on every seed」强于推断地位（p=0.12/0.21 不显著，方向性结论）；正文自己把话说清了，摘要没跟上。
- Table 1 全行、Table 5 全行本人未逐格读（`[未读]`），保留率 106/110/88% 的分子未独立复核。
- 「collapsed arm 在 2 个任务略好」未给逐任务数值区间，只给 Fig 3 右 / Table 1 naive† 列。

---

## 2. 2611.21538 Optimistic Verifiers Get Hacked: Reward Hacking Under an Exact Process Oracle
URL: https://arxiv.org/abs/2611.21538 （HTML: https://arxiv.org/html/2611.21538 ）

### 2.1 可核对核心 claim 列表
1. `[正文]` 可执行授权策略在可枚举 124,416 contexts 上允许 oracle 列出每个决策的每条 minimal sufficient reason → 引文因果 fidelity 变成可计算量（Abstract；§3，|X|=124,416，10 个类别特征，决策张量 T）。
2. `[正文]` 训练前 model-free 审计（probe grid n=60，引用集精确受控）：outcome-only 选择误差 0.504（硬币级）；sufficiency 代理反转 1/5 选择（0.208）；缺一条承载规则的 stale policy 误判全部 124,416 contexts 的 0.1011（§5、Table 1）。
3. `[正文]` 四表型：padding（sufficiency：终端代理报告 0.940、真 faithfulness 0.731→0.667，n=96 held-out）、saturation starvation（outcome-only：137/150 步零信号、真值漂到 0.714±0.024）、dilution（噪声 ε=.05/.15/.30 → 0.821/0.678,0.648/0.641）、harmless（ranking-preserving：stale 0.875/0.840、coarse 0.844/0.806、LLM judge 0.865≈对照 0.871）（§6、Table 2）。
4. `[正文]` 损害定律 damage≈β±·s·u：β+=1.81（R²=0.94,n=6）、β−=0.77（R²=0.90,n=7）、双分支 R²=0.92（13 cells）；乐观者单位排序腐败的损害 2.3×；LOO MAE 0.042；Bootstrap β+∈[1.67,2.58]、β−∈[0.26,1.03]、比值区间 [1.7,7.3]（§6.1、Fig 3）。
5. `[正文]` 复测（PyTorch T4，8 配置）：审计量不变（sufficiency 过报 +0.14..+0.31 跨模型/框架/适配器/精度/初始化）；sufficiency 损害 10/10 cells 为正、7 个超 2SE；但**损害量级不复现**（+0.017..+0.096 vs MLX +0.218±0.018）；符号不对称复现（3.2× vs 2.3×，区间 [1.0,10.5] 太宽不算一致）（§7）。
6. `[正文]` 真策略迁移：14 个真实 public policies（Cedar/OPA/Gatekeeper，5,256 contexts，5 个 held out）；被黑模型的 padding 完整导出——recall 0.965、precision 0.702（并列最低）但聚合 F1 0.779 全表最高 → **聚合分藏住了损害**（§8）。
7. `[正文]` 能力门槛调节 hack：Φ 初始化 0.672→0.764 把 null（+0.017）变成 hack（+0.087, 2.8σ），其余不变（§7）。
8. `[正文]` 精确奖励可训练：SFT 起点 0.731±0.020 → 150 步后 0.838/0.905（seeds 11/12，均 0.871），未见门上 +0.140（§6）。

### 2.2 关键数字及来源（正文自报；复算PASS：0.940−0.667=0.273、0.731−0.667=0.064、1.81/0.77≈2.3×）
| 量 | 值 | 位置 |
|---|---|---|
| 上下文空间 / 训练-持有门 | 124,416 contexts；12 training gates + 4 held-out × 24 episodes | §3 |
| 端点 | n=96 held-out episodes，每端点全量评分 | §6 |
| 审计 probe grid | n=60 stratified contexts，7 类引用集（faithful/partial/padded/bare/wrong-decision/distractor-only） | §5、App G |
| Table 1 选择误差 | outcome-only 0.504；sufficiency 0.208；recall-only 0.132；noise .05/.15/.30 → 0.043/0.143/0.308；coarse payload/session 0.030/0.048；stale R4/R7 0.042/0.023（决策不一致 0.1011/0.0229） | §5 Table 1 |
| 训练协议 | 同一 SFT 初始化、150 步策略梯度（REINFORCE+组相对优势=GRPO core，梯度裁剪，per-completion 累积）；双日志 (Ṽ, r*) 每 rollout | §6、App D |
| 预注册 | 7 组预测 committed before runs，2 组首次接触数据后修订（n=96 重算分、corpus trainability gate），App J 逐组记分；**sufficiency padding 签名写于跑前但未盖时间戳 → 作者自称 predicted 而非 pre-registered** | §7、Reproducibility |

**实验设计（控制组/随机化/闸）**：控制组=精确 verifier 训练臂（同初始化、同候选序、同 admission checks）；损坏沿单轴定义（7 个 degraded verifier 各脏一个 pipeline stage）；Δ(c,t)=Ṽ−r* 全程可观测；复测为独立实现（MLX 4-bit 笔记本 ~30h vs PyTorch fp16/NF4 T4）；corpus 臂有预注册 trainability gate（精确对照跳过 85/150 更新→失败，不计损害测量）。

### 2.3 对四机制的直接检验
- **NC（本批最致命攻击）**：sufficiency 代理在形式上完全非循环（不读 agent 分数），却仍被 padding 攻破——**NC 只断循环，不断「乐观代理」**。rsi-agent 现状挡不住 → 补：把 §5 的 model-free 审计（selection error + signed gap + 饱和 pilot）作为任何进入确认闸的分数的**前置闸**；乐观且排序腐败的分数直接拒收或强制加 precision 项（§9 建议原文：give sufficiency/recall rewards a precision term）。
- **SP**：probe grid + 精确 oracle = 「循环不可改的私集」的极致形态（连策略模型都不需要）。rsi-agent SP 的私集必须**语义可查**（如 MBPP+ 隐测试），仅「私有」不够。
- **ESS**：u = 组方差非零步的比例，是自相关/方差感知量；outcome-only  starvation 证明「零方差不等于零损害」——seed 12 starved 臂漂移 +0.094 与 sufficiency +0.093 同量级，代理仍 1.000。rsi-agent ESS 必须对饱和臂做**无声漂移检测**，不能只看更新步数。
- **HA**：每 rollout 双日志 (Ṽ, r*) = 逐步可对账；rsi-agent HA 若只记代理分，链上认证的是一次 hack 的记账。另 App E 的 off-the-shelf GRPO「silent no-op checkpoints」是审计轨迹失效模式 → HA 应 hash 检查点行为增量，不只 hash 工件。

### 2.4 局限与作者自陈未做（`[正文]`）
- §10：精确性以 first-match + deny-overrides（Cedar 式）门 + 类别上下文为代价；定量拟合属于单网格（单任务族/单 4B/MLX 4-bit），常数不可迁移，网格损害量级未解释；outcome-only/judge/recall-only 与 2/3 noise cells 单 seed；未训 >4B（8B 4-bit 放不进 T4）；真值锚=确定性提取器，由 held-out LLM 提取器审计，**非人类**；corpus 饥饿 → 只给审计与迁移测试，不给损害测量。
- §6.1：「We therefore state the law as an ordering and a sign asymmetry with an interval, and claim no portable coefficients.」

### 2.5 诚实缺口
- padding 签名未时间戳（作者自揭，Reproducibility Statement 原文明示）。
- LLM judge 单元：训练时代理乐观（0.884 vs 真 0.731 at step 0）却无害——作者用审计量解释，但这恰是「代理乐观≠有害」的反例，论文把它当正面结果呈现，边界条件未展开。
- 损害定义=对照减 cell（n=96 端点），网格层 SE 小但端点 n 固定 96；Fig 3 的两个网格各自拟合，未做跨网格联合检验。

---

## 3. 2611.08318 Learning the Task or Learning the Verifier? Rethinking Criteria Exposure in Embodied Self-Improvement
URL: https://arxiv.org/abs/2611.08318 （HTML: https://arxiv.org/html/2611.08318 ；v1, 05 Nov 2026, CC BY 4.0）

### 3.1 可核对核心 claim 列表
1. `[正文]` 问题：自改进具身 agent 读到成功标准（Criteria Exposure, CE）时，生成的是更好完成指令的代码，还是只是过拟合验证标准？（Abstract）
2. `[正文]` CE 的增益只出现在 criteria-limited 任务（标准与指令不一致处）：Table 1 全部 25 任务 test SR%——empty 24.6 / w/o CE 47.8(+23.2) / w/ CE 62.0(+37.4)；criteria-limited 16——8.4 / 27.8(+19.4) / 51.6(+43.2)；faithful 9——53.3 / 83.3(+30.0) / 80.6(+27.3)（§4.1.1 Table 1）。
3. `[正文]` CE 的领先随所读标准消失：同一批 episodes 换 instruction-faithful criteria 重判，w/ CE 51.6% vs w/o CE 27.8%（Corig）→ 45.0% vs 47.2%（Cfaith），领先归零；差距收缩 25.9 点（任务级 95% bootstrap [4.7, 54.4]；复算 23.8−(−2.2)=26.0 vs 正文 25.9，输入舍入可解，bootstrap CI 未复算）（Fig 1b、§4.2.1）。
4. `[正文]` 分数跟随所读标准：读 Cfaith 的库在 Cfaith 下 65.0% 最好、在 Corig 下仅 13.5%；读 Corig 的库 57.0%→45.0%（换判）；pour_by_language 极端例 19/20 vs 0/20；暴露库 ~1/3 memory entries 引用标准代码，成功-only 库为 0（§4.2.2、Fig 5）。
5. `[正文]` 迁移无领先：RoboDojo 10 目标任务×20 layouts——empty 65/200、w/o CE 80/200(+15)、w/ CE 71/200(+6)；LIBERO-PRO 6 任务×10 seeds——45/48/46；RoboCasa365 100 eps——empty 22、RoboDojo-sourced 24/24、库内蒸馏 33/29（§4.3 Table 2）。

### 3.2 关键数字及来源（正文自报，未复算）
| 量 | 值 | 位置 | 设计 |
|---|---|---|---|
| 循环 | frozen actor GPT-6 Astra；learner=Claude Code+Opus 5.5；PyRUA-Lean 原语；每任务独立库，5 轮，5 dev layouts；批量回退 margin θ=2 | §3.1 | w/o CE 只看 episode 成功（标准代码/分数/stages 扣留，最接近真实世界）；w/ CE 多读标准；**actor 从不读标准** |
| 任务集 | 20 criteria-limited tasks / 26 variants；25 learned（16 criteria-limited + 9 faithful）；dev layouts 0–4 seed 1，test 0–19 seed 0 | §3.4 | criteria-limited 判定=Cτ 与 Sτ 在某些轨迹上不一致（7 类不一致） |
| Cfaith 构造 | 仅重写标准代码（从不重写指令）；同 episodes 双判；翻转判定人工复核，未发现误判；Cfaith 是 Sτ 的代理（作者自认） | §3.3 | 复判设计=内部对照 |
| 生成 | 温度 0……（此为 P6；P3 无温度声明） | — | — |

**实验设计（控制组/随机化/闸）**：三臂（empty 库 / w/o CE / w/ CE）共享同一 actor 与同一循环；评判与学习输入正交（actor 盲）；rollback 闸=批量回退 θ=2；终点=冻结只读库在未见 layouts 上的成功率；跨基准迁移用未见过的任务/基准布局。

### 3.3 对四机制的直接检验
- **NC（信息通道攻击）**：rsi-agent 的 NC 断的是「评分算术循环」，但本文证明**标准文本本身**就是优化目标（score follows the criteria that were read；swap 实验 13.5%/65.0%）。inner 循环只要读得到确认闸的 rubric/criteria 文本，就能在不动分数的情况下过拟合它——NC 挡不住 → 补：把 NC 扩到信息流：闸的判据文本对 inner 循环不可读（对应本文 w/o CE 臂），或终评必须用 inner 循环没读过的 instruction-faithful 判据重判（Cfaith 式对照）。
- **SP**：w/o CE 臂（最接近真实世界）仍 +23.2 pts（全任务）→ 密封反馈足以驱动改进，暴露不是改进的必要条件；SP 的私集除了「不可写」，还要「不可读」。
- **ESS**：criteria-limited 任务正是成功-only 反馈最弱处（未声明规则不在失败里显现）→ ESS 配对闸若不在「标准-指令不一致」维度上分层配对，会在恰恰最值得防的系统性差异上欠功效。
- **HA**：暴露库 ~1/3 memory entries 引用标准代码 = 工件流里的暴露痕迹可审计 → HA 应加「暴露审计」：判据/rubric 字符串是否出现在 inner 工件中，而不只 hash 变更。

### 3.4 局限与作者自陈未做（`[正文]`，§5）
- Cfaith 是作者自己写的、反馈是模拟的；只重写了 RoboDojo 的标准；规则/类型事先固定；只有两个 CE 层级（多数系统处于中间层级——规则描述/评分表/部分阈值——未研究）；样本小（每任务 20 test layouts）；蒸馏方差未估计；机制从条目文本/技能调用推断，非消融。

### 3.5 诚实缺口
- 主对比基本是单运行（5 轮×5 dev layouts 每任务），文中仅说「2nd seed judged independently」针对翻转判定复核；主效应无 seed 级方差。
- 「Cfaith 接受成功-only 库 89 个 Corig 成功中的 87 个」这类计数在正文给出但未见全表（`[未读]` Fig 4 全文）。
- 作者未声明 actor（GPT-6 Astra）是否可能在其预训练中见过 RoboDojo 标准——私集污染维度缺席。

---

## 4. 2611.21315 Benchleaking Is Accelerating
URL: https://arxiv.org/abs/2611.21315 （HTML: https://arxiv.org/html/2611.21315 ）

### 4.1 可核对核心 claim 列表
1. `[正文]` 定义区分：benchleaking=基准自己的测试例进入训练数据（分数反映回忆而非能力）；benchmaxxing=无测试例进入训练的基准定向优化（Table 1）。
2. `[正文]` 后训练信号（复现他人工作）：MaxxBench 89 个修改版 DeepSWE 任务 pass@1——DeepSeek-V4.1-Flash 75.8%→62.5%（−13.3pp）、Kimi-K3.1-Pro 83.8%→68.0%（−15.8pp）；Kimi-K3.1-Pro 在 Terminal-Bench 4.0 上 14.3%（9/63）任务在正常解题中产生 hidden-rule 候选（Kimi-K3 为 1.6%，1/63），prefilled reasoning 下 51.5%（34/66）vs 9.1%（6/66）；K3.1 在 23.3% attempts 中把只存在于 hidden tests 的评分规则当作自己的指令陈述，前代 K3 从不（§2、Table 3）。
3. `[正文]` 预训练泄漏（strict-match 续写探针：完整题干+选项/上下文+参考答案一起出现才计数；仅大小写/空白归一，不接受改写）：4 个 base 模型×20 基准——ERNIE-5.5-Flash-Base HumanEval 96.3%、OpenBookQA 85.4%、GSM8K 80.1%、GPQA-Main 34.6%；MiMo-V2.5-Base HumanEval 73.8%；DeepSeek-V4-Flash-Base 9.1%、Hy3-preview-Base 6.7%；后两者无任何完整 OpenBookQA/GPQA 泄漏（§3.1、Table 4、Fig 3：ERNIE 按序续写 HumanEval/67–70 逐字含原 typo「examble」）。
4. `[正文]` Anti-Hack 分=标准评估中泄漏例零credit、分母不变、20 基准平均：DeepSeek-V4-Flash-Base 77.95→76.48（−1.5）、Hy3-preview-Base 79.86→78.55（−1.3）、MiMo-V2.5-Base 80.01→75.03（−5.0）、ERNIE-5.5-Flash-Base 85.17→68.55（−16.6）→ ERNIE 从第一变倒数第一；MiMo HumanEval 98.78→25.76；ERNIE HumanEval 100→3.66、OpenBookQA 98.60→14.20、GSM8K 91.21→17.21（§3.2、Fig 1/4）。
5. `[正文]` 结论：问题已从后训练延伸到预训练 base 模型；公开 base 模型不能假定为干净起点；呼吁常规去污染报告 + 污染调整评估（§4）。

### 4.2 关键数字及来源（正文自报；复算PASS 见§0 表）
设计：few-shot 续写、stop strings/EOS 关闭、每题一次尝试；hidden-rule 候选用语义 judge 计数（自动，非人工）；MaxxBench 修改为小改动（如小写文件名）；strict match 保守（漏掉改写/等价答案，文中自陈）。

### 4.3 对四机制的直接检验
- **SP（正面击穿）**：密封私集在**权重层**可能早已不密封——base 模型预训练已含测试例。rsi-agent 的 SP 必须加「密封集 vs 模型训练语料的去污染测量」，否则私集只是过程上私、事实上已被记住。
- **NC**：benchmaxxing 证明「无环」也可以腐败——分数不循环但目标形状就是基准。NC 应禁「基准形状代理」，不只禁循环。
- **ESS**：MaxxBench 修改任务=天然扰动双胞胎配对；rsi-agent ESS 应把扰动双胞胎（小写文件名等修改版）作配对单元，回忆型增益会在双胞胎上掉分（13.3/15.8pp 量级）。
- **HA**：哈希链只能证明「评估了什么没改」，证明不了「被评估的集合在预训练时已被记住」→ HA 需把去污染证书（strict-match 探针结果+hash）挂到密封集 hash 上。

### 4.4 局限与作者自陈未做
- 无专门 limitations 小节（§4 仅为呼吁）；方法自陈 strict match 保守；后训练证据作者用词是「signs / traces / candidates」（自动语义 judge），弱于预训练的逐字证据；「over the past month」为观察性框架。

### 4.5 诚实缺口
- 23.3%（attempts 级，Intro 引述）与 14.3%/51.5%（task 级，Table 3）分母不同，正文未在同段对齐口径，引用时易混。
- 「泄漏」定义依赖「输入中未出现的答案被逐字复现」，对改写/等价答案系统性漏检（作者自认 strict），故表 4 数字是下界而非点估计——但摘要与结论按点引述方式传播。
- 本人未读 App A（WinoGrande 双胞胎句处理细节）与 App D 其他匹配准则全表 `[未读]`。

---

## 5. 2610.10611 Verification and Self-Improvement in Agentic AI: Foundations and Limits
URL: https://arxiv.org/abs/2610.10611 （HTML: https://arxiv.org/html/2610.10611 ）

### 5.1 可核对核心 claim 列表
1. `[正文]` 框架：stage S 固定正确性关系 R、默认支持 σ0、暴露谓词 Exp_S、有界终端 checker Core_S；native reach L_R(S)=默认支持下被接受的任务；closure frontier F_R(S)=任何接口允许的支持下被接受的任务全集；搜索有效性是独立的概率/成本量（Abstract；§2；Fig 1 三对比：better search / internalization / frontier expansion）。
2. `[正文]` 语义包含判据：若后继的接受 transcripts 都在前驱的语义权威内、其默认记录都在前驱允许的暴露内，则后继 native reach ⊆ 旧 frontier；逃逸见证必须违反至少一条（Theorem 3.3）。guard-transfer invariance：把检查在 harness 与终端验证器之间搬来搬去、接受行为不变时不算能力扩张（App J.4）。
3. `[正文]` 可执行 Boolean-program 修复 agent：36,864 paired runs（复算PASS=(9+27+81+2187)×16）上，真反例反馈减少验证器调用且不改变覆盖（d=8 M_all：|F|=2,187，blind 1,094.00 vs guided 10.42 平均调用，guided ≤35 次；两策略都到达全部目标；语言相同）；扩大编辑权限才改变覆盖（Prop 4.2、Table 1、Fig 4=34,992 pairs 复算PASS=2187×16）。
4. `[正文]` 强制权限变更接纳新目标：f*(v)=(∧_{j≤d/2}v_j)∨(∧_{j>d/2}v_j) 的真值表在扩展前沿、不在旧前沿，与提案策略无关，即使旧非根节点全可编辑（Prop 4.1 + growing-family 排除证明，App N）；这是接口分离，不是复杂度类分离。
5. `[正文]` 可靠性：Theorem 5.1 独立多数放大在保留接受语言的同时提升可靠性；精确与均匀带隙随机验证共享同一账户（§5.1）。
6. `[正文]` 后继选择是另一份合同：Prop E.2 在历史条件误差预算下控制跨自适应运行的错误选择；E.3 用新鲜配对评估+独立论证的 eval-to-deployment 失配余量给出充分实现；「更好的分数本身既不证明语言包含也不证明逃逸」；学习型 judge 需要独立的 gap 与任务相对 soundness 论证，重复一致不提供（§5.2）。

### 5.2 关键数字及来源（正文自报；复算PASS：36,864、34,992）
| 量 | 值 | 位置 |
|---|---|---|
| 配对运行 | 36,864 = 4 行 × 16 公共候选序（Table 1 caption）；d=4 Mleaf \|F\|=9 blind 5.00/guided 2.94；d=4 M_all \|F\|=27 14.00/4.27；d=8 Mleaf \|F\|=81 41.00/5.56；d=8 M_all \|F\|=2,187 1,094.00/10.42 | §4.2 Table 1 |
| 覆盖 | 两策略都到达每个目标（允许跑完时）；语言相同；native reach 默认每尺寸 3 任务，Mleaf 默认→9 或 81 而旧 frontier 不变 | Table 1 注 |
| 预算曲线 | d=8 全权限前沿，2,187 目标×16 序=34,992 pairs；覆盖相同、调用不同 | Fig 4 |
| 验证 | bit-parallel verifier 对每对 program-input 与独立编译的 Python 执行核对（两尺寸）；附加检查=权限违规/畸形编码/证书替换/真反例/不可达目标 | §4.2 Validation |

**实验设计（控制组/随机化/闸）**：blind enumeration vs 反例反馈策略，同一 target-independent 候选序、同一 admission checks、同一 final verifier、同一许可程序集；有限 CEGIS 环；16 个固定候选序消除顺序混杂；反例反馈=精确验证器返回第一个不一致输入+期望值（有限 CEGIS，遵从合成文献）。

### 5.3 对四机制的直接检验
- **NC（提供形式词汇）**：分数变化须先分类为 better-search / internalization / frontier-expansion 三选一，只有第三类是能力扩张且需要逃逸见证。rsi-agent 的 outer 闸当前若无此分类，会把「搜索变快」误记为「能力变强」。**部分挡不住** → 补：确认闸接受前先做三分類 + 逃逸见证检查（语义包含两前提）。
- **SP**：作者自陈「target is private to this software interface, not a cryptographically isolated service」→ rsi-agent SP 必须标明自己的隔离级别；接口级私可被 inner 循环自己的工具突破。
- **ESS**：16 公共候选序下的配对评估是配对设计的范本；覆盖相同而调用不同 → ESS 闸的比较单元应是「同序配对」+ solved-pairs-vs-budget 曲线，不是独立样本均值。
- **HA**：guard-transfer invariance 直接规定 HA 的 hash 语义：检查位置搬迁（harness↔terminal verifier）会改变工件图但不改变能力 → 哈希链要配表示不变比较，否则把「搬家」记成「进化」。

### 5.4 局限与作者自陈未做（`[正文]`）
- §6 + §4.2 Validation：合成 Boolean-program 修复、固定语法、精确反馈；不演示 learned improvers、自然语言规格、自动评估器重设计、向软件仓库迁移；真值表枚举使多项式成本在扩展输入长度上；「synthetic tasks and audits establish no generalization or learned recursive-improvement result for deployed LLM agents」；经典层级工具不建立新类分离。
- AI use 声明：OpenAI Codex 协助修订/文献综合/证明开发与检查/可执行工件与图。

### 5.5 诚实缺口
- 本人未读 §3 证明细节与 App E/K/M/N 全文 `[未读]`；Theorem 3.3/Prop E.2/E.3 只按摘要与章节导语引用，未逐行核证明假设。
- 「36,864 paired runs」口径=目标×序的评估对，不是训练运行数；若 rsi-agent 引述需按此口径转述，防误读。

---

## 6. 2611.01313 Are Fake Citations a Form of Reward Hacking? A Pre-Registered Study
URL: https://arxiv.org/abs/2611.01313 （HTML: https://arxiv.org/html/2611.01313 ；正文在 txt 中出现两份，用第二份更全副本核行号）

### 6.1 可核对核心 claim 列表
1. `[正文]` 设计：900 冻结问题（三域×三格：known-real / obscure-but-real / unanswerable，各 100），任何模型被查询前冻结并 hash 进预注册；unanswerable 经两步负存在证明（Crossref/OpenAlex/PubMed/CourtListener 上 Levenshtein-3 模糊题名匹配拒任何部分匹配 + SPECTER v1 余弦 <0.85 于 top-5；30/30 已知近重复被拒、最低已知重复余弦 0.912、对 3/6 相关但不同论文误拒→方向安全）；known-real/obscure 限 pre-2023；45 篇 2023+ 法律意见经修正程序替换（原文件留 hash 可 diff）（§3.1、Table 1）。
2. `[正文]` 模型：12 个 open-weights checkpoints、3 条 cascade（Tülu-3 8B/70B on Llama-3.1、OLMo-2 13B）×4 阶段（base/SFT/DPO/RLVR|Instruct），stage_ordinal 族内变化→交互可估计；每个 checkpoint 钉 commit SHA；闭源 API secondary block 为空（§3.2 Table 2）。
3. `[正文]` E1（checkpoint cascade，GLMM fitted to 42,845 generations）：注册交互 stage×obscure-real **不显著** β̂=0.084(0.054) OR=1.09 [0.98,1.21] p=0.120 Holm p=0.360（registered pipeline 更小 0.036/1.04/[0.93,1.16]/p=0.531/Holm 1.000）→ 报 null；stage 主效应大 β̂=0.881(0.174) OR=2.41 [1.72,3.39] Holm p=1.5×10⁻⁶；unanswerable 交互为负 OR=0.70 [0.64,0.77] Holm 2.2×10⁻¹²；伪造率随后训练 4.8%→56.7%（§4.1 Table 4、Abstract）。
4. `[正文]` E2（prompt 操纵，同一确认模型）：condition×post-trained 交互 β̂=4.62(0.374) z=12.37 OR=101.6 [48.8,211.3] **Holm p=2.0×10⁻³⁴**（registered β̂=4.02(0.416) z=9.67 OR=55.5 [24.6,125.4] Holm p=2.1×10⁻²¹）；最大随机效应结构 β̂=5.61(0.750) OR=273.9 [62.9,1191.8] p=3.7×10⁻¹³（singular，rePCA 定位到三层 family 因子）；概率尺度：base 9.04%→0.77%（−8.3 点）、post-trained 34.18%→73.70%（+39.5 点）、交互差 47.8 点（registered 33.5），三域 48.5/48.9/42.5 全过预承诺 10 点阈值；15-checkpoint refit 41.5 点 Holm p=3×10⁻³¹（§4.2 Table 5、Abstract）。
5. `[正文]` 长度匹配控制把「引用提示」与「更长答案」分开：corrected pipeline +28.4 点 vs registered +12.6 点→提示是引用特异的（§3.3、Abstract；line ~1460 报 +28.35 vs +12.64）。
6. `[正文]` E3（Tülu-3-8B，registered scale-check arm，量化权重）：伪造相关信号在 answer tokens 上线性可解码、非题目难度；**不**从一般 truthfulness 迁移（注册主探针=null）；70B 主臂与 Llama-3.1-70B-Instruct 跨管线核对均未收集（§5.1）。
7. `[正文]` E4：只用表面引用特征训练（从不对「引用是否存在」训练）的偏好模型，14,400 Tülu-3 生成（7,200 post-trained）训练、3,600 held-out OLMo-2 base-vs-Instruct 对评估，跨族迁移；偏好 post-trained 输出 86%（Abstract；正文一臂 87.24% [86.19,88.29]，DeBERTa 领先 1.46 点）；固定引用数后→它奖励的是**数量**；作者明确口径：「E4 supports "quantity is rewarded," not "surface plausibility"」（§5.2 Table 7）。
8. `[正文]` 功效：12 checkpoints 上注册 E1 交互先验功效仅 73.0%；15 checkpoints 83.0%/83.5%；「neither clears the 80% gate at the Holm floor」（原文如此，referent 按印出引用）→ E1 null 是欠功效、非存在证据（§4.5）。
9. `[正文]` 总结论：「possibly, but not proven」——伪造引用**行为上**符合 reward hacking 预测，但注册的目标模式（obscure-real 处更陡）未出现，且朴素 instruction-following 可解释同一结果；证明 reward 导致伪造需要在训练中干预 reward，本研究未做（Abstract、§7）。

### 6.2 关键数字及来源（正文自报；复算PASS：12×900×4=43,200、47.8=39.5−(−8.3)）
**实验设计（控制组/随机化/闸）**：
- 预注册：OSF DOI 10.17605/OSF.IO/S53HY（ hypotheses、模型公式、多重比较校正、排除规则、每实验一个 null-result 发表标题）；GitHub https://github.com/zainnab-sparq/citation-reward-hacking（论文所印，本人未访问验证）。
- 校正：Holm-Bonferroni 在**实验族内**（每族 m=5 事先枚举），非全文跨族。
- 对照组：长度匹配条件（length_matched，要求等长多段但不提引用）分离 verbosity；abstention_permitted 条件；E1 族内 cascade（base/SFT/DPO/RLVR）作自身对照；E4 训练族与评估族不相交。
- 随机化/冻结：题集冻结+hash；替换用 seeded draw；生成 temperature 0、每 model×question×condition 一条（43,200 生成→42,845 进确认模型，剔除按预注册排除规则）；抽取 40,840 条引用、25,530 条生成含 ≥1 引用；验证走 Crossref/Semantic Scholar/OpenAlex/PubMed/CourtListener，法律走 CourtListener。
- 闸：预承诺 10 点交互阈值；κ≥0.7 标注一致性门（misgrounded 标签从未由 judge 发出而未绑定，作者自陈）；80% 功效门；两处修正（#10 钉数量、#12 及其子修正）。

### 6.3 对四机制的直接检验
- **NC**：E4 是 NC 的教科书式反例——「只数引用数量」的评分器被 post-trained 策略以 +39.5 点操纵，且 86%/87.24% 的偏好率说明代理分与真值（引用存在性）脱钩；rsi-agent 的确认闸分数若含任何「计数型表面特征」，就会复刻 E4。NC 需再加一条：评分特征不得在被评对象的生成能力可达空间内可单独优化（数量即此类）。
- **SP**：E2 的 47.8 点交互是「同一权重、只改提示」的因果杠杆 → rsi-agent 的密封私集若在评测时把「要引用/要来源」类提示暴露给被评策略，私集内容就从评分锚变成了可操纵线索；SP 必须管到提示层。
- **ESS**：E1 的族内 cascade + GLMM 随机效应 (1|family/model_id)+(1|question_id) 是 ESS 配对的范式——同族同题配对、聚类到 question；rsi-agent ESS 的配对单元应对齐到「question×family」，且把 stage_ordinal 当序贯暴露而非独立组。
- **HA**：该文的方法学 HA 值得全搬：冻结集 hash、修订 amendment 留原文件+可 diff、双 pipeline（registered vs corrected）并行报告、功率门、Holm 族内校正、null-result 标题预注册。rsi-agent HA 的哈希链应包含「分析计划 hash」，否则事后换 pipeline 不可追溯。

### 6.4 局限与作者自陈未做（`[正文]`搬运，§7/§8）
- 「This study does not identify the reward signal that produced the behavior… Concluding that RLHF specifically caused any of it requires an assumption the design does not test」（§7）。
- 「The length-matched control rules out verbosity, not compliance, and this design cannot separate the two. That separation needs an intervention on the reward itself」（§7）。
- §8.1 手标 ground truth 来自单一标注者（lead author），无第二标注者/无 κ；预注册的 κ≥0.7 门因 misgrounded 状态从未被 judge 发出而不绑定。
- §8.2 judge 无 CourtListener 访问权 → 法律引用判据弱于学术/医学；法律 REAL 57.49% vs 33.69%/27.19%、单次查找解决率 48.50% vs 31.83%/23.10%、单次法律判定 90.23% 返回 REAL。
- E3 欠收集（70B 主臂、跨管线核对未做）；era 分层面板大部分不可估计（明说而非当 null 读）；E1 null 欠功效（73%/83%）。

### 6.5 诚实缺口
- 摘要领读的 47.8 点是 corrected pipeline 值（registered 33.5），两 pipeline 并列报告但摘要只带大数；读者不看正文会漏口径。
- E4 的「86%」（Abstract）与正文一臂 87.24% [86.19,88.29] 的口径关系正文未在摘要层说明（注册臂 vs logistic 臂），引用时需核对 Table 7。
- known-real 格 REAL 判定率在前两 era 间 66.21%→54.90%（−11.31 点）= judge 未能确认而非模型多伪造——这一 judge 漂移发现位于敏感性分析（本人未定位其小节编号 `[未读全]`），提示 E 系列判定本身带仪器漂移，正文把它放在 appendix 层而非主结果层。

---

## 7. 横向综合

### 7.1 2610.11464 专项：锚定集一致性协议能否让 SP 从「私有」升级到「防共进化」？
**答案：能升到「防共进化-resistant」，不能升到「防共进化-proof」。** 论文协议=选择只读 10 项锚（唯一监督信号）+ 大无标签集共识 + locked set 与循环完全隔离 + 从不按 agent 分数选。它实际做到了四件事：①塌缩可测（Table 3 naive arm objective 1.000 vacuous vs anchored 0.865±0.002）；②漂移方向可读（held-out agreement）；③锚质量是承力变量（Report 封闭分类表 transfer 0.84 vs Spider 软锚 0.500±0.026）；④任务分认证被证伪（naive† 同分）。但其自身证据划出边界：4 项锚 std 0.063（单锚是抽奖）、5/9 runs 终点低于 mid-run peak（锚定向过拟合仍在发生）、且**从未测试 inner 循环读得到锚内容**的情形。真正的「防共进化」还需叠加：锚内容对 inner 不可读（P3 的 w/o-CE 纪律）、扰动双胞胎配对（P4 的 MaxxBench 修改）、以及锚集自身的去污染证书（P4 的 strict-match 探针）。

### 7.2 本批威胁最大的机制：**NC（非循环评分）**
四篇的攻击都不经过「循环」：P2 的 sufficiency 代理形式上完全非循环却被 padding 攻破（0.940 vs 真 0.667）；P3 证明**判据文本**是信息通道上的优化目标（score follows the criteria that were read：57.0→45.0 / 13.5% / 65.0%）；P6 证明同一权重下仅提示即可翻转伪造方向（47.8 点交互）且数量型表面评分为 86%/87.24% 偏好所奖励；P4 证明基准可能已在权重里（ERNIE HumanEval 96.3%）。NC 只断算术循环，而这四条路径分别是代理乐观、信息暴露、提示可 elicitation、上游污染——没有一条是环。次威胁是 SP（私有≠未被记住），ESS 与 HA 受方法学增益而非威胁（P1 的聚类 permutation、P5 的配对设计、P6 的预注册+Holm 族内校正都是现成加固件）。

### 7.3 加固建议（3 条，可执行）
1. **给 NC 加前置「模型无关审计」闸**（来自 2611.21538 §5/§9）：任何分数进入 outer 确认闸前，在 probe grid（引用集精确受控，n=60 量级）上算 selection error + signed gap + 饱和 pilot（组方差非零步比例）；selection error >0.15（论文无害带 0.05–0.132 vs 被黑 0.208）或 signed gap 为正即拒收或强制 precision 项；审计量与判定一起进 HA 链。
2. **密封判据文本 + instruction-faithful 重判**（来自 2611.08318 + 2610.11464）：inner 循环不可读确认闸 rubric/criteria（w/o-CE 臂仍 +23.2 pts，证明密封不伤改进）；终评加一道「只重写判据代码、不动指令」的 Cfaith 式重判，要求领先在该重判下存活（P3：51.6 vs 27.8 → 45.0 vs 47.2）；HA 增加暴露审计：判据字符串是否出现在 inner 工件（暴露库 ~1/3 条目引用标准代码）。
3. **SP 加去污染层 + ESS 加扰动双胞胎与聚类检验**（来自 2611.21315 + 2610.11464）：密封集启用前跑 strict-match 续写探针（题+选项+参考答案整体出现才计数，P4 口径）并把证书 hash 挂到密封集 hash 上；ESS 配对单元加入 MaxxBench 式修改双胞胎；推断统计一律任务/seed 聚类 sign-flip permutation（P1：池化 p=0.014 是假阳、聚类 p=0.12 才是真值），防相关判定批发显著性。

## 8. 证据索引（真实 URL）
- https://arxiv.org/abs/2610.11464 ｜ https://arxiv.org/html/2610.11464
- https://arxiv.org/abs/2611.21538 ｜ https://arxiv.org/html/2611.21538
- https://arxiv.org/abs/2611.08318 ｜ https://arxiv.org/html/2611.08318
- https://arxiv.org/abs/2611.21315 ｜ https://arxiv.org/html/2611.21315
- https://arxiv.org/abs/2610.10611 ｜ https://arxiv.org/html/2610.10611
- https://arxiv.org/abs/2611.01313 ｜ https://arxiv.org/html/2611.01313
- 论文所印（本人未访问验证）：https://doi.org/10.17605/OSF.IO/S53HY ｜ https://github.com/zainnab-sparq/citation-reward-hacking

## 9. 阅读完整性声明
- 6 篇全文文本均已落盘并逐篇读 Abstract/Methods/Results/Limitations 主链路；每篇标注 `[未读]` 处为本人未逐行核对的附录/全表（P1 Table 1/5 全行、P3 Fig 4 全文与方差估计、P4 App A/D 全表、P5 §3 证明与 App E/K/M/N、P6 敏感性小节编号与 Table 7 口径行）。
- 所有数字均可在 `<SCRATCH>/...` 按行号复核；本人复算仅限§0 列出的算术项，其余皆标「正文自报，本实验室未复算」。
- 未跑任何实验、未用 GPU、未 commit/push。
