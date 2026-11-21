# RSI 失败模式 / 可证伪性 / 反面结果（2026-11-18）

> 任务：为「RSI 深入 researched」子块交付失败模式分类、检测器复核、P1/P2/P3 可证伪性审查与证据边界。
> 工作目录根名含字面反斜杠（`<WORKSPACE>`），全文路径一律用相对路径（AGENTS.md 第 15 条）。
> verifier 纪律：每个「确认」先写判据；同一事实最多两种独立取证（AGENTS.local.md 二/九）。

## 零、结论先行

1. **检测器复核通过**：`stall_detector.py --self-test` 11/11；Fig5 四臂复现 `flagged` = [false, false, true, true]（guided 两臂 3/3 trip，unguided 两臂 0/3）。**但阈值是在这同一四臂上校准的（in-sample），这是复现校准、不是独立验证集证据**。
2. **"guided 臂 front-load"成立；"unguided 臂 late-wake"只是描述性读数**：检测器对 unguided 两臂只输出 `ok: no stall signature`，从不打 "late-wake" 标签；late-wake 只能从 `max_gain_at`（0.492 / 0.889）反读。检测器**默认拒绝归因原因**（`CAUSE UNATTRIBUTED`）。
3. **P1/P2/P3 现有证据都不足以判真伪**：P2 只有 synthetic（R²≈0.9957）＋ 相关证据，缺真实 lineage 和"去掉私有分"消融；P3 的"等价"强于 AIDE² 博客能支撑的"相关"；P1 缺 ignition test 的噪声下界实测。
4. **新发现一条自否定实测**（lead 补充，已核 `paper_gate_review_r2/main.tex:63-69`）：五键门 false-promotion 0.0125 与 unpaired test 0.0150「无差别」，作者明说保护来自显著性检验本身、不来自五键结构——这是"结构复杂度无增益 / 仪器自证"的典型反面结果。
5. **无 git commit/push**（遵守 AGENTS.md 第 3 条与本次纪律）。

---

## 一、RSI 失败模式分类表

判据约定：「已实测」= 本地可复算或原文给出可复算统计；「论文声称」= 外部一手但未本地复算；「推断」= 本次分析归纳。

