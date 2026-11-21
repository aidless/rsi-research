---
tags: [deep-research-report, status/review]
created: 2026-11-19
scope: RSI agent 燃料：可跑数据集 前沿论文 对标开源项目
sources: "内部 5 / 外部 40+"
search-mode: 导航→实体消歧→联网→一手页验证
---

# 深度检索报告：《RSI agent 燃料：可跑数据集 前沿论文 对标开源项目》

## 范围
- 查询范围：为已跑通的 rsi-agent（上下文决策试验床）找三类燃料——可跑公开数据集、2025–2026 前沿论文、可对标开源项目
- 时间窗口：数据集/项目为截至 2026-11-19 的在册状态；论文以 2026 年 10–11 月为主（2025 年覆盖不足，已标缺口）
- 检索路径：盘上 4 份 RSI notes 去重 → platform_search / domain_search 召回 → 官方页 + arXiv abs + GitHub API 一手验活
- 检索模式：关键词 + 语义召回 + 一手页精读
- 查询改写：规范化（数据集名 / 机构名 / arXiv id 精确匹配）

## 核心结论
- [内] **rsi-agent 已在合成试验床验证，缺的是真实燃料**：现有 demo 用 ContextualTask（隐藏 w\*）跑出 SELF_IMPROVEMENT_VERIFIED，但这是自建合成任务；三轮审计已确认「能跑、不自证」，下一步的价值在于换真实评测源与对标系，见 notes/2026-11-18-rsi-claim-audit.md 与 rsi-agent/results/demo.json。
- [外] **数据集三档已定位**：SWE-bench（full 2,294 / Verified 500，https://www.swebench.com/）、MLE-Bench（75 竞赛 / Lite 22，https://github.com/openai/MLE-Bench）、ALE-Bench（40 题连续分数，https://sakana.ai/ale-bench/）——分别覆盖「大规模客观打分 / 长程改进 / 细粒度连续分数」三种评测需求，均有 CPU pilot 路径。
- [外] **论文给出四道机制的四个致命压力测试**：评分循环（https://arxiv.org/abs/2610.11464）、闸功效反向激励（https://arxiv.org/abs/2610.38392）、即时≠持久（https://arxiv.org/abs/2611.12988）、代理信号≠能力（https://arxiv.org/abs/2611.04952）——都是对 rsi-agent 现有密封闸 / ESS 闸 / 哈希审计的直接挑战。
- [外] **项目存在同构参照**：agentdescent 的 held-out gate（https://github.com/ThinkFlowLab/agentdescent）与 rsi-agent 密封确认闸同构；RSIAgent 的 Actor/Verifier 隔离（https://github.com/AetherLabsAI/RSIAgent）是密封闸的工程化大尺度版。
- [内+外] **行动路径已可执行**：零成本起步（agentdescent 离线 demo + DGM Ollama，https://github.com/mmtmn/Darwin-Godel-Machine）→ CPU pilot（ALE-Bench + SWE-bench Verified 做 public/private split）→ 需资源层（MLE-Bench Lite 上云、KernelBench 需 GPU）。

## 关键要点（Key Takeaways）
- 零成本今天就能跑：agentdescent demo（离线无 key 10 秒）与 mmtmn/Darwin-Godel-Machine（Ollama 本地无 key），本机直接对标自改进循环。
- CPU pilot 首选 ALE-Bench（40 题、连续分数、Apache-2.0、不需 GPU）：唯一能直接量化「每轮改进了多少」的公开集，正对改进曲线需求。
- public/private split 直接用 SWE-bench 官方切分：Verified(500) 当 public、full 剔除后当 private，避免进化记住测试题。
- 四机制加固有论文可依：私集加「锚定语义」(2610.11464)、ESS 闸审计「看不见的冗余防护」(2610.38392)、改进必须过「再更新一次仍好」(2611.12988)、哈希审计后补「能力 checkpoint」(2611.04952)。
- 环境红线：本机零可用 GPU（AGENTS.md §17），KernelBench / MLE-Bench full / RE-Bench GPU 任务需云端通道，上云前现测 ssh。

