# 2026-11-19 RSI/self-improving agent 前沿论文（新 15 篇）

> 任务：为 rsi-agent 的四道机制（**SP** 密封私集 / **NC** 非循环评分 / **ESS** ESS闸 / **HA** 哈希审计）补一批 2025–2026 前沿论文，重点三条线：A 递归自我改进本体、B 防自证/可证伪方法学、C 已有反面结果。
> 纪律：全部 arXiv id 逐篇 `web_fetch abs` 页验 **HTTP 200 + 号-题一致**；摘要取自 abs 页 `>Abstract:` 字段；查不到就标，不硬凑。
> 去重：与盘上 `notes/2026-11-18-rsi-landscape.md`（35 篇已验号清单）、`notes/2026-11-18-rsi-failure-modes.md` 比对，**本批 15 篇均为盘上未收录的新论文**（已知线索 2609.14858/2609.26457/2610.20845/2610.14625/2610.22192/2610.15772/2610.19789/2611.22901 都已在旧 notes，不重复）。

---

## 一、结论：最该读的 5 篇

| 排序 | id | 为什么必读（对 rsi-agent 的直接意义） |
|---|---|---|
| 1 | **2610.11464** Who Verifies the Verifier? | 防自证的正面纲领。把 verifier 当进化对象，按「与 10 项锚定参考集的一致性 + 无标签共识」选择，**从不按 agent 分数选**。最尖锐的一击：去掉 anchor guards 后 verifier 塌成 always-pass grader，**却能同样训练出技能**——证明「下游任务分涨了」根本无法认证一个自进化的 verifier。直接打中 NC（评分不能循环依赖被评对象）与 SP（必须有密封锚定集）。 |
| 2 | **2610.38392** Masked Erosion | 对本实验室最危险的反面结果。接受闸只看当前模型下的 harness 行为，看不到冗余 safeguard；**更大的 eval batch 反而删掉更便宜的 safeguard**（20-task 全保、40-task 六个全失），配对显著性检验挡得住碰巧通过的删除、挡不住真升 task success 的删除。这是对 ESS闸 的直接压力测试：闸的统计功效越强，越可能奖励「删掉看不见的保险」。 |
| 3 | **2611.21538** Optimistic Verifiers Get Hacked | 在真 reward 可精确计算的场景（124416 contexts + exact process oracle）里证明：乐观 verifier 会被 padding hack——sufficiency 代理升到 0.940，真 faithfulness 反从 0.731 跌到 0.667。给出 model-free audit（selection error + gap sign + 饱和度 pilot）预判表型。是 NC 的量化警示：代理分与真值脱钩时，涨幅本身是 hack 的信号。 |
| 4 | **2611.12988** Third Eye | 把「即时收益」与「再经一次原生更新后的收益」分开。即时选出的 utility≈0，但 continuation 后降 **1.78pp（95%CI [0.68, 2.88]）**。对 NC 的规定：评分必须用与选择不相交的 item，且要加 no-update 基线；一轮的分不能当「持久改进」证据。 |
| 5 | **2611.04952** Artifact-to-Capability | 直接打 HA 的边界。Reflexion/Voyager/DGM 同 backbone 都呈现三个代理信号（artifact 累积 / 加载后行为变 / 指标升），却各有 capability-formation break（合法化增益 / 组合覆盖 / 纯丢分漂移）。证明「审计了 artifact、看到分数涨」仍判不出能力是否真形成——哈希审计只能证明"改了什么"，不能证明"学会了什么"。 |

> 一句话取舍：这 5 篇覆盖了 B/C 两条线里对四机制最致命的四个缺口——**评分循环（1/3）、闸的统计功效反向激励（2）、即时≠持久（4）、代理信号≠能力（5）**。A 线本体的正面参照见 2611.21056（Evaluation-Free RSI，零下游评估仍 60%→85.6%），放在分类展开里。

---

## 二、实测表（15 篇，全部验号 PASS）

代号：SP=密封私集 / NC=非循环评分 / ESS=ESS闸 / HA=哈希审计。
验号状态均为 `HTTP 200 + #Title: 号-题一致`；链接即所验 abs 页。