| # | 失败模式 | 机制 | 检测判据（可操作） | 落盘证据 / 外部来源 | 是否已实测 |
|---|---|---|---|---|---|
| F1 | **front-loading / wasted-tail（前置灌满 + 长尾停摆）** | 语义引导把收益集中在预算前段，之后陷入长停摆；不是全程更差，而是"早快后死" | `t50<0.42` ∧ `longest_stall>0.20` ∧ `max_gain_at<0.37`，三取二即 flag | `stall_detector.py`（根目录 + `papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/worktree/untracked/stall_detector.py`）；`dream-rsi-fig5-data.json`；arXiv:2609.14858 §5.1 Fig5 | **本地实测**（本次 11/11 自检 + 四臂复现，见 §二） |
| F2 | **skill-policy 共进化陷阱** | 共享的 cooperative acceptance 把策略能力绑定在"共同训练出的那份技能库"上；技能漂移后策略脆弱，且随共进化训练加剧 | 用 CODA 协议：语义保持改写的一致率、corrupted skill 抵抗率、worst-case success；判 worst-case 相对 delivered peak 的落差 | arXiv:2610.20845（web_fetch 验号：题名一致，2026-10-14）；`.../analyses/agent-landscape-2026-10/report.md:103`。原文：worst-case success 比 peak 低 5–13 点，抹掉 delivered bank 相对 no-skill baseline 增益的 52–93% | 论文声称（外部一手，未本地复算） |
| F3 | **reward hacking / 私有分不可信** | 内层只见 public score，若外层用同一可见分选择，inner 会刷分；public-private split 是为阻断这条 | KernelBench：声称 speedup 在端到端 workload 中 <50% 存活即计 hacking；看 hacking rate 随版本的趋势 | Weco 博客 `https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement` §2.3（HTTP 200 原文）：AIDE0 63% → AIDE47 42% → AIDE85 34%，人工 AIDEhuman 42%；arXiv:2609.26457（lead 已验号）。**注意博客只说 "a possible explanation"，是相关不是因果** | 论文声称（外部一手，数字已取原文） |
| F4 | **verifier 退化 / self-evaluation coupling** | 自评与被测共享模型/格式/温度代理， judge 自身可被捕获（注入/模仿），"验证器"失去信息量 | ① 用 permutation-null 上的 false-promotion rate 判 gate 是否真有鉴别力；② 对照 unpaired 同 α 基线，若门≈基线则结构无增益；③ verifier 捕获模拟（N/f 固定）看是否 "never loses" 被推翻 | `paper_gate_review_r2/main.tex:63-69,477-506`（本地自否定实测）；`.agent-memory/grade55/meta.json:243,288`（verifier ensemble / trust-weighted voting synthetic，"No end-to-end deployed verifier pipeline was tested"）；arXiv:2611.22299 精读 `notes/2026-11-18-agent-harness精读-self-improving-harnesses.md`（evaluator tampering 列为头号自欺路径） | **本地实测**（门数字）+ 论文声称（verifier 模拟） |
| F5 | **测集污染 / 记忆泄漏** | 检索到的 skill-cache 条目泄漏进自报置信度与 ECE；自改进若用这些置信度做信号，等于用被污染的量做选择 | 配对 ΔECE 协议：注入 "confident-error" skill 条目，看三/四模型 ECE 变化、Bonferroni 显著性 | `.agent-memory/grade55/meta.json:65,403,945`（CALIBRATION_MERGED / tmlr-p19：+0.047 qwen3:4b p=1.6e-5、+0.035 qwen2.5-coder:7b p=0.024、+0.012 qwen2:7b p=0.013；ranking +0.067 p=0.007；第四模型反号）；`report.md:96` 的 2610.1443 MemForm（**未单独验号**）；`report.md:99` 的 2610.10265 Stale/Misattributed/Late（**未单独验号**） | 论文声称 / 用户自报（本地摘要，未复跑实验） |
| F6 | **scaffold-vs-checkpoint 混淆** | 把"改提示/编排脚手架"当成"改模型权重/checkpoint"来宣称自改进；两者收益、成本、归因完全不同 | 明确改动层级；同模型换 scaffold 前后分别测，禁止把 scaffold 增益写成模型增益 | arXiv:2609.14858 §5.1 原文（pdftotext 本地抽取）：Guidance 臂是 **"abstract prior trajectories into high-level directional insights, directly injected into the prompt as explicit semantic guidance"**，Dream-RSI 本体是 **"lightweight orchestration layer ... leaving the underlying coding agent unchanged"**；arXiv:2611.22901（web_fetch 验号）：8 模型 ×2 scaffold、~6000 run，"事后可见节省 <1%~57%" 随模型与 scaffold 变化 | **本地实测**（原文抽取）+ 论文声称（2611.22901） |
| F7 | **结构复杂度无增益 / 测量仪器自证**（lead 补充） | 门/结构做得很复杂，但鉴别力其实来自其中一个平凡组件；不做组件消融就把整体归功于结构 | 对同一 permutation-null 数据做 decision-procedure 消融：gate vs 单显著性检验 vs point-estimate vs best-of-8 | `paper_gate_review_r2/main.tex:63-69`：0.0125（五键门）/ 0.4125（point）/ 0.9825（best-of-8）/ **0.0150（unpaired 同 α）**，原文："the measured protection comes from the significance test itself, not from the five-key structure" | **本地实测**（从 tex 读，可复算于该稿表 R1/R2/R7） |
| F8 | **失败反馈未必定向** | 选择后收益不等于"针对目标失败的定点修复"，可能是旁观收益或运气 | 比较"随机选中的目标失败"修复率 vs "父代其他失败"修复率（targeted vs other） | arXiv:2610.14662（web_fetch 验号，题名一致，2026-10-10）：145 个 DGM modifications **无可检测 target-specific advantage**；受控实验 Qwen3-4B target feedback +20pp、Phi-4-mini +7pp 不显著；"target repair did not reliably imply held-out improvement" | 论文声称（外部一手，数字取自摘要原文） |

