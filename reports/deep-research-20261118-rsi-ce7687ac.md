---
tags: [deep-research-report, status/review]
created: 2026-11-18
scope: 六路深挖汇总：15篇论文全文+13个开源项目源码+11个数据集接入级，以及由 C 线证据驱动的 rsi-agent 逐实例回归闸修补（TDD 三证）
sources: "内部 6 / 外部 40+"
search-mode: 导航→实体消歧→双链→关键词→联网
---

# 深度检索报告：《RSI 深挖总报告 六路全文源码级 逐实例回归闸修补》

## 范围
- 查询范围：六路深挖汇总：15篇论文全文+13个开源项目源码+11个数据集接入级，以及由 C 线证据驱动的 rsi-agent 逐实例回归闸修补（TDD 三证）
- 时间窗口：2026-10 至 2026-11（论文全为 2026-10/11 新刊）；repo 读数 2026-11-19
- 检索路径：arxiv HTML 全文 `https://arxiv.org/html/<id>`（15/15 HTTP 200）+ gh api 源码树 + raw.githubusercontent.com 源码（184 文件落盘 analyses/tracks/rsi-deep/*/sources/）；git clone 被坏代理 127.0.0.1:7900 挡，一律走 gh api + raw
- 检索模式：语义（论文按 A 本体/B 防自证/C 反面三线）+ 文件定位（repo 按机制关键词）+ 关键词（数据集按官方 split）
- 查询改写：规范化（arXiv id 逐一 abs 页验号；repo 双源验活 gh api + web_fetch）

## 核心结论
- [内] **rsi-agent 已跑通且不自证**：纯标准库、零 GPU、零 API key；`run_demo.py --self-test` 17/17 OK（本轮修补后 15→17），`--generations 60` → verdict=SELF_IMPROVEMENT_VERIFIED，权重 (1,0,0)→(0.683,-0.488,0.886) 逼近 w*，公开均分 0.751→1.245，审计链 240 条 OK，sealed 真实调用 59 次。
- [内+外] **深挖挖出真缺陷并已修补**：C 线（Masked Erosion 2610.38392 + Artifact-to-Capability 2611.04952）指出 `paired_gate` 只验均值、不查逐实例回归。实测证实：大样本负担型攻击（190 实例 +0.05、10 实例 -0.5，burden=0.526）均值+ESS+显著性全过、旧闸放行。补 REGRESSION_BURDEN 闸（burden=sum|neg|/sum(pos)，默认 0.25，真提升实测上限 0.086 留 3 倍余量）。TDD 三证：零容忍拒/0.25 拒/None 放行/真提升过，全套自测 17 OK，demo 仍 VERIFIED。
- [外](https://www.swebench.com/) **数据集三档**：SWE-bench（Verified 500 public/full 其余 private、二值 F2P、CPU 可跑）作主评测；ALE-Bench（40 题、public_eval=public/private_secrets=private 框架级天然 SP、连续 overall_absolute_score 最贴 paired_gate/ESS）作细粒度改进曲线；Terminal-Bench（core v0.1.1=80、task_id_subset 一半 public）。SWE-smith 52k 合成自举 private split；KernelBench 需 GPU 暂不可跑。
- [外](https://arxiv.org/html/2610.11464) **论文四压力测试直击四机制**：NC 威胁最大（四篇攻击都不经过循环——乐观代理 0.940 vs 真 0.731→0.667、判据文本泄漏 51.6vs27.8→45.0vs47.2、提示可诱导、上游权重污染 ERNIE 96.3%→68.55），NC 只断算术环；SP 次之（私有≠未被记住）；ESS 要配对加扰动双胞胎（池化 p=0.014 假阳、聚类 p=0.12）；HA 九 repo 全空（grep prev_hash/hash_chain/merkle 全无）。
- [外](https://github.com/ThinkFlowLab/agentdescent) **项目同构参照**：agentdescent（diff=梯度/aggregator=优化器/held-out gate，离线 demo 无 key 10s）同构度最高；RSIAgent（Actor/Verifier 真隔离）密封闸工程化大尺度版；ScienceBuddy（inner 改 harness + outer GRPO，sha256+os.link 不可变消息互锁）；Godel_Agent 无循环防护=反面教材；lemoz DGM 逐 benchmark 非回归过滤+held-out 冻结重放（507 tests/480 私有）。
- [⚠ 未覆盖] **数字复算边界**：六路笔记 114 处「本实验室未复算」标注（论文数字均摘要/正文自报、本机零 GPU 只读不跑长实验）；aira-dojo 独立任务数未给、ALE-Bench 40 无 repo 字面计数、HF 全通道本机不通；P3 真实 lineage 4 条件×30 步 ablation 仍 pending。

## 关键要点（Key Takeaways）
- 可执行加固 ①NC 前置 model-free 审计闸（probe grid 算 selection error+signed gap+饱和 pilot，error>0.15 或乐观 gap 即拒，判定进 HA 链）②判据文本对 inner 不可读+instruction-faithful 重判（领先须在重判下存活）+HA 增判据字符串暴露审计 ③SP 启用前 strict-match 续写去污染探针+证书 hash 挂密封集 hash。
- 抄作业优先级：agentdescent FalsificationAcceptance（needs_probes/defer、三分区+拒绝点名 task id、probes.evaluate 外部注入 inner 不可碰）> ScienceBuddy atomic_json(immutable=True)+sha256 补 HA > lemoz _passes_regression_filter + 不相交 held-out 重放 > meclaw 测量 ledger 循环不可写（NC 强形态）。
- 接入路径零成本起步：rsi-agent demo（已验证）→ ALE-Bench CPU pilot（连续分数直接量化每轮改进）→ SWE-bench Verified 500（Docker+F2P 客观打分）→ 需资源层（MLE-Bench full 3.3TB / KernelBench GPU）上云。
- 方法论教训：简单「任何实例回归即拒」会误杀真提升（实测真提升代 burden 0.000~0.086、worst d_i -0.902、n_neg 0~15，reward 异质性导致真提升也有大量实例级回归）；正确操作化是「回归总额占比 + 阈值用 noise_floor 标定」。

## 证据清单
| # | 结论摘要 | 支撑 | 来源 |
|---|---------|------|---|
| 1 | rsi-agent 17/17 自测过、demo VERIFIED 0.751→1.245 | [内] | [[rsi-agent/tests/test_rsi_agent.py]] + [[rsi-agent/results/demo.json]] |
| 2 | 逐实例回归漏洞：大样本负担型攻击旧闸放行（burden 0.526） | [内] | [[rsi-agent/rsi_agent/gates.py:74-117]] + 反证探针 |
| 3 | REGRESSION_BURDEN 闸三证（零容忍拒/0.25 拒/None 放行/真提升过） | [内] | [[rsi-agent/tests/test_reg_burden.py]] |
| 4 | A 线：RSI-Prime decision 级 36.56→48.24；ELITE 去闸崩 52.0、快 8.81×；Flex-RSI 配对 7 对 6 对提升；RSI Tax verification 占税 74.8–93.6% | [外](https://arxiv.org/html/2610.11464) | notes/2026-11-19-rsi-deep-papers-a.md（12 URL，66 未复算标注） |
| 5 | B 线：NC 威胁最大（乐观代理 0.940 vs 真 0.731→0.667；判据泄漏 51.6vs27.8→45.0vs47.2；ERNIE 96.3%→68.55） | [外](https://arxiv.org/html/2611.21538) | notes/2026-11-19-rsi-deep-papers-b.md（16 URL，5 未复算标注） |
| 6 | C 线：Artifact-to-Capability 属性 4 直指 paired_gate 不查逐实例回归 | [外](https://arxiv.org/html/2610.38392) | notes/2026-11-19-rsi-deep-papers-c.md（8 URL，37 未复算标注） |
| 7 | agentdescent 真提升闸 defaults.py:359 Beta 后验+回归护栏、acceptance.py:157 三分区证伪 | [外](https://github.com/ThinkFlowLab/agentdescent) | notes/2026-11-19-rsi-deep-projects-core.md（27 file:line 引用） |
| 8 | RSIGame value_stop.py:46-47 stop_k()=3、:71-79 saturated 判定；meclaw/Geneclaw/lemoz 五层门 | [外](https://github.com/WenyiWU0111/RSIGame) | notes/2026-11-19-rsi-deep-projects-more.md（36 file:line 引用，21 URL） |
| 9 | SWE-bench Verified 500 public、ALE-Bench public_eval/private_secrets 框架级 SP、Terminal-Bench v0.1.1=80 | [外](https://www.swebench.com/) | notes/2026-11-19-rsi-deep-datasets.md（20 URL） |
| 10 | 9 repo 均无 HA（grep prev_hash/hash_chain/merkle 全空） | [外](https://github.com/ThinkFlowLab/agentdescent) | notes/2026-11-19-rsi-deep-projects-more.md |
| 11 | 六路 184 源码文件落盘 analyses/tracks/rsi-deep/{projects-core,projects-more}/sources/ | [内] | analyses/tracks/rsi-deep/README.md |
| 12 | 论文 15 篇全部 web_fetch abs HTTP200+号题一致；repo 13 个 gh api+web_fetch 双源验活；数据集 11 个官方 URL HTTP200 | [外](https://arxiv.org/html/2609.14858) | 三路燃料笔记 + lead 补验 3 arXiv id |
| 13 | 缺口：论文数字摘要自报未复算、2025 覆盖不足、P3 真实 lineage ablation pending | [⚠] | 各笔记「诚实读取缺口」节 |

## 口径与来源（跨文档对比/专业术语查询时启用）
- 术语/指标口径：burden=sum|neg deltas|/sum(pos deltas)（配对差 d_i=c_i-b_i）；ESS=n*(1-rho)/(1+rho)（lag-1 自相关）；NC=non-circularity（标签不来自候选自身输出）；SP=sealed-private（私集密封）；ESS=effective sample size；HA=hash-chain audit。
- 实体解析："RSI" → Recursive Self-Improvement（aliases: 递归自我改进）；"AIDE²" → Weco 双层递归自改进博客+arXiv:2609.26457；"paired_gate" → rsi-agent/rsi_agent/gates.py:74。
- 数据状态：论文/repo 读数 2026-11-19；arXiv 摘要自报数字标「本实验室未复算」；rsi-agent 数字为本机实测（可复算）。

## 来源分区
### 内部资料（6）
- [[notes/2026-11-19-rsi-deep-papers-a.md]]
- [[notes/2026-11-19-rsi-deep-papers-b.md]]
- [[notes/2026-11-19-rsi-deep-papers-c.md]]
- [[notes/2026-11-19-rsi-deep-projects-core.md]]
- [[notes/2026-11-19-rsi-deep-projects-more.md]]
- [[notes/2026-11-19-rsi-deep-datasets.md]]
- [[rsi-agent/rsi_agent/gates.py]] [[rsi-agent/tests/test_reg_burden.py]] [[rsi-agent/results/demo.json]] [[analyses/tracks/rsi-deep/README.md]]

### 公开信息（40+）
- [RSI-Prime / ELITE / Flex-RSI / No Second Run / RSI Tax 等 15 篇 arXiv 全文](https://arxiv.org/html/2610.11464)（15/15 HTTP 200，号-题一致）
- [Weco AIDE² 博客](https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement)
- [ThinkFlowLab/agentdescent](https://github.com/ThinkFlowLab/agentdescent) [Arvid-pku/Godel_Agent](https://github.com/Arvid-pku/Godel_Agent) [AetherLabsAI/RSIAgent](https://github.com/AetherLabsAI/RSIAgent) [Gen-Verse/ScienceBuddy](https://github.com/Gen-Verse/ScienceBuddy) [WenyiWU0111/RSIGame](https://github.com/WenyiWU0111/RSIGame) [mmeyerlein/meclaw](https://github.com/mmeyerlein/meclaw) [Clawland-AI/Geneclaw](https://github.com/Clawland-AI/Geneclaw) [lemoz/darwin-godel-machine](https://github.com/lemoz/darwin-godel-machine) [mmtmn/Darwin-Godel-Machine](https://github.com/mmtmn/Darwin-Godel-Machine) 等 13 repo
- [SWE-bench](https://www.swebench.com/) [MLE-Bench](https://github.com/openai/mle-bench) [ALE-Bench](https://ale-bench.github.io/) [Terminal-Bench](https://www.terminal-bench.com/) [SWE-smith](https://arxiv.org/abs/2504.21798) [MLAgentBench](https://arxiv.org/abs/2310.03302) [KernelBench](https://arxiv.org/abs/2502.10517)

## 研究方法（使用外部检索 [外] 时填写）
- 子问题：①15 篇论文全文级机制与数字 ②13 repo 源码级闸/门/隔离 ③11 数据集接入级 split/打分/算力 ④C 线指出的 rsi-agent 漏洞是否成立、如何修补 ⑤修补后是否误杀真提升
- 检索查询：web + news 共 40+ 条（六路并行，每路独立检索+源码下载）
- 唯一来源：40+ 个（目标 15-30，超出）
- 深读篇数：15 篇 arXiv 全文（WebFetch html 全文）+ 13 repo 源码（184 文件）+ 11 数据集官方页
- 源优先级：arXiv 全文/官方 README/gh api 源码 > 博客 > 论坛；2026-10/11 优先
- 修补纪律：TDD（先红测试→补闸→转绿）+ 反证（零容忍/0.25/None 三档）+ 不误杀（真提升默认过）+ demo 仍 VERIFIED

## 免责声明（按触发条件）
⚠️ 本报告基于现有知识库资料的整理归纳，信息以原始资料为准，不构成专业医疗/法律/投资/教育等建议。论文数字为作者自报、本实验室未复算（114 处标注）；repo star 为约数（读数 2026-11-19）。
