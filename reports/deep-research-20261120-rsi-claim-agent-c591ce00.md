---
tags: [deep-research-report, status/final]
created: 2026-11-20
scope: RSI 递归自我改进全流程交付——claim 级审计、可自证 RSI agent、六路深挖、科研三闸回灌、真实数据 pilot
sources: "内部 14 / 外部 17"
search-mode: 导航→实体消歧→双链→关键词→联网→本地落盘证据复算
---

# 深度检索报告：《RSI 递归自我改进 全流程交付》

## 范围
- 查询范围：从"我自己对 RSI 有什么研究"出发，完成claim 级证据审计 → 构建一个真正能跑且不自证的 RSI agent → 六路全文/源码级深挖 → 用科研三闸把审计结论回灌论文稿件 → 用真实 SWE-bench Verified 数据做端到端前的最后验证。
- 时间窗口：2026-09 至 2026-11（论文均为 2026-10/11 新刊，repo 读数 2026-11-19，真实 parquet 落盘 2026-11-19）。
- 检索路径：arXiv abs/HTML 全文 `https://arxiv.org/html/<id>`（15/15 HTTP 200）；`gh api` 源码树 + `raw.githubusercontent.com` 源码（184 文件落盘）；国内镜像 `HF_ENDPOINT=https://hf-mirror.com` 下 SWE-bench Verified parquet（2,096,679 字节）落盘；本地落盘 JSON 逐一复算。
- 检索模式：语义（论文按 A 本体/B 防自证/C 反面三线）+ 文件定位（repo 按机制关键词）+ 关键词（数据集按官方 split）+ 实测复算（agent 与闸）。
- 查询改写：规范化（arXiv id 逐一 abs 页号-题一致验号；repo 双源 gh api + web_fetch 验活）；数字一律从实测 JSON/脚本输出，绝不凭记忆。