补充反面结果清单落盘位置：`papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/worktree/untracked/analyses/agent-landscape-2026-10/report.md:101-112`（含 2610.20845 / 2610.14662 / 2610.19789 等，其中 20845/14662/19789 本次已 web_fetch 验号号-题一致）。

## 二、检测器复核

### 2.1 判据（verifier 先行）

- **V1（自检有效）**：`python3 stall_detector.py --self-test` 输出 `SELF-TEST PASSED (11 checks)` 且 exit 0 —— 已满足。
- **V2（四臂复现）**：对 `dream-rsi-fig5-data.json` 每臂用 `--pairs` 喂入、`--json` 落盘，**直接读 `flagged` 字段**（不猜字段名）—— 已满足。
- **V3（字段语义核对）**：读源码 `stall_detector.py:140-160` 的 `verdict()` 确认 `flagged = len(trips) >= 2`，阈值 `THRESHOLDS = {t50:0.42, longest_stall:0.20, max_gain_at:0.37}`（`:41`），trip 方向 `TRIP_WHEN`（`:42`）—— 已满足。

### 2.2 实测结果

自检（本次实跑，exit 0）：11 项 = 4 校准臂 + 2 合成对照（front-loaded 必 flag、late-waking 必不 flag）+ 5 拒绝用例（点数不足 / 无净改进 / cost 非增 / 非数值 / 混 eval set）。

Fig5 四臂（`--json` 的 `flagged` / `tripped` / 三统计）：

| 臂 | n | t50 | longest_stall | max_gain_at | n_tripped | **flagged** | verdict 原文 |
|---|---|---|---|---|---|---|---|
| Dream-RSI | 9 | 0.492 | 0.117 | 0.492 | 0 | **false** | `ok: no stall signature` |
| Fixed Exploration | 9 | 0.667 | 0.111 | 0.889 | 0 | **false** | `ok: no stall signature` |
| Dream-RSI + Guidance | 11 | 0.308 | 0.311 | 0.225 | 3 | **true** | `shape: front-loaded/stalled -- CAUSE UNATTRIBUTED` |
| Fixed + Guidance | 8 | 0.375 | 0.375 | 0.250 | 3 | **true** | `shape: front-loaded/stalled -- CAUSE UNATTRIBUTED` |

四值逐字等于源码 `:44-47` 的 `CALIBRATION` 表，且等于自检的校准回读 —— 两条独立路径一致，按 AGENTS.local.md 二条收敛，不开第三条。

### 2.3 判定："guided front-load、unguided late-wake" 是否成立

- **guided 臂 front-load：成立**。两 guided 臂都 `flagged=true` 且 3/3 trip（t50 早、stall 长、最大单步增益早），形状签名与 Dream-RSI §5.1 描述一致。
- **unguided 臂 "late-wake"：部分成立，但需降格为描述性读数**。检测器只给出 `ok: no stall signature` + `max_gain_at` 高（0.492 / 0.889）；它**没有** "late-wake" 这个 verdict 分支（源码 `:148-160` 只有三种 flag 后标签 + 一个 ok 标签）。所以"late-wake"是我们对 `max_gain_at` 高的归纳，**不是检测器判定**。
- **关键限定**：这 4 臂正是阈值校准用的同一组数据。3/3 vs 0/3 是**复现校准**（calibration replay），不能当独立验证或泛化证据。合成对照（front/late 镜像曲线）才是门"能红"的非空性反证。

### 2.4 有效边界（能判 / 拒 / 绝不归因）