## 证据清单
| # | 结论摘要 | 支撑 | 来源 |
|---|---------|------|------|
| 1 | rsi-agent 合成试验床已验证、需真实燃料 | [内] | notes/2026-11-18-rsi-claim-audit.md + rsi-agent/results/demo.json |
| 2 | SWE-bench 2,294/500 可做客观打分与 split | [外] | https://www.swebench.com/ |
| 3 | MLE-Bench 75/22 对改进策略敏感 | [外] | https://github.com/openai/MLE-Bench |
| 4 | ALE-Bench 40 题连续分数、CPU 可跑 | [外] | https://sakana.ai/ale-bench/ |
| 5 | Who Verifies the Verifier：下游分无法认证 verifier | [外] | https://arxiv.org/abs/2610.11464 |
| 6 | Masked Erosion：更大 eval batch 反删 safeguard | [外] | https://arxiv.org/abs/2610.38392 |
| 7 | Third Eye：即时 utility≈0、continuation 降 1.78pp | [外] | https://arxiv.org/abs/2611.12988 |
| 8 | Artifact-to-Capability：三信号全绿仍无能力 | [外] | https://arxiv.org/abs/2611.04952 |
| 9 | RSIAgent Actor/Verifier 隔离 | [外] | https://github.com/AetherLabsAI/RSIAgent |
| 10 | agentdescent held-out gate 同构、离线 demo | [外] | https://github.com/ThinkFlowLab/agentdescent |
| 11 | Godel_Agent 自指改 policy，无循环防护 | [外] | https://github.com/Arvid-pku/Godel_Agent |
| 12 | ScienceBuddy 双层交替、冻结 715/90/90 | [外] | https://github.com/Gen-Verse/ScienceBuddy |
| 13 | 三个数据集 arXiv id 已补验号一致 | [内+外] | https://arxiv.org/abs/2504.21798 + notes/2026-11-19-rsi-datasets.md |
| 14 | 2025 年论文覆盖不足、数字自报未复算 | [⚠] | 三份 notes 缺口清单 |

## 来源分区
### 内部资料（5）
- notes/2026-11-19-rsi-datasets.md
- notes/2026-11-19-rsi-papers.md
- notes/2026-11-19-rsi-projects.md
- notes/2026-11-18-rsi-claim-audit.md
- rsi-agent/README.md

### 公开信息（40+，列主要）
- [SWE-bench](https://www.swebench.com/)
- [MLE-Bench](https://github.com/openai/MLE-Bench)
- [ALE-Bench](https://sakana.ai/ale-bench/)
- [Who Verifies the Verifier](https://arxiv.org/abs/2610.11464)
- [Masked Erosion](https://arxiv.org/abs/2610.38392)
- [Optimistic Verifiers Get Hacked](https://arxiv.org/abs/2611.21538)
- [Third Eye](https://arxiv.org/abs/2611.12988)
- [Artifact-to-Capability](https://arxiv.org/abs/2611.04952)
- [ELITE / Evaluation-Free RSI](https://arxiv.org/abs/2611.21056)
- [Benchleaking Is Accelerating](https://arxiv.org/abs/2611.21315)
- [RSI Tax](https://arxiv.org/abs/2611.13524)
- [agentdescent](https://github.com/ThinkFlowLab/agentdescent)
- [Godel_Agent](https://github.com/Arvid-pku/Godel_Agent)
- [RSIAgent](https://github.com/AetherLabsAI/RSIAgent)
- [ScienceBuddy](https://github.com/Gen-Verse/ScienceBuddy)
- [SWE-smith](https://github.com/SWE-bench/SWE-smith)
- [RE-Bench](https://github.com/METR/RE-Bench)
- [Terminal-Bench](https://www.tbench.ai/)
- [KernelBench](https://github.com/ScalingIntelligence/KernelBench)
- [AIDE aideml](https://github.com/wecoai/aideml)
- [DGM (mmtmn)](https://github.com/mmtmn/Darwin-Godel-Machine)

## 研究方法（使用外部检索 [外] 时填写）
- 子问题：①哪些公开集能当 RSI 的 public/private split 且本机可跑 ②2025–2026 有哪些对四机制的新压力测试 ③哪些开源项目与 rsi-agent 同构可对标
- 检索查询：platform_search(github) + domain_search(arxiv) + web_search，共 3 路 × 多轮
- 唯一来源：40+ 个（三份 notes 逐条 web_fetch / gh api 验活）
- 深读篇数：三路子代理各精读一手页（数据集 11 / 论文 15 abs / 项目 13 repo + README）
- 源优先级：官方 GitHub README / 项目官网 / arXiv abs > 博客 > 论坛；近 12 月优先；arXiv id 逐篇号-题一致验号
- lead 复核：补验 3 个数据集 arXiv id（2504.21798 / 2310.03302 / 2502.10517）全部 HTTP 200 + 号-题一致

## 免责声明（按触发条件）
⚠️ 本报告基于现有知识库资料的整理归纳，信息以原始资料为准，不构成专业医疗/法律/投资/教育等建议。
