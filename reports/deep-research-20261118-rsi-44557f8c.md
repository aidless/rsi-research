---
tags: [deep-research-report, status/review]
created: 2026-11-18
scope: "工作区 RSI 自有产出盘点 + 2610.* 领域进展 + 外部一手锚点 + Auditable RSI 正文恢复"
sources: "内部 15 / 外部 9"
search-mode: "文件定位→验号→四轴归纳→反面证据→会话/备份恢复"
---

# 深度检索报告：《RSI 递归自我改进——你做了什么，领域到了哪里》

## 范围
- 查询范围：工作区 `<WORKSPACE>` 内用户的 RSI 相关产出与台账；2610.* RSI 论文摘要库；外部一手 arXiv abs 页 + Weco AI 官方博客；`$HOME/.research` 与 `<DATA_ROOT>` 下的正文真源。
- 时间窗口：自有产出 2026-09～2026-11；领域论文 2026-09～2026-10；外部核验与正文恢复 2026-11-18。
- 检索路径：三个并行子代理（文献版图 `notes/2026-11-18-rsi-landscape.md`、失败模式 `notes/2026-11-18-rsi-failure-modes.md`、正文恢复 `notes/2026-11-18-auditable-rsi-recovery.md`）+ 主会话独立 web_fetch 验号与恢复件复核。
- 检索模式：文件定位 + 关键词 + 号-题一致验号（AGENTS.md 铁律 6）+ dsh 会话/备份恢复 + 哈希对账。
- 查询改写：无（精确标识主导，跳过 HyDE / Step-back）。