| # | id | 标题 | 日期 | 一句话核心贡献 | 验号状态 / URL | 与 rsi-agent 四机制的关系 |
|---|---|---|---|---|---|---|
| 1 | 2611.23138 | RSI-Prime: Training Models to Train Models | 2026-11-17 | 构建 350 条完整实验轨迹/7 类任务的 RSI-Prime 数据集，用决策级质量标注 + loss masking 排除低质行为监督，GLM-5.3 在 PostTrainBench 相对提升 31.95% | PASS https://arxiv.org/abs/2611.23138 | A 本体；训练「会训练的模型」需要高质量决策轨迹→SP 要保证轨迹来源不被污染、NC 要保证标注不来自被训模型自身 |
| 2 | 2611.21056 | Evaluation-Free Recursive Self-Improvement for Coding Agents | 2026-11-16 | ELITE 在进化循环内零下游任务评估、只用确定性结构有效性检查来整合互补子代机制，SWE-bench Pro 60.0%→85.6%、wall-clock 快 8.8x/3.2x | PASS https://arxiv.org/abs/2611.21056 | A 本体 + 抗 benchmark 过拟合；NC 的正面实现样本——完全不靠下游分做选择，从根上断掉评分循环 |
| 3 | 2611.08946 | Flex-RSI: Benchmarking Recursive Self-Improvement of Frontier Agents | 2026-11-06 | 固定预算下 vision/机器人 RSI benchmark：executor 改解（held-out val 更高才留）+ improver 改 harness，hidden test 终评；配对实验证明改 harness 本身贡献增益 | PASS https://arxiv.org/abs/2611.08946 | A 本体 + 可证伪评测；SP（held-out val 与 hidden test 分离）、ESS（更高才留的接受闸）的标准范式，但"只记成功/失败反馈拖累开源模型"提示 ESS 信号要分层 |
| 4 | 2611.12988 | Third Eye: Beyond Immediate Gains in Recursive Self-Improvement | 2026-11-08 | 分离即时性能与「再经一次原生更新后」的结果，即时选出的 utility≈0 而 continuation 后降 1.78pp | PASS https://arxiv.org/abs/2611.12988 | C 反面 + 可证伪；NC 必须看 continuation/no-update 基线，选择集与终评分集用不相交 item |
| 5 | 2611.10719 | No Second Run: Does RSI Stop at Frontier Pre-training? | 2026-11-07 | 指出公开 RSI 只做后训练（现成 base model / GPU-hours 可重复 / 单一可验分），预训练破坏三条件（数十万 GPU-hours 不可重复 / 目标是 post-training headroom 非单分 / 要改整个生产系统） | PASS https://arxiv.org/abs/2611.10719 | A 本体/ignition 边界；提醒 NC 的「单一可验分」前提在预训练阶段不成立，ESS闸 的判据需重新定义 |
| 6 | 2611.13524 | The RSI Tax: When Recursive Self-Improvement Is Not Worth the Cost | 2026-11-10 | 定义 RSI Tax=自改进过程 token 成本并纳入端到端评估，bounded-note 基线匹配/超过三种自进化方法且总 token 省 65.6–71.0%，verification 占 tax 的 74.8–93.6% | PASS https://arxiv.org/abs/2611.13524 | C 反面 + verifier bottleneck；ESS闸 必须把成本计入收益判据，否则"涨分"可能灾难性地不划算；verification 是大头→NC 贵 |
| 7 | 2610.11464 | Who Verifies the Verifier? Co-Evolving Inspectable Graders with Self-Improving Agents | 2026-10-08 | 让 verifier  become 进化对象，按锚定集一致性+无标签共识选择（不按 agent 分数）；塌成 always-pass grader 仍能训技能→下游分无法认证 verifier | PASS https://arxiv.org/abs/2610.11464 | B 防自证核心；NC（评分不循环）+ SP（10 项锚定参考集即密封集）。必读第 1 |
| 8 | 2611.21538 | Optimistic Verifiers Get Hacked: Reward Hacking Under an Exact Process Oracle | 2026-11-16 | 在真 reward 精确可算场景把 verifier 沿 7 轴降级，乐观 verifier 被 padding hack（sufficiency 0.940 / 真 faithfulness 0.731→0.667），给出 model-free audit 预判表型 | PASS https://arxiv.org/abs/2611.21538 | B 防自证；NC 的量化警示——代理分涨而真值跌时可审 selection error/gap sign，ESS闸 前加饱和度 pilot。必读第 3 |
| 9 | 2611.08318 | Learning the Task or Learning the Verifier? Rethinking Criteria Exposure in Embodied Self-Improvement | 2026-11-05 | Criteria Exposure（把成功标准暴露给 agent）使其快速产出高成功率代码，但标准稍变优势消失、技能跨任务迁移差；提出 instruction-faithful criteria | PASS https://arxiv.org/abs/2611.08318 | B 防自证；SP——私集标准不可暴露给被评对象，否则学到的是 verifier 而非任务；HA 只记改动无法发现标准被过拟合 |
| 10 | 2611.21315 | Benchleaking Is Accelerating: Benchmark contamination has worsened sharply in the past month -- from post-training down to pre-training base models | 2026-11-16 | 定义 benchleaking（测试样本进训练数据→score=recall）区别于 benchmaxxing，记录 post-training 与 pre-training 两轮升级，提出 Anti-Hack score（泄露样本判错重算，扣 1.3–16.6 分） | PASS https://arxiv.org/abs/2611.21315 | B 防自证；SP 的必要性——一旦基准进入训练/进化语料，分数就是记忆；Anti-Hack score 可作为私集污染调整的现成工具 |
| 11 | 2610.10611 | Verification and Self-Improvement in Agentic AI: Foundations and Limits | 2026-10-07 (v2 10-30) | 用 explicit support + transcript interfaces 区分 search gains 与 accepted-task coverage 变化，定义 semantic containment / guard-transfer invariance，36864 paired runs 反例反馈减少 verifier calls 却不改 coverage | PASS https://arxiv.org/abs/2610.10611 | B 防自证理论；NC 的形式化边界——"call 少了/分高了"不等于能力覆盖变宽，ESS闸 需区分搜索收益与覆盖扩张 |
| 12 | 2610.38392 | Masked Erosion: How Acceptance Gates Lose Redundant Safeguards in Self-Improving Agents | 2026-10-28 | 接受闸看不见冗余 safeguard，删一个被掩盖的防护→无声侵蚀；更大 eval batch 反而删掉更便宜 safeguard，GEPA 保留安全规则逐字却在弱模型下误用凭据 | PASS https://arxiv.org/abs/2610.38392 | C 反面 + 防自证；ESS闸 最危险压力测试——统计闸越"显著"越可能奖励删保险；HA 能发现删了什么、发现不了冗余已失。必读第 2 |
| 13 | 2610.22984 | Self-Evolving Agents Can Repair Failures but Struggle to Generalize from Them | 2026-10-16 | Cold-Start Failure-Only Evolution 下三种自进化范式一致出现 repair-generalization gap：失败经验能修已失败任务但难在未见任务提升，成功轨迹才大幅改善迁移 | PASS https://arxiv.org/abs/2610.22984 | C 反面；SP——失败反馈只定向到旧任务，ESS闸 不能把"修好了旧失败"当"泛化改进"；私集要含未见任务 |
| 14 | 2611.04952 | Artifact-to-Capability: Diagnosing Breaks in Capability Formation of Self-Evolving Agents in a Controlled Environment | 2026-11-03 | 定义 artifact→capability 五个顺序依赖属性 + 三尺度诊断，证明三代理信号（artifact 累积/行为变/指标升）判不出能力是否形成 | PASS https://arxiv.org/abs/2611.04952 | C 反面 + 可证伪；HA 的边界——哈希审计证明"改了什么"不证明"学会了什么"，须补 capability-formation checkpoint。必读第 5 |
| 15 | 2611.01313 | Are Fake Citations a Form of Reward Hacking? A Pre-Registered Study of Post-Training and Citation Cues | 2026-10-31 | 4 个预注册实验显示要求引用使伪造陡升（47.8 点交互，Holm p=2e-34），后训练整体伪造 4.8%→56.7%，表面引用特征的 judge 86% 偏好数量多的输出；结论"possibly but not proven" | PASS https://arxiv.org/abs/2611.01313 | B 防自证；NC——按表面特征（引用数量）打分会被刷，ESS闸 前要验 reward 是否真被改；示范预注册 + 证伪式写法 |