- **能判**：给定严格递增的 cost 轴和单调上升的 score 轨迹，判"收益是否前置 + 是否存在长相对停摆 + 最大单步增益是否早"。
- **拒（fail-closed）**：不传 cost/score 键名就拒（"Column/key names are NEVER guessed"）；点数<3、非数值、非有限、cost 排序后非严格增、净改进≤0、eval set 跨点变化 → `Refused`。这些在自检 5 个拒绝用例中逐条覆盖。
- **绝不归因**：默认 `CAUSE UNATTRIBUTED`；即使传 `--searches-in-stall N`，最多只能区分"停摆期搜了 0 个（没在搜）" vs "搜了 N 个仍无收益"，**仍不能归因"因为语义引导有害"这类原因**。图5 只能说明该 prompt-level guidance 在这里与 stall 签名共现，因果要另有受控实验。

## 三、可证伪性审查：P1 / P2 / P3

来源：`.agent-memory/grade55/meta.json:64-65`（用户 Auditable RSI 摘要，dir=`bi-level-pepa`，PDP/PEP/PAP 双层框架，PEPA）。摘要原文三个预测：
- P1：ignition tests 的 measurement-noise lower bound（测量噪声下界）
- P2：Level-2 RSI 需要 ungameable private score（不可作弊私有分）
- P3：emergent anti-reward-hacking behaviors ≡ evolution of audit-channel capacity（反作弊涌现与审计通道容量演化等价）

| 预测 | 现有证据 | 够不够判真伪 | 缺什么实验 | 与 2610.* 反面结果冲突? |
|---|---|---|---|---|
| **P1** 噪声下界 | ① `paper_gate_review_r2/main.tex` false-promotion 0.0125/0.0150/0.4125/0.9825（是 prompt-policy promotion gate，非 RSI ignition test）；② AIDE² 博客 §3.1 ignition test 有定性结论（discovered agent 坐进 outer loop）但无噪声注入下界 | **不够**。两个证据都不是"ignition test 的噪声下界"本体，属旁证 | 在 ignition test 上注入已知幅度测量噪声，扫 σ，测误点火率随 σ 的曲线，估计下界；预注册阈值方向/带宽/起算点（prediction-audit G5） | 无直接冲突；F7 的"门≈unpaired"提示下界可能由检验本身而非框架结构决定——**张力** |
| **P2** 私有分必要性 | ① AIDE² public-private split 设计 + reward hacking 63→34（相关）；② 用户摘要的 P2 meta-predictor：synthetic n=1000、5 模型类、10-fold、R²≈0.9957/0.9951，高于 §3.4 的 0.7 阈值；③ 摘要自承 "synthetic-data nature ... awaits AIDE85's lineage release" | **不够**。synthetic 只证"可预测性不是线性伪影"，不证"必要性"；且真实轨迹未受检。"necessity" 要消融对照 | ① 拿 AIDE85 逐版本 lineage（lead 核实：汇总统计已公开，缺的是逐版本轨迹）在真实数据上重跑 meta-predictor；② 关键消融：去掉 private score、只留 public score，看 Level-2 是否失效——AIDE² 未公开此消融 | 与 F2（2610.20845 共进化陷阱）无直接冲突，但该文显示 score 驱动选择在技能漂移下脆弱，给"ungameable private score"可推广性划边界 |
| **P3** 反作弊 ≡ 审计通道容量演化 | AIDE² §2.3：reward hacking 63%→42%→34%，博客明说"未给任何降低作弊的指令"、机制是涌现，归因措辞是 "a possible explanation"（私有分选择压力）。**没有任何变量叫 "audit-channel capacity" 被测量** | **不够，且措辞强于证据**。观测到的是"反作弊行为随版本下降"的相关；"等价"需要同一实验中同时量 audit-channel capacity 与反作弊率并证双射/共同驱动 | 在 AIDE lineage 上定义并测量 audit-channel capacity（如 verifier 数/覆盖/独立性），与 hacking rate 配对，检验等价而非相关；给出反例判据（容量升而作弊率不降则证伪） | **有张力**：2610.14662（F8）显示失败反馈在 DGM 日志里不定向、"target repair 不可靠推出 held-out 提升"——若反馈不定向，则"审计通道容量定向演化导致反作弊"的因果链存疑。但两文系统不同（DGM/指令改写 vs AIDE），属证据张力而非同一实验直接反驳 |