## 核心结论
- [内] **结论 1——你的 RSI 研究盘点**：手上 4 类资产，均无活跃实验。① Dream-RSI 失败签名复现（`stall_detector.py` + `dream-rsi-fig5-data.json` + `Dream-RSI.pdf`，本次实跑 11/11 自检 + 四臂 replay 通过）；② Auditable Recursive Self-Improvement 完整投稿包（本次已找回，见结论 4）；③ 证据最硬的 `paper_gate_review_r2/`《Fail-Closed Auditable Promotion Gates》（43 KB 正文 + CLAIM_LEDGER + 三轮评审 + 真凭链）；④ 一圈 2610.* 领域情报跟踪。
- [内+外] **结论 2——领域到哪了**：RSI 已从"提出自进化框架"进入"出现可证伪反面结果"阶段。CODA 抹掉 skill-policy 共进化增益的 52–93%、TEB 的 no_evolve 靠 rescore 白涨 7.5/5.0/12.5pp、arXiv:2610.14662 的 145 条修改无 target-specific advantage（[2609.14858](https://arxiv.org/abs/2609.14858) 与 [Weco AIDE²](https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement) 提供正向对照）——Landscape report.md:112 的"成熟信号"判读被本批论文证实。
- [内+外] **结论 3——你的三条预测都还不够判真伪**：P1（ignition 噪声下界）缺本体实验，只有旁证；P2（ungameable private score 必要性）只有 synthetic R²≈0.9957 + 相关证据，缺真实 lineage 与"去私有分"消融；P3（反作弊 ≡ 审计通道容量演化）措辞强于证据——AIDE² 博客只给 "a possible explanation" 的相关解释，无任何名为 audit-channel capacity 的变量被测量。
- [内] **结论 4——Auditable RSI 正文完整找回，PROVEN**：2026-10-28 评审包记录的 `main.tex`（sha256 `a40d6df5…`）在工作区 `.research/` 消失，但真源仍在 `<DATA_ROOT>`，`$HOME/.research/bi-level-pepa/` 是其软链视图。本次已逐字恢复到 `recovered/auditable-rsi/`（47 文件整投稿包），主会话独立复核四份 sha256 全为 `a40d6df5…`、`cmp` 无输出、`main.pdf` 37 页、7 正文节 + 7 附录节 + 5 图 + 24 文献 + CLAIM_LEDGER + evidence/ + pre_registration/ 齐全。**这是完整正文，不是 507 词骨架**——`words=507`/`paper-portfolio.md:90` 的"0 页"是单文件 wrapper 与未编译登记口径，与真源矛盾，以后者为准。注意 P2 实证仍是 synthetic，原文自述 awaits AIDE85's lineage release。
- [内+外] **结论 5——一条自否定实测值得单独记**：你自己的 `paper_gate_review_r2/main.tex:63-69` 里，五键门 false-promotion 0.0125 与 unpaired 检验 0.0150 几乎无差别，作者明说"保护来自显著性检验本身，不来自五键结构"。这是"结构复杂度无增益"的典型反面结果，与 2610.19789（self-distillation reality check）同向。

## 关键要点（Key Takeaways）
- 你的 RSI 线有两份可重启的完整资产：`recovered/auditable-rsi/`（37 页完整投稿包，本次找回）与 `paper_gate_review_r2/`（含真凭链和一条诚实自否定结论，C15/§7 口径矛盾只差下一轮处置）。两者的共同短板都是 P2 实证缺失。
- 侦探器 `stall_detector.py` 的四臂就是阈值校准集（in-sample）：11/11 自检与四臂 replay 是"复现校准"，不是独立验证集证据，引用时不能当泛化性能。
- 领域的可证伪性现在主要来自反面结果（CODA / TEB / 2610.14662 / AIDE² 非单调泛化），而不是新的正向框架——这是判断"这条线还值不值得追"的关键背景。
- P3 的"等价性"要降级为"相关性"，除非在真实 lineage 上同时测 audit-channel capacity 与 hacking rate 并给出反例判据。

## 口径与来源
- 术语口径：RSI = Recursive Self-Improvement；Level 1（net positive）/ Level 2（ignition 后）/ ignition / third-order generalization 采用 [Weco 4 Levels](https://www.weco.ai/blog/4-levels-of-recursive-self-improvement) 定义；"可证伪"按 prediction-audit G5 预注册阈值方向口径。
- 实体解析："Auditable RSI" → 稿件 `bi-level-pepa`（title: *Auditable Recursive Self-Improvement: A PDP-style Formal Framework for Bi-Level Agent Systems*，源 `<WORKSPACE_ROOT>/.research/bi-level-pepa/main.tex`）；与 `paper_gate_review_r2`（Fail-Closed Auditable Promotion Gates）是**两篇不同稿件**。
- 数据状态：内部摘要台账截至 2026-11-18；领域论文摘要截至 2026-10；外部核验与正文恢复 2026-11-18。2610.* 数字均为摘要内数字（证据等级 B，未 PDF/源码复算）；AIDE² 数字取自官方博客原文（一手）。

## 证据清单
| # | 结论摘要 | 支撑 | 来源 |
|---|---------|------|------|
| 1 | stall_detector 11/11 自检 + 四臂 replay 通过，阈值 in-sample | [内] | `stall_detector.py`；`dream-rsi-fig5-data.json`；`notes/2026-11-18-rsi-failure-modes.md:38-56` |
| 2 | Auditable RSI 正文完整找回，四份 sha256 全匹配、cmp 逐字节相同、37 页 | [内] | `recovered/auditable-rsi/main.tex`；`notes/2026-11-18-auditable-rsi-recovery.md:6-14,27-30` |
| 3 | 2610.* 四轴分类 + 35 篇验号 + 反面结果组 | [内] | `notes/2026-11-18-rsi-landscape.md`；`papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/worktree/untracked/analyses/agent-landscape-2026-10/report.md:101-112` |
| 4 | F1–F8 失败模式 + P1/P2/P3 不够判真伪 + 四级证据边界 | [内] | `notes/2026-11-18-rsi-failure-modes.md:21-30,69-91` |
| 5 | 五键门 0.0125 ≈ unpaired 0.0150，保护来自显著性检验 | [内] | `paper_gate_review_r2/main.tex:63-69` |
| 6 | Dream-RSI 号-题一致，2026-09-14 提交 | [外] | [arXiv:2609.14858](https://arxiv.org/abs/2609.14858) |
| 7 | AIDE² 100 步/8 天/9-of-10 拒/public-private split/reward-hack 63→34%/ignition 无强证据 | [外] | [Weco AIDE² 博客](https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement)；[arXiv:2609.26457](https://arxiv.org/abs/2609.26457) |
| 8 | RSI 九维分类学主干 | [外] | [arXiv:2610.14625](https://arxiv.org/abs/2610.14625) |
| 9 | CODA skill-policy 共进化陷阱抹掉 52–93% 增益 | [外] | [arXiv:2610.20845](https://arxiv.org/abs/2610.20845) |
| 10 | 145 条 DGM 修改无 target-specific advantage | [外] | [arXiv:2610.14662](https://arxiv.org/abs/2610.14662) |
| 11 | 正文恢复五级证据链（路径/血缘/接口/编译/hash）+ PROVEN 非 CIRCULAR | [内] | `notes/2026-11-18-auditable-rsi-recovery.md:32-44` |

## 来源分区
### 内部资料（15）
- `stall_detector.py`（根目录 + `papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/worktree/untracked/stall_detector.py`）
- `dream-rsi-fig5-data.json` / `dream-rsi-fig5-reconstructed.png`
- `Dream-RSI.pdf`（36 页，sha256 `5701649c…`）
- `notes/2026-11-18-rsi-landscape.md`（188 行）
- `notes/2026-11-18-rsi-failure-modes.md`（117 行）
- `notes/2026-11-18-auditable-rsi-recovery.md`（110 行）
- `recovered/auditable-rsi/`（47 文件整投稿包，`main.tex` sha256 `a40d6df5…`，`main.pdf` 37 页）
- `paper_gate_review_r2/main.tex` / `TRANSFER_RECORD.md` / `CLAIM_LEDGER.md`
- `.agent-memory/grade55/meta.json:64-65` / `digest.md:18-21` / `extract_meta.py:24-52`
- `q-one-life/artifacts/research-factory/v2/review-packets/qf-auditable-recursive-self-improvement-0-2-e2b788577f.json`
- `q-one-life/artifacts/research-paper-portfolio/v1/paper-portfolio.md:90` / `research-factory/v2/triage.md:86` / `agent-dispositions.md:101`
- `papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/worktree/untracked/analyses/agent-landscape-2026-10/report.md:101-112` / `readers/03-self-evolve.md`
- `论文总结_2026-11-01.md:77`
- `harnesspro-dig/paper-skeleton-test-quality.md:26`
- `aidless-maintenance/patches/H1-seed-readme/bi-level-pepa/README.md`（Status: reserved 空槽，非本稿）
- `ci-delivery/workflows/bi-level-pepa.yml`

### 公开信息（9）
- [Dream-RSI: Recursive Self-Improvement through Evolving Worlds（arXiv:2609.14858）](https://arxiv.org/abs/2609.14858)
- [Recursive self-improvement of AI research agents（arXiv:2609.26457）](https://arxiv.org/abs/2609.26457)
- [AIDE²: The First Evidence of Recursive Self-Improvement（Weco AI, 2026-07-14）](https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement)
- [4 Levels of Recursive Self-Improvement（Weco AI）](https://www.weco.ai/blog/4-levels-of-recursive-self-improvement)
- [From Existing Methods to New Research Ideas: Two Perspectives on RSI（arXiv:2610.14625）](https://arxiv.org/abs/2610.14625)
- [Evolving Together, Failing Apart: The Skill-Policy Co-evolution Trap（arXiv:2610.20845）](https://arxiv.org/abs/2610.20845)
- [Does Failure Feedback Direct Agent Self-Modification?（arXiv:2610.14662）](https://arxiv.org/abs/2610.14662)
- [More Methods, Better Agents? A Reality Check on Self-Distillation（arXiv:2610.19789）](https://arxiv.org/abs/2610.19789)
- 其余 35 篇验号清单见 `notes/2026-11-18-rsi-landscape.md:21-63`

## 研究方法（使用外部检索 [外] 时填写）
- 子问题：① 用户手上有哪些 RSI 产出、各自状态；② 2610.* RSI 领域到了哪个阶段；③ 自有产出的证据强度与失败模式；④ Auditable RSI 的 P1/P2/P3 能否判真伪；⑤ 丢失的正文能否找回并验证身份。
- 检索查询：本地 3 子代理并行检索；主会话 web_search 1 组 + web_fetch 深读 3 篇（2609.14858 abs、2609.26457 abs、Weco 博客全文）；正文恢复走 dsh 会话/备份搜索 + sha256 对账。
- 唯一来源：landscape 35 篇 + failure 5 篇（均号-题一致）+ 主会话独立核实 3 个来源；正文恢复 5 个定位路径 + 四份哈希。
- 深读篇数：Top 3（WebFetch 全文）；其余仅作交叉印证。
- 源优先级：arXiv abs 页 / 官方博客 / 原始副本 sha256 > 落盘摘要台账（B 级）> 推断（L4）。