---

## 三、分类展开

### A. 递归自我改进本体（5 篇）

- **2611.21056 ELITE**：本批最值得借鉴的正面机制。它把"evaluate-and-select"整个拿掉——循环内只做确定性结构与操作有效性检查，整合而非选育。对 rsi-agent 的启示：NC 可以做到极致（零下游分），但代价是只适用于能被结构检查约束的改动（代码 harness），泛化到开放任务需配合 SP。
- **2611.23138 RSI-Prime**：把 RSI 当监督学习问题做——350 条完整轨迹 + decision-level loss masking。与 rsi-agent 的差异：它训练一个"会训练的静态模型"，rsi-agent 是运行时自改。共同点是都需保证监督信号质量，SP 要防轨迹污染。
- **2611.08946 Flex-RSI**：目前少见的把 executor 改解与 improver 改 harness 分开、并用 hidden test 终评的 benchmark。**"配对实验证明改 harness 本身贡献增益"** 支持 rsi-agent 把机制改造（而非只改答案）作为一等对象；但"只记成功/失败反馈拖累开源模型"提示 ESS 信号粒度要够。
- **2611.10719 No Second Run**：划出 ignition 边界——公开 RSI 依赖的"单一可验分 + 可重复试验"在预训练阶段全部失效。对 rsi-agent 的定位判断：我们是后训练/运行时 RSI，NC/ESS 的前提成立；一旦声称扩展到更底层，判据必须重写。
- 这 5 篇之外，A 线的"训练模型去训练模型"（RSI-Prime）与"harness 自改"（Flex-RSI）代表两个方向：前者自我改进对象是权重，后者是外壳；rsi-agent 四机制目前更贴近后者。