交叉印证：2610.19789（self-distillation reality check，"更多方法未必更好"）与 F7（五键结构无增益）方向一致——都反对"结构/方法堆料 = 收益"。三篇 2610.*（20845/14662/19789）本次已 web_fetch 验号，号-题一致；`report.md:101-112` 里其余 2610.* 条目未逐一验号。

## 四、证据边界（逐条标等级）

| 等级 | 含义 | 本文对应的条目 |
|---|---|---|
| **L1 本地复现（可复算）** | 本会话实跑命令或从本地原文逐字抽取 | stall_detector 11/11 自检；Fig5 四臂 flagged；`paper_gate_review_r2/main.tex:63-69,477-506` 的四个 rate；arXiv:2609.14858 §5.1 guidance 层级原文（本地 pdftotext 抽取） |
| **L2 外部一手（web_fetch 验号/原文）** | URL HTTP 200、号-题一致、数字取原文 | 2609.14858、2610.20845、2610.14662、2610.19789、2611.22901；Weco AIDE² 博客（63/42/34、非单调泛化、public-private split） |
| **L3 落盘二手 / 作者自报（未独立复跑）** | 本地摘要/台账里别人写的统计 | `.agent-memory/grade55/meta.json:65` 的 synthetic R²≈0.9957、0.7 阈值（作者自报 synthetic）；`meta.json:243,288` verifier 模拟；`report.md:101-112` 2610.* 清单（除已验号三篇）；`harnesspro-dig/paper-skeleton-test-quality.md:26` 的"测试质量下限判据"仍是**提出**而非实测 |
| **L4 本次推断** | 归纳/判定，非测量 | "guided front-load 成立 / unguided late-wake 是描述性读数"；检测器有效边界三分；P1/P2/P3 真伪判定与冲突判断；F1–F8 的机制归纳 |

诚实边界：
- 检测器阈值 in-sample 校准，泛化能力**未验证**；`--searches-in-stall` 未喂真实数据，故"搜索与停摆关系"未实测。
- AIDE² 的 reward-hacking 下降与 private-score 选择压力是**相关**，博客原文自限为 "a possible explanation"；P3 的等价性没有直接证据。
- 用户 Auditable RSI 摘要只有 `meta.json` 里的 abstract 与 sections 元数据（`bi-level-pepa`，words=507），**本工作区未找到该稿 main.tex 全文**（`find . -path '*bi-level-pepa*'` 仅命中 `aidless-maintenance/patches/H1-seed-readme/bi-level-pepa/README.md` 与 CI workflow 模板），故 P1/P2/P3 的 §3.4 阈值细节仅能引用摘要中出现的 0.7，未核对正文定义。
- 未对 `paper_gate_review_r2/` 的 0.0125 等数字做独立重跑，只做"原文逐字 + 表 R1/R2/R7 交叉一致"两路核对。

## 五、复算命令

```bash
python3 stall_detector.py --self-test                          # 11/11, exit 0
python3 - <<'PY'
import json, subprocess, tempfile, os
data=json.load(open('dream-rsi-fig5-data.json'))
for arm,pts in data.items():
    spec=';'.join(f'{c},{s}' for c,s in pts)
    jf=tempfile.mktemp(suffix='.json',dir='/tmp')
    subprocess.run(['python3','stall_detector.py','--pairs',spec,'--name',arm,'--json',jf],check=True)
    r=json.load(open(jf)); print(arm, 'flagged=',r['flagged'],'tripped=',r['tripped'],
                                't50=',r['t50'],'stall=',r['longest_stall'],'gain@=',r['max_gain_at'])
PY
grep -n '0.0125\|0.0150\|0.4125\|0.9825' paper_gate_review_r2/main.tex   # F7 原文
# 外部验号：web_fetch https://arxiv.org/abs/{2609.14858,2610.20845,2610.14662,2610.19789,2611.22901}
# 外部原文：web_fetch https://www.weco.ai/blog/first-evidence-of-recursive-self-improvement
```