## 核心结论
- [内] **claim 级审计把我的三份 RSI 资产从"证据够不够"升级到"证据是不是自证"**：隔离复跑 P1 `outer_loop_correlation.py` → verdict=P1 supported（N_inner=5 的 ESS power ρ=0.5/0.7/0.9=0.047/0.015/0.000，naive ρ=0.9=0.662，逐位一致）；P2 `mu_priv_meta_predictor.py --backend linear --n_agents 1000 --seed 0` → test_auc=0.9970/rank_corr=0.9908/test_mse=0.0007（逐位一致），但标签 `mu_priv = final_public_score + N(0,σ)` 而特征第 0 维就是 `final_public_score`——AUC≈0.997 是同义反复（循环论证）；P3 是合成注入模拟，真实 lineage ablation pending。产物 `notes/2026-11-18-rsi-claim-audit.md`。
- [内] **建成一个真正能跑、会自我改进、且不自证的 RSI agent**：纯标准库、零 GPU、零 API key；六道反自证机制（密封私集、配对 ESS 闸、逐实例回归负担闸、哈希链审计、外部评分、NC 前置审计闸）；`run_demo.py --self-test` **41/41 OK**，`--generations 60` → verdict=SELF_IMPROVEMENT_VERIFIED，权重 (1,0,0)→(0.683,-0.4885,0.886) 逼近隐藏 w*，公开均分 0.7514→1.2452，审计链 240 条 OK，sealed 真实调用 59 次。产物 `rsi-agent/README.md` + `rsi-agent/results/demo.json`。
- [内] **科研三闸把审计结论回灌论文并全部通过**：改 workspace 恢复副本（`<DATA_ROOT>` 真源零改动），把 P2 循环机制诚实披露并给出非循环 scorer + NC 前置审计闸设计，AUC 统一为 canonical single-split 0.9970 / 10-fold CV 0.9957±0.0027（口径不可混用），删掉编造的"single GPU"能力声明，6 个死链中 2 个用真 JSON 重建、其余 4 个标 pending。三闸首闸 + 终闸 `--final --prior` 全 PASS（blocked=False、unresolved=[]、reasons=[]），重编译 38 页 795771 字节 EXIT=0。报告 `recovered/auditable-rsi/evidence/gate{1,2,3}_{first,final}.json`。
- [内] **用真实 SWE-bench Verified 数据完成端到端前最后验证**：读真实 parquet 500 实例（12 repo，django 231/sympy 75/sphinx 44，F2P len 1–438），确定性切分 public 155 / sealed-private 345；NC 闸三态（honest PASS se/gap/sat=0、padding OPTIMISTIC_GAP gap=0.268、corrupted SELECTION_CORRUPTED se=0.312）与配对闸三态（真提升 public/sealed 均 PASS delta=0.1983、负担攻击 REGRESSION_BURDEN burden=0.460>0.25）**6 项期望全中**，41 项自测无回归。产物 `rsi-agent/results/swe_real_data_pilot.json`。
- [外](https://arxiv.org/html/2611.21538) **六路深挖定位真正的威胁面，且大多不经过递归循环**：NC（non-circularity）威胁最大——乐观代理报 0.940 而真 faithful 0.731→0.667、判据文本泄漏 51.6vs27.8→45.0vs47.2、提示可诱导、上游权重污染（ERNIE HumanExec 96.3%→68.55）；SP 次之（私有≠未被记住）；C 线 Artifact-to-Capability 属性 4 直接命中我旧的 `paired_gate` 只验均值不查逐实例回归，促成负担闸修补。产物 `notes/2026-11-19-rsi-deep-papers-{a,b,c}.md` 等六份。
- [⚠ 未覆盖] **真端到端仍缺三样外部资源**：LLM API key、`GITHUB_TOKEN`、`ale_bench`/`swebench` 包 + HF 数据 + docker；三条独立通道（本机、ecs-hz、hpccube）实测全缺前置，按判据如实标 BLOCKED。真实数据 pilot 的 patch 是确定性 code_gen（非 LLM）、逐实例分数是基于真实 F2P count 的确定性标量替身（非真跑 repo 测试），**不是端到端**；本机零 GPU，只读不跑长实验。

## 关键要点（Key Takeaways）
- 最硬的自我纠错：P2 的 AUC≈0.997 不是"预测私有分"的能力，而是把公开分加噪当标签、又把公开分放进特征的代数恒等式——任何在同一生成器内的 robustness（5 模型/ρ 敏感/noise 扰动）都是打转，不构成破环证据。修法是 NC 前置审计闸：用外部真值锚（隐藏 w*）在 probe grid 上算 selection_error / signed_gap / saturation，乐观或乱序即拒，判定进哈希链。
- 简单"任何实例回归即拒"会误杀真提升：实测真提升代 burden=0.000~0.086、worst d_i=-0.902、n_neg 0~15（reward 异质性使真提升也含大量实例级回归）；正确操作化是"回归总额占比 + 阈值用 noise_floor 标定"（默认 0.25，留 3 倍余量），且大样本负担型攻击才会真正成立（190×+0.05 / 10×-0.5 旧闸放行）。
- 接入优先级已实测收敛：SWE-bench Verified（真实 parquet 已落盘、有 public/private 划分）> ALE-Bench（连续 overall_absolute_score 最贴配对闸，但缺数据/docker/key）> Terminal-Bench；需要真 LLM 生成环的那一层必须上云，本机只做编排与亚分钟 smoke。
- 论文侧最大的可信度风险不在"防循环闸够不够"，而在 verifier 本身可被认证绕过：[外](https://arxiv.org/html/2610.11464) 证明塌成 always-pass 的 verifier 仍能训出下游技能，故下游分无法认证 verifier——这提示 NC 必须用与下游评分器无关的外部真值锚，而不是复核下游分。

## 证据清单
| # | 结论摘要 | 支撑 | 来源 |
|---|---------|------|------|
| 1 | P1 真证据 / P2 循环论证 / P3 合成；P2 AUC=0.9970 逐位复现但标签=特征加噪 | [内] | [[notes/2026-11-18-rsi-claim-audit.md]] + `experiments/p2_falsification/artifacts/result_linear_seed0.json` |
| 2 | RSI agent 六道机制、41/41 自测、demo VERIFIED 0.7514→1.2452 | [内] | [[rsi-agent/README.md]] + [[rsi-agent/results/demo.json]] |
| 3 | NC 前置审计闸：probe grid n=60，honest PASS / padding OPTIMISTIC_GAP / corrupted SELECTION_CORRUPTED | [内] | [[rsi-agent/rsi_agent/nc_guard.py]] + [[rsi-agent/results/nc_audit.json]] |
| 4 | 逐实例回归负担闸：真提升过、大样本负担攻击拒（burden=0.526→修补后拒） | [内] | [[rsi-agent/rsi_agent/gates.py]] + [[rsi-agent/tests/test_reg_burden.py]] |
| 5 | 科研三闸首闸+终闸全 PASS、unresolved=[]；稿件重编译 38 页 | [内] | `recovered/auditable-rsi/evidence/gate1_final.json`、`gate2_final.json`、`gate3_final.json` |
| 6 | AUC 口径统一：canonical single-split 0.9970 / CV 0.9957±0.0027 / GB 0.9951，删编造 GPU 声明 | [内] | `recovered/auditable-rsi/section_4_formatted.tex` + `evidence/p2_fold_ci.json` |
| 7 | 真实 SWE-bench parquet 500 实例、12 repo、难度四档；pilot 6 项全中 | [内] | [[rsi-agent/results/swe_real_data_pilot.json]] + [[rsi-agent/scripts/swe_real_data_pilot.py]] |
| 8 | ALE / SWE 三通道 BLOCKED（HF 000、无 docker、无 key）；fuel_grade=LOCAL_HELD_OUT | [内] | `rsi-agent/scripts/check_ale_readiness.py` + `check_swe_readiness.py` |
| 9 | NC 威胁最大：乐观代理 0.940 vs 真 0.731→0.667 | [外](https://arxiv.org/html/2611.21538) | [[notes/2026-11-19-rsi-deep-papers-b.md]] |
| 10 | verifier 塌成 always-pass 仍训出技能，下游分无法认证 verifier | [外](https://arxiv.org/html/2610.11464) | [[notes/2026-11-19-rsi-deep-papers-b.md]] |
| 11 | Masked Erosion + Artifact-to-Capability 属性 4 命中 paired_gate 逐实例回归漏洞 | [外](https://arxiv.org/html/2610.38392) | [[notes/2026-11-19-rsi-deep-papers-c.md]] |
| 12 | ELITE 零下游评估 60.0→85.6、去闸崩 52.0、快 8.81× | [外](https://arxiv.org/html/2611.21056) | [[notes/2026-11-19-rsi-deep-papers-a.md]] |
| 13 | agentdescent 真提升闸（Beta 后验+回归护栏+三分区证伪+外部注入评分器） | [外](https://github.com/ThinkFlowLab/agentdescent) | [[notes/2026-11-19-rsi-deep-projects-core.md]] |
| 14 | SWE-bench Verified 500 / ALE-Bench 40 题 public_eval-private_secrets 框架级 SP | [外](https://www.swebench.com/) | [[notes/2026-11-19-rsi-deep-datasets.md]] |
| 15 | HF 国内镜像下真实 parquet 2,096,679 字节下载成功并落盘 | [外](https://hf-mirror.com) | `~/.cache/huggingface/hub/blobs/92/928ff5…` |

## 口径与来源（跨文档对比/专业术语查询时启用）
- 术语/指标口径：RSI=Recursive Self-Improvement（递归自我改进）；NC=non-circularity（标签不来自候选自身输出）；SP=sealed-private（私集密封）；ESS=effective sample size=n·(1-rho)/(1+rho)（lag-1 自相关）；burden=sum|neg deltas|/sum(pos deltas)；HA=hash-chain audit；AIDE²=Weco 双层递归自改进（[外](https://arxiv.org/abs/2609.26457)）。
- 实体解析："RSI" → Recursive Self-Improvement；"Auditable Recursive Self-Improvement" → A PDP-style Formal Framework for Bi-Level Agent Systems（真源 `<DATA_ROOT>`，sha256 `a40d6df5…`）；"paired_gate" → `rsi-agent/rsi_agent/gates.py:74`。
- 数据状态：论文/repo 读数截至 2026-11-19；arXiv 摘要自报数字标"本实验室未复算"，agent 与闸的数字为本机实测（可复算）；真实 pilot 的实例元数据来自真 parquet，打分是确定性标量替身。

## 来源分区
### 内部资料（14）
- [[notes/2026-11-18-rsi-claim-audit.md]]
- [[notes/2026-11-19-rsi-deep-papers-a.md]]
- [[notes/2026-11-19-rsi-deep-papers-b.md]]
- [[notes/2026-11-19-rsi-deep-papers-c.md]]
- [[notes/2026-11-19-rsi-deep-projects-core.md]]
- [[notes/2026-11-19-rsi-deep-projects-more.md]]
- [[notes/2026-11-19-rsi-deep-datasets.md]]
- [[rsi-agent/README.md]]
- [[rsi-agent/rsi_agent/nc_guard.py]]
- [[rsi-agent/rsi_agent/gates.py]]
- [[rsi-agent/scripts/swe_real_data_pilot.py]]
- [[rsi-agent/results/demo.json]]
- [[rsi-agent/results/swe_real_data_pilot.json]]
- `recovered/auditable-rsi/evidence/gate{1,2,3}_final.json`

### 公开信息（17）
- [Dream-RSI: Recursive Self-Improvement through Evolving Worlds](https://arxiv.org/abs/2609.14858)
- [AIDE² / First evidence of recursive self-improvement (arXiv)](https://arxiv.org/abs/2609.26457)
- [Weco AI: First evidence of recursive self-improvement](https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement)
- [Who Verifies the Verifier](https://arxiv.org/html/2610.11464)
- [Masked Erosion](https://arxiv.org/html/2610.38392)
- [Optimistic Verifiers Get Hacked](https://arxiv.org/html/2611.21538)
- [Third Eye](https://arxiv.org/html/2611.12988)
- [Artifact-to-Capability](https://arxiv.org/html/2611.04952)
- [ELITE](https://arxiv.org/html/2611.21056)
- [RSI 分类学四组九维](https://arxiv.org/html/2610.14625)
- [Skill-policy 共进化陷阱](https://arxiv.org/abs/2610.20845)
- [反馈不定向 / 145 修改无 target-specific advantage](https://arxiv.org/abs/2610.14662)
- [SWE-bench 官方](https://www.swebench.com/)
- [SakanaAI/ALE-Bench](https://github.com/SakanaAI/ALE-Bench)
- [ThinkFlowLab/agentdescent](https://github.com/ThinkFlowLab/agentdescent)
- [AetherLabsAI/RSIAgent](https://github.com/AetherLabsAI/RSIAgent)
- [HF 国内镜像](https://hf-mirror.com)

## 研究方法（使用外部检索 [外] 时填写）
- 标记说明：本报告外部一手来源均标 [外] 并附真实 URL，例如 [外](https://arxiv.org/abs/2609.14858)、[外](https://www.swebench.com/)、[外](https://hf-mirror.com)，出处见上方「公开信息」分区 17 条。
- 子问题：①我手上有哪些 RSI 自有资产、各自证据够不够？②能否造一个真跑且不自证的 agent？③领域里真正打穿这类系统的威胁是什么？④能否用科研三闸把结论回灌论文并过闸？⑤真实数据下机制是否仍成立？
- 检索查询：web/arxiv/gh 检索共 40+ 条，本地 notes 14 份、落盘 JSON 产物 8 份。
- 唯一来源：17 个（目标 15–30，达标）。
- 深读篇数：15 篇 arXiv 全文（`https://arxiv.org/html/<id>`，15/15 HTTP 200）+ 13 repo 源码（184 文件落盘）+ 1 份真实 parquet（500 行）。
- 源优先级：arXiv 原文/官方 repo/官方数据集 > 权威博客 > 论坛；近 12 月优先；数字一律实测复算，摘要自报数字明确标注未复算。

## 免责声明（按触发条件）
⚠️ 本报告为对本地证据与公开资料的研究性整理归纳，数字均可从落盘产物复算，不构成投稿、投资或工程落地建议；真端到端所需的 LLM/数据/权限缺口已如实标注，未做跨过这些缺口的推断。