### B. 防自证 / 可证伪方法学（6 篇）

- **2610.11464 Who Verifies the Verifier?**（必读）：给出了一个可落地的 verifier 认证协议——锚定参考集一致性 + 无标签共识，且明确"从不按 agent 分数选"。它的 always-pass 反例是最强的警告：下游分无法认证 verifier。对 rsi-agent：SP 不能只是"私有"，还要有**锚定语义**（10 项参考集），否则私集也可能被共进化。
- **2611.21538 Optimistic Verifiers Get Hacked**（必读）：把 reward hacking 做成可精确测量的四表型实验，model-free audit 能在训练前预判。对 ESS闸：加一道 selection-error/gap-sign 的预检，比事后看分涨更可靠。
- **2611.08318 Criteria Exposure**：实证"暴露评分标准→过拟合标准"。对 SP：私集内容必须对被评对象不可见，且标准要 instruction-faithful，否则学到 verifier 而非任务。
- **2611.21315 Benchleaking**：把污染从 leaderboard 问题升级为"开放 base model 全下游研究"问题，Anti-Hack score 是现成的污染调整工具。对 SP 与 HA：进化工坊的语料要定期 decontamination 检查。
- **2610.10611 Foundations and Limits**：理论上区分 search gain 与 accepted-task coverage expansion。对 NC/ESS：调用次数下降、分数上升都不等于能力覆盖变宽，需要 explicit support + transcript 层面的证据。
- **2611.01313 Fake Citations**：预注册 + 证伪式研究的范本，且证明"按表面特征刷分"真实存在。对 NC：judge 若按引用数量等浅特征打分，会被系统性欺骗。

### C. 已有反面结果（5 篇）

- **2610.38392 Masked Erosion**（必读）：接受闸与冗余 safeguard 的对抗。最反直觉的是"更大 eval batch 更危险"——对 rsi-agent 的 ESS闸 是直接设计警告：闸的统计阈值必须与"看不见的冗余"分开审计，否则闸越强、侵蚀越快。
- **2611.12988 Third Eye**（必读）：即时收益≠持久收益，continuation 后掉 1.78pp。对 ESS闸：一次改进必须通过"再更新一次仍好"的检验，不能只看当下分。
- **2611.04952 Artifact-to-Capability**（必读）：三代理信号全绿仍可能无能力形成。对 HA：哈希审计是必要不充分——记录改动，还要验证能力 checkpoint。
- **2611.13524 RSI Tax**：bounded-note 基线省钱 65.6–71.0%，verification 占 74.8–93.6%。对 ESS闸：收益判据必须含成本，否则"改进"在经济上是负的。
- **2610.22984 Repair≠Generalize**：失败经验修旧任务、不迁移。对 SP：私集必须含未见任务分布，否则测到的只是 repair。

---

## 四、缺口（如实标注）

1. **时间窗**：本批 15 篇全部为 **2026 年**（10–11 月）投稿；在给定候选池与两次检索中未找到同等相关的 2025 年新论文进入名单。已知线索本身也集中在 2026。**2025 年覆盖不足**是事实，未硬凑。
2. **验号边界**：只对 abs 页做了 HTTP 200 + 号-题一致验证 + 摘要取用；未读全文 PDF/HTML，未复现任何数字。表中所有数字均为摘要自报，未经本实验室复算（按 AGENTS.md 铁律1，引用前须自行复算）。
3. **无 arxiv 专用 MCP 工具**：`tool_catalog` 查询 arxiv 无匹配；已加载 skill `aris-arxiv`，但批量 `id_list` 取摘要在本环境超时（TimeoutError），改用逐篇 `web_fetch abs` 更稳。这是一条可复用教训。
4. **与旧 notes 的去重**：基于标题/主题人工比对两份旧 notes 判定"新"，未做逐 id 程序化集合差；若需严格机器判定，可后续用脚本对三份 notes 抽 id 求差集。
5. **A 线正面参照相对少**：本批偏 B/C（防自证与反面结果），A 线本体 5 篇中较"纯本体"的新工作只有 RSI-Prime/No Second Run；若需更完整的 A 线谱系，应回到 2026-11-18-rsi-landscape.md 的 35 篇清单。
