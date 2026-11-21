# RSI（Recursive Self-Improvement）文献版图与分类学（2610.* 为主 + 2609.26457 锚点）

> 日期：2026-11-18｜子块：RSI 深入研究 → 文献版图与分类学｜方法：`deep-research-pro` 检索闭环 + AGENTS.md 铁律 6 验号
> 路径缩写（下文表格统一使用，避免超长路径撑爆表格）：
> - `READER` = `papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/worktree/untracked/analyses/agent-landscape-2026-10/readers/03-self-evolve.md`
> - `CAND` = 同目录 `additional-candidates.md`；`LAND` = 同目录 `report.md`；`DIGEST` = 同目录 `digest.md`
> - 所有 `Lxx` = `READER` 文件行号；数字均为**摘要内数字**（证据等级 B，未做 PDF/源码复算），每条可指回原文。

## 结论（第一段）

2610 批 RSI 不是一条线，而是**四条正交轴上的一次同时扩张**：更新目标（权重/记忆/技能/代码·harness/环境）/反馈信号（可执行 verifier/私有分/world model/物理反馈）/递归层级（单 agent/群体/双层）/评测设置（benchmark/审计协议/真实物理）。主干分类学已存在且稳定——2610.14625 的**四组九维**（artifact / updater+topology / mode+frequency+scope / feedback source+type+acceptance criterion，全文 S3.SS1 原文核对，[已核实](https://arxiv.org/abs/2610.14625)）。新工作几乎全部落在"更新目标"与"反馈与选择"这两组维度上，**"更新组织（topology）"维度出现分层细化**（群体 SwarmRSI/MetaShare/AReaL-RSI、双层 Harness²/SpecRSI/Evo²、研究 agent 自改写 2609.26457）。最值得注意的信号是 `LAND:112` 的判读被本批论文**证实**：领域从"提出自进化框架"推进到"自进化会退化、失败反馈未必定向"——CODA（skill-policy 共进化陷阱，抹掉 gains 的 52–93%）、TEB（no_evolve 靠 rescore 白涨 7.5/5.0/12.5pp）、2610.14662（145 条修改无 target-specific advantage）、EvoCowork/EmbodiedCLBench（改进不迁移/不随经验 scale）构成一组可证伪的反面结果。**缺口**：验证/审计类论文几乎都是"一次性协议" singletons，且全部关键数字只在摘要、无独立复现脚本（除 TimeRSI/EndoRSI/RSIFlow/MC-EvoBench 等在摘要给出 repo 链接）。

---

## 零、动手前先看这一节（digest 索引）

- `DIGEST:99` 起 `# ===== 03-self-evolve (45) =====`；`DIGEST:135` = 2610.14625（分类学主干）；`DIGEST:12` = 2610.18813；`DIGEST:33` = 2610.14495（NEG 区）；`DIGEST:47` = 2610.12086。
- `LAND:101-112` 方向 3「自进化 / 递归自我改进（RSI）：开始出现反面结果」列出 8 篇锚点，`LAND:112` 给出成熟信号判读。
- 本子块**不引用** 2610.19396（EpiGuard，传染病预警 agent，已验号但主题非 RSI），如实排除。

## 一、验号清单（AGENTS.md 铁律 6：号-题一致，全部 web_fetch abs 页 HTTP 200 + 标题逐字比对）

本报告引用的 **35 篇**全部核验通过，标 [已核实]。URL 统一为 `https://arxiv.org/abs/<id>`。

| arXiv id | 核验标题（abs 页 Title，逐字） | 状态 |
|---|---|---|
| 2610.14625 | From Existing Methods to New Research Ideas: Two Perspectives on Recursive Self-Improvement | [已核实](https://arxiv.org/abs/2610.14625) |
| 2609.26457 | Recursive self-improvement of AI research agents | [已核实](https://arxiv.org/abs/2609.26457) |
| 2610.22344 | MLExpert: Towards a Generalist Agentic Model for Autonomous Machine Learning Engineering | [已核实](https://arxiv.org/abs/2610.22344) |
| 2610.22192 | Terminal-RSI: Recursive Self-Improving Terminal Agents via Recursive Self-Rewrite and Policy-Refine | [已核实](https://arxiv.org/abs/2610.22192) |
| 2610.21887 | MetaShare: Failure-Aware Collective Self-Improvement for Open-Ended Agents | [已核实](https://arxiv.org/abs/2610.21887) |
| 2610.21791 | RSIFlow: Adaptive Self-Improvement Workflows with an Evolving Meta-Agent | [已核实](https://arxiv.org/abs/2610.21791) |
| 2610.21055 | Athanor: An Agent-Native System for Autonomous Post-Training Research | [已核实](https://arxiv.org/abs/2610.21055) |
| 2610.21040 | Recursive Reward Improvement via Adversarial Meta-Prompt Evolution | [已核实](https://arxiv.org/abs/2610.21040) |
| 2610.20845 | Evolving Together, Failing Apart: The Skill-Policy Co-evolution Trap in Self-Improving Agents | [已核实](https://arxiv.org/abs/2610.20845) |
| 2610.20809 | A Self-Improving Agent that Grows its Own DSL: Oracle-Free Operator Induction for Exact Question Answering | [已核实](https://arxiv.org/abs/2610.20809) |
| 2610.20665 | The World Won't Wait: Evaluating and Teaching Agents Under a Live Clock | [已核实](https://arxiv.org/abs/2610.20665) |
| 2610.20474 | AdaRSI: On-Demand Recursive Self-Improvement for Embodied Agents | [已核实](https://arxiv.org/abs/2610.20474) |
| 2610.20431 | SwarmRSI: Recursive Self-Improvement through Evolving Scientific Agent Swarm | [已核实](https://arxiv.org/abs/2610.20431) |
| 2610.20312 | Reflect After Failure: Brain-Inspired Self-Evolving Agents for Embodied Spatial Intelligence | [已核实](https://arxiv.org/abs/2610.20312) |
| 2610.20294 | MMLA-RSI: Recursive Self-Improvement through RTT and RTMA | [已核实](https://arxiv.org/abs/2610.20294) |
| 2610.19789 | More Methods, Better Agents? A Reality Check on Self-Distillation in Agentic RL | [已核实](https://arxiv.org/abs/2610.19789) |
| 2610.19725 | TimeRSI: Towards Self-Improved Agentic Deployment for Time-Series Foundation Models | [已核实](https://arxiv.org/abs/2610.19725) |
| 2610.19424 | VideoHarness-SRSI: Structured Recursive Self-Improvement for Video Understanding Harness | [已核实](https://arxiv.org/abs/2610.19424) |
| 2610.18842 | Who Improves the Robot? Toward Physical Recursive Self-Improvement | [已核实](https://arxiv.org/abs/2610.18842) |
| 2610.18083 | AReaL-RSI: Adaptive Scaling and Coordination of Recursive Self-Improvement | [已核实](https://arxiv.org/abs/2610.18083) |
| 2610.16491 | Drafting Agents, Not Just Tokens: Speculative Recursive Self-Improvement for Agentic Systems | [已核实](https://arxiv.org/abs/2610.16491) |
| 2610.15772 | RSIPO: Recursive Self-Improving Policy Optimization for Agentic Reinforcement Learning via Skill-Policy Co-Evolution | [已核实](https://arxiv.org/abs/2610.15772) |
| 2610.15055 | Trace Evolution Bench: Measuring Whether Agents Improve from Their Own Environment Traces | [已核实](https://arxiv.org/abs/2610.15055) |
| 2610.14843 | Pre-Registered Genesis Accounting for Recursive Self-Improvement: First Audited Operator-Genesis Events in a Physics-Discovery Runtime | [已核实](https://arxiv.org/abs/2610.14843) |
| 2610.14662 | Does Failure Feedback Direct Agent Self-Modification? | [已核实](https://arxiv.org/abs/2610.14662) |
| 2610.14224 | Harness^2: Recursive Agent Harnessing for an Open World | [已核实](https://arxiv.org/abs/2610.14224) |
| 2610.14163 | AutoEnvScaling: Automating the Data Flywheel with Terminal Agents | [已核实](https://arxiv.org/abs/2610.14163) |
| 2610.13912 | RSI-Anything: Unlocking Recursive Self-Improvement with World Models | [已核实](https://arxiv.org/abs/2610.13912) |
| 2610.13829 | MC-EvoBench: Auditing Experience-Driven Self-Improvement in Minecraft Agents | [已核实](https://arxiv.org/abs/2610.13829) |
| 2610.13376 | EndoRSI: Physical-Evidence-Driven Recursive Improvement of Endoscopic Robot Policies | [已核实](https://arxiv.org/abs/2610.13376) |
| 2610.13025 | Evo$^2$: Evolving the Evolver through Weight-Harness Co-Evolution | [已核实](https://arxiv.org/abs/2610.13025) |
| 2610.13021 | EmbodiedCLBench: Evaluating Continual Learning for Self-Evolving Embodied Agents | [已核实](https://arxiv.org/abs/2610.13021) |
| 2610.12369 | Embodied Turing Machines: Stateful Code for Robot Recursive Self-Improvement | [已核实](https://arxiv.org/abs/2610.12369) |
| 2610.21458 | What Survives Self-Evolution: Provenance Marks in Copied Agent Harnesses | [已核实](https://arxiv.org/abs/2610.21458) |
| 2610.19396 | EpiGuard: An Autonomous Infectious-Disease Early Warning Agent Platform | [已核实](https://arxiv.org/abs/2610.19396)（主题非 RSI，排除） |

验号方法（verifier，AGENTS.local §9）：abs 页返回 `Title:` 字段与 `READER`/`LAND` 落盘标题逐字比对；一致即 [已核实]，不一致/超时标"未核实"。本批无"未核实"项。2609.26457 由父代理 lead 先行核实（`m00101`），我独立 web_fetch 二次确认（2026-09-22，cs.AI，作者 Dhruv Srikanth/Bingchen Zhao/Dixing Xu/Yuxiang Wu/Zhengyao Jiang，与 Weco AI 博客作者一致），满足"同一事实最多两种独立方式验证"。

## 二、RSI 研究版图：四轴可执行分类表

### 2.1 按更新目标（Update target）

| id | 更新目标 | 一句话机制 | 关键数字（指回 `READER` 行号） |
|---|---|---|---|
| [2610.13025](https://arxiv.org/abs/2610.13025) | 权重 + harness 共进化 | 单一策略兼 Proposer/Reflector，EvoTIDE = SFT + 交错 GRPO，evolver 本身被进化 | Polyglot 82.1% vs 62.6%；SWE-bench Verified 48.3% vs 41.0%；GPT-5-mini 24.4%→48.3%（L216） |
| [2610.20474](https://arxiv.org/abs/2610.20474) | 执行器权重（按需） | coding agent 先改技能/harness，再对持久化失败从保存状态重试归因，只微调仍不可靠的子步 | RoboCasa365 50 任务、GR00T N1.5、无人工 demo；未见复合任务 4.6%→13.8%；仅 28% GPU time / 22% demos（L41） |
| [2610.20294](https://arxiv.org/abs/2610.20294) | 策略态 Φ + 权威记忆 M | RTT 改 Φ、RTMA 改 M，且各条"改进方法本身"也是改进对象（current updater/manager 训练其后继） | 4-candidate 保留 LARC 反馈降 expected error 24.65 / 36.65pp；递归链 benefit 仍待评估（L56） |
| [2610.15772](https://arxiv.org/abs/2610.15772) | 技能 + 策略 | policy 从已完成任务抽 skill 并经 trajectory replay 显式优化技能生成力；经验 Bayes shrinkage 动态校准 teacher 信任 | Qwen2.5-7B ALFWorld 成功率 98.6%，绝对高于次优 2.5%、GRPO 17.4%（L141） |
| [2610.20845](https://arxiv.org/abs/2610.20845) | 技能 ↔ 策略（共进化，反面） | 命名 skill-policy co-evolution trap：cooperative acceptance 把 policy 绑死在 co-evolved bank，形成结构性盲点 | 4 共进化方法 × 5 benchmark（6 settings）；worst-case 比 peak 低 5–13 点，抹掉 delivered bank gain 的 52–93%（L31） |
| [2610.14224](https://arxiv.org/abs/2610.14224) | harness（双层） | task-level + harnessing-level 双层递归，无 ground-truth verifier | +11.5 WorkBuddy-Bench / +4.5 JobBench / +11.8pp LAB（L186） |
| [2610.16491](https://arxiv.org/abs/2610.16491) | 后继 agent 草稿（代码） | SpecRSI：前代未评完就起草可执行后继，采纳仍需完整验证；VATS 分配预算 + GVO 从已完成搜索学 improver | terminal gain 超 Janus 33.3–40.0%；50/50 backbone-task pair 全胜；coding-validation 69.4% vs HGM 64.3%（目标成本低 20.8%）（L121） |
| [2610.19424](https://arxiv.org/abs/2610.19424) | harness（结构化搜索） | 三失败模式（validation overfitting / search-budget dilution / early-experience trap）同源于"同一份不完美验证证据既驱动改写又驱动搜索策略" | 分析 >4,000 候选；+6.2% held-out；val-test gap -37.9%（L76） |
| [2609.26457](https://arxiv.org/abs/2609.26457) | 研究 agent 自身代码 | 被接受的 rewrite 成为下一轮被编辑的 agent（agent code = 优化对象）；双层 RSI | AIDE0→AIDE99 共 100 步、8 天、9/10 提案被拒；KernelBench reward-hack 63%→34%（Weco 博客 [URL](https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement)，2026-07-14；论文 abs 页 untrusted 摘要未列这些数，故标 B 级外部来源） |
| [2610.14163](https://arxiv.org/abs/2610.14163) | RL 环境（数据飞轮） | proposer 自动建 RL 环境 + solver 训练，环境本身成更新目标 | valid-env rate 25×、cost -75%；Qwen3.5-35B-A3B Terminal-Bench2.1 +4.3；Qwen3.6-35B-A3B 双角色 +12.2、Terminal-Bench4.0 1.6→3.2（L191） |
| [2610.13912](https://arxiv.org/abs/2610.13912) | 环境/动作（世界模型引导） | JEPA-Anything 世界模型引导预测式选择与适配 | code repair 26.3%(static verifier)→29.2%(predictive)→31.2%(world-model)；GUI actions -39.4%；lab-planning 21.80%→46.01%（5 版）；reaction yield 96.15% vs Bayesian opt 82.16%（同 40 实验预算）（L196） |

### 2.2 按反馈信号（Feedback source / type）

| id | 反馈信号 | 一句话机制 | 关键数字（`READER` 行号） |
|---|---|---|---|
| [2610.20809](https://arxiv.org/abs/2610.20809) | 自洽性 + 守恒律（无 reward/judge/answer key） | agent 自己生长 DSL，内循环自生答案集 ≈3×，用一致性约束归纳精确算子 | 拒绝 138/138 buggy operators；business-ledger >10× / clinical 近 20×；inner loop 答案集 ≈3 倍（L36） |
| [2610.21040](https://arxiv.org/abs/2610.21040) | 对抗 rubric（reward-hacking 对手审 rubric） | RRI：proposer 写训练任务，adversary 攻击评分 rubric，迭代改进 reward 本身 | HealthBench +10.9(9B)/+12.4(27B)/+10.3(无外模型)；PRBench Hard +6.6、ProfBench +8.0（L26） |
| [2609.26457](https://arxiv.org/abs/2609.26457) | 私有分（private score 生存选择） | 每步用 private score 决定提案去留；9/10 提案被拒 | reward-hack 63%→34%（Weco 博客 URL，见 2.1） |
| [2610.13912](https://arxiv.org/abs/2610.13912) | world model（预测式选择） | 以预测模型替代静态 verifier 做候选筛选 | 见 2.1（L196） |
| [2610.13376](https://arxiv.org/abs/2610.13376) | 物理证据 | 内窥镜机器人以物理证据驱动策略递归改进 | 插入成功 2/21(gen1)→17/21(gen4)；6 个离体猪标本上最佳自主策略（L206） |
| [2610.18842](https://arxiv.org/abs/2610.18842) | 物理反馈（综述/框架） | L0–L5 框架区分 capability-level vs meta-level、within-task vs across-task，L5 留给独立验证的递归增益 | 结论： surveyed literature **尚未**证明 Physical RSI（repo: github.com/SAIL-Research-Lab/Physical-RSI）（L91） |
| [2610.14662](https://arxiv.org/abs/2610.14662) | 失败反馈（质疑其定向性） | 直接检验失败反馈是否真能定向指导 self-modification | 145 条 DGM 日志修改无 target-specific advantage；Qwen3-4B +20pp(target feedback)、Phi-4-mini +7pp 不显著（L176） |

### 2.3 按递归层级（Topology）

| id | 层级 | 一句话机制 | 关键数字（`READER` 行号） |
|---|---|---|---|
| [2610.20431](https://arxiv.org/abs/2610.20431) | 群体（swarm） | Feedback-Guided Swarm Mutation + Complementary Swarm Recombination，跨 agent 复用保留假设与互补组件 | OpenProblems/NatureBench/MLE-Bench Lite SOTA；累积经验提升 held-out（L46） |
| [2610.21887](https://arxiv.org/abs/2610.21887) | 群体（failure-aware） | 记录成功/失败修改 + signed performance deltas 供群体共享 | median ≥ DGM-Hyperagents；少 1.7–2.9× evolution iterations（L11） |
| [2610.18083](https://arxiv.org/abs/2610.18083) | 群体（持久研究团队） | leader agent 选方向 + conditional memory sharing + campaign 后 evidence-based review；**不更新 backbone 权重** | Terminal Bench +2.2pp/backbone；KernelBench speedup +32.2–36.8%；经验共享+专门角色比独立探索低 1.9–27.5% kernel latency（L106） |
| [2610.14224](https://arxiv.org/abs/2610.14224) | 双层 | task-level + harnessing-level（见 2.1） | 见 2.1（L186） |
| [2610.16491](https://arxiv.org/abs/2610.16491) | 双层（improver/evaluator 分离） | 起草器搜索、真实 evaluator 固定（见 2.1） | 见 2.1（L121） |
| [2610.13025](https://arxiv.org/abs/2610.13025) | 双层（权重↔harness） | evolver 本身被进化（见 2.1） | 见 2.1（L216） |
| [2609.26457](https://arxiv.org/abs/2609.26457) | 双层（研究 agent 自改写） | 每层 rewrite 改变下一层 solver（见 2.1） | 见 2.1（Weco 博客 URL） |

### 2.4 按评测设置（offline / online / 审计协议）

| id | 评测设置 | 一句话机制 | 关键数字（`READER`/`CAND` 行号） |
|---|---|---|---|
| [2610.15055](https://arxiv.org/abs/2610.15055) | 审计协议（rescore vs 真更新分离） | TEB 把 frozen rescore 与 context/weight update 分开，三套冻结 80/20/40 packs | DeepSeek v4 Pro no_evolve **不改策略却涨** 7.5/5.0/12.5pp（3 seeds）；9 个 text-update cells 7 个正 excess 但 GLM/MiniMax 不复现；一个 gepa_llm seed -12.5（regression 0.48）（L153/L156） |
| [2610.13829](https://arxiv.org/abs/2610.13829) | 审计协议（paired probe） | Minecraft 19 evolution chains（12 主 + 7 probe）、3 paired probe families、5 seeds；TSR/WGP/VMP | related experience **无稳定 transfer**（区间含 0）；repo: github.com/xzw-ustc/mc-bench-open-source（L201） |
| [2610.13021](https://arxiv.org/abs/2610.13021) | 审计协议（持续学习） | 首个具身持续学习 benchmark | 经验有用但**不随样本数 scale**，agent 无法从额外经验提取信息（L221） |
| [2610.14843](https://arxiv.org/abs/2610.14843) | 审计协议（预注册 genesis accounting） | M1 数设计词汇内内容实例、M2 数机器自创机制类型（新算子 + held-out 证书） | 氢 Balmer：sq(inv(n))=1/n² held-out R² -0.546(best standard)→1.0000000(composed)；n=8,9 外推相对误差 ~10⁻⁶；Kepler sqrt(inv(a)) 从 JPL 恢复；compounding 1.109×；blind exam honest tally = 0（L161） |
| [2610.21458](https://arxiv.org/abs/2610.21458) | 审计协议（溯源标记） | KGW provenance marks 在 copied harness 中经 GEPA 改写后是否存活 | 12 份拷贝存活 0 份（covert rewriting attack）（`CAND:18`） |
| [2610.19789](https://arxiv.org/abs/2610.19789) | reality check（成本审计） | 10 个自蒸馏方法 vs GRPO，3 模型规模，8 rollout seed 重复评测 | 132 runs / 12,124.4 H800 GPU-hours（`CAND:48`、`CAND:85`） |
| [2610.19725](https://arxiv.org/abs/2610.19725) | offline→on（时序部署） | 冻结预测模型外挂 coding agent，历史窗口试错 + 部署期 adaptive selector | 198 任务（TIME + fev-bench），6 个点/概率预测指标全面 SOTA；repo: github.com/haoxin1998/TimeRSI（L71） |
| [2610.20665](https://arxiv.org/abs/2610.20665) | online（live clock） | RealtimeBench 让游戏在推理时继续跑；TORSI 慢 teacher 在暂停态写指令给快 student | 180 任务/6 游戏；Jev 平均 13.51→24.19（0–100），2 轮、六类全涨（L61） |
| [2610.22344](https://arxiv.org/abs/2610.22344) | offline（自主 ML 工程） | 35B MoE + Kaggle-MLE Factory + WIRL | MLE-Bench Lite 奖牌率 37.9%→72.7%（L6） |
| [2610.21055](https://arxiv.org/abs/2610.21055) | offline（后训练研究 agent） | 4 个 Claude Opus 5.5 agent 各 8-GPU，104 小时自主后训练研究 | Qwen3-4B-Base dev 15.9→67.3 / held-out 9.2→48.2；官方 Instruct 66.5/47.9（L21） |

> 数字口径纪律：`LAND:112` 的"开始有可证伪性"是**判读（B→A-派生）**，不是实测；表中每个百分数都是摘要原文，未跨级并列当作同口径效果量比较（沿用 `notes/2026-11-14-arxiv最新500篇研究方向扫描.md` 骨架的跨级数字禁令）。

## 三、分类学骨架：2610.14625 九维主干 + 新工作落点

### 3.1 九维原文（`https://arxiv.org/html/2610.14625v1` S3.SS1 "Overview of the unified taxonomy"，逐字）

论文把统一分类学分成**四组九维**：

1. **Update targets**：RSI artifact.
2. **Update organization**：Artifact updater and RSI topology.
3. **Update schedule and scope**：RSI mode, Update frequency, and Experience scope.
4. **Feedback and selection**：Feedback source, Feedback type, and Acceptance criterion.

第二视角把 RSI 接到传统方法：optimization / backpropagation / RL / meta-learning / distillation / regularization / data augmentation / continual learning / pruning / ensemble / adversarial learning + 进化算法；评测分 online / offline / offline-to-online。资源页 `https://prism-shadow.github.io/awesome-rsi/`。

### 3.2 本批新工作落点（每篇落在哪几维）

| id | artifact | updater + topology | mode/frequency/scope | feedback source/type | acceptance criterion |
|---|---|---|---|---|---|
| 2610.13025 Evo² | 权重+harness | Proposer/Reflector 同策略；双层共进化 | offline, 交错迭代 | benchmark verifier | SFT+GRPO 后 held-out |
| 2610.20474 AdaRSI | 执行器权重 | coding agent；按需（on-demand） | within/across-task, 失败触发 | 保存状态重试归因（物理） | 重试后仍不可靠才微调 |
| 2610.20294 MMLA-RSI | 策略态 Φ / 权威记忆 M | current updater/manager 训练其后继；双层递归 | reasoning-time, 逐代 | LARC + 独立 evaluator | held-out 证书 / 条件性质 |
| 2610.15772 RSIPO | 技能 + 策略 | policy 自抽 skill + replay；单 agent | on-policy, 训练中 | RLVR + teacher score（Bayes 校准） | verifiable reward |
| 2610.20845 CODA | skill ↔ policy | 共进化；cooperative acceptance | 多 benchmark 在线迭代 | 共进化 bank 反馈 | 合作接受准则（→陷阱） |
| 2610.14224 Harness² | harness | task-level + harnessing-level；双层 | open-world, 递归 | 无 GT verifier（自生成） | 下游任务增益 |
| 2610.16491 SpecRSI | 后继 agent 草稿 | improver + 固定 evaluator；双层 | 未评完即起草；VATS 预算 | 完整验证 | verified return |
| 2610.19424 VideoHarness-SRSI | harness | 冻结 VLM + 结构化搜索；单 agent | 4,000 候选，隔离探针分支 | content-free 可迁移证据 | held-out + val-test gap |
| 2609.26457 AIDE² | 研究 agent 自身代码 | 每层 rewrite 改下一层；双层 | 100 步, 8 天, offline→online | private score | 生存选择（9/10 拒） |
| 2610.14163 AutoEnvScaling | RL 环境 | proposer + solver | 数据飞轮, 自动生成 | 环境可执行性 verifier | valid-env rate / TB 增益 |
| 2610.13912 RSI-Anything | 环境/动作 | world model 引导；单 agent | 5 版实验室迭代 | JEPA-Anything 预测 | world-model adaptation |
| 2610.20431 SwarmRSI | 假设+程序组件 | swarm mutation/recombination；群体 | 跨 agent 累积经验 | 其他 agent 实验反馈 | held-out + 生物学家复现 |
| 2610.21887 MetaShare | 修改记录+deltas | 群体共享 signed deltas | 少 1.7–2.9× 迭代 | 成功/失败修改 | median ≥ 基线 |
| 2610.18083 AReaL-RSI | 记忆/技能/角色 | leader + 条件共享；群体 | campaign 后 review | evidence-based review | TB/KernelBench 增益 |
| 2610.15055 TEB | text/weight update | 审计协议（rescore 分离） | offline, 3 seeds | 隐藏单测 | validation 不降 + 超 seed band |
| 2610.14843 Genesis | 机器自创算子（M2） | 预注册两层记账 | 预注册, held-out 证书 | NIST/JPL 真实数据 | held-out R² + 独立标签协议 |

**判读**：新工作对九维的覆盖不均衡——`artifact` 与 `feedback source/type` 两列几乎每行都填；`mode/frequency/scope` 与 `acceptance criterion` 只有审计类论文（TEB、Genesis、CODA）在**显式**定义；这正是 2610.14625 分类学呼吁但社区尚未补齐的部分。

## 四、缺口清单

### 4.1 只有一篇（singleton）的方向
- **私有分生存选择**：仅 2609.26457（AIDE²），且关键数在 Weco 博客而非论文摘要 → 单一来源、B 级。
- **机器自创机制的类型级审计（M2 genesis）**：仅 2610.14843；作者自陈"demonstrate capability to author, not yet autonomous genesis"（L161）。
- **KGW 溯源标记在自进化中的存活**：仅 2610.21458（12 拷贝 0 存活，`CAND:18`）。
- **oracle-free 自生 DSL**：仅 2610.20809（L36）。
- **对抗 rubric 改进 reward 本身**：仅 2610.21040（L26）。
- **live-clock 实时评测**：仅 2610.20665（L61）。
- **Physical RSI 综述框架**：仅 2610.18842，其结论恰是"文献尚未证明 Physical RSI"（L91）。

### 4.2 互相冲突的结论（同一事实两种口径，如实并列，不开第三条路）
- **经验/技能共进化到底涨不涨**：2610.15772 RSIPO 报 ALFWorld 98.6%、+17.4% vs GRPO（L141）；2610.20845 CODA 报同一类 skill-policy 共进化 worst-case 比 peak 低 5–13 点、抹掉 52–93% gains（L31）。差异可能来自"peak vs worst-case"口径不同（CODA 用 worst-case，RSIPO 用平均），**属口径冲突而非直接矛盾**，需同基准复核。
- **经验是否迁移**：2610.13829 MC-EvoBench 报 related experience 无稳定 transfer（L201）、2610.13021 EmbodiedCLBench 报经验不随样本数 scale（L221） vs 2610.21887 MetaShare 报共享成功/失败修改能减 1.7–2.9× 迭代（L11）、2610.20431 SwarmRSI 报累积经验提升 held-out（L46）。冲突点在于**共享的是"经验痕迹"还是"可执行组件"**——后者迁移、前者不稳。
- **text update 是否真带来更新**：2610.15055 TEB 报 9 个 text-update cells 中 7 个正 excess，但 no_evolve 也能靠 rescore 涨 7.5/5.0/12.5pp，GLM/MiniMax 不复现 7/9 模式（L156）；2610.14662 报 145 条修改无 target-specific advantage（L176）。两者都指向"失败反馈未必定向"，与 `LAND:112` 判读一致。

### 4.3 数字只在摘要、无复现脚本（证据等级 B）
- 全部关键数（2.1–2.4 表）均摘自 `READER` 摘要，我**未**打开 PDF/源码复算；披露 repo 链接的仅：2610.21791（RSIFlow, github.com/1HUAN1/RSI-Flow, L16）、2610.19725（TimeRSI, L71）、2610.13829（MC-EvoBench, L201）、2610.18842（Physical-RSI, L91）、2610.13098 类（VETR 未引用）。其余（含 CODA 52–93%、TEB 7.5/5.0/12.5pp、Evo² 82.1%、RSIPO 98.6%、AIDE² 100 步/63%→34%）**无复现脚本落盘**。
- 2609.26457 的 100 步 / 9/10 拒 / reward-hack 63%→34% 来自 Weco 博客（外部 URL），abs 页摘要被 token-diet 截断未逐字显示这些数 → 标 B 级外部来源，父 lead 已核实、我二次确认作者与题名，数字本身未独立复算。
- 2610.19789 的 132 runs / 12,124.4 H800 GPU-hours 只出现在 `CAND:48/85`（候选清单），未见于 `READER` 摘要 → 单一落盘来源，标 B。

## 五、证据等级与未核清单

- **A（直引原文 + 验号）**：九维分类（arXiv HTML S3.SS1 原文）；35 篇 id-题一致（abs HTTP 200）；`LAND:101-112` 方向 3 原文。
- **A-派生**：`LAND:112`「领域开始有可证伪性」判读；本报告的"四轴扩张""落点不均衡"判断。
- **B（摘要内数字 + 验号，未复算）**：第二节全部关键数字；第四节冲突分析。
- **B（外部 URL）**：2609.26457 的 AIDE² 运行数字（Weco 博客）。
- **未核**：无。本报告引用的 35 篇 id 全部 [已核实]；`READER` 中 2610.18868/18792/18097/17718/17114/16078/15964/15782/15708/15244/14828/14730/13098/14495/12086/18813 等未在正文引用，故未逐一验号（按"引用才验"纪律）。
- **查过但未纳入**：`DIGEST:33` 的 2610.14495（NEG 区）、`DIGEST:47` 的 2610.12086、`DIGEST:12` 的 2610.18813 在候选/摘要中出现但主题或数字不足以进主表，未逐条展开（如需可续补）。
