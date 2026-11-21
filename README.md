# RSI 研究产物：claim 级审计、可证伪框架与燃料盘点

这是递归自我改进（Recursive Self-Improvement, RSI）方向的一组研究产物，围绕一个核心问题：

> **我用 AI 做科研得到的"自我改进"结果，怎么知道不是自证（circular / self-certifying）？**

配套的可运行 agent 见 [aidless/rsi-agent](https://github.com/aidless/rsi-agent)。本仓库是它的**证据与论证层**——审计、框架稿件、文献/数据/项目盘点、以及让稿件通过科研三闸的证据链。

## 仓库结构

| 目录 | 内容 |
|---|---|
| `manuscript/` | *Auditable Recursive Self-Improvement: A PDP-style Formal Framework for Bi-Level Agent Systems* 的完整 LaTeX 源码（11 节 + 附录 + CLAIM_LEDGER + 5 图）+ `evidence/`（三闸声明/报告、复跑日志、四份真实实验 JSON、companion 表）。稿件按此前修订统一了 AUC 口径、披露了 P2 循环机制、补了可重建的 companion 表。**未投稿** |
| `notes/` | 15 份研究笔记：claim 级审计、失败模式、文献版图（四轴分类学 + 35 篇验号）、燃料盘点（11 数据集 / 15 论文 / 13 项目）、六路全文深挖（papers a/b/c + projects core/more + datasets）、范式迁移核验、分组研究成果汇总 |
| `reports/` | 4 份深度研究报告（md + HTML）：自有资产盘点、燃料盘点、六路深挖总结、claim-agent 全流程总结 |

## 三条主线结论

1. **自有资产盘点**：Dream-RSI 失败签名复现、Auditable RSI 正文找回（真源 37 页完整稿）、paper_gate_review_r2 五键门自否定实测、2610.* 领域情报——全部 unreviewed，领域已进入"可证伪反面结果"阶段。
2. **claim 级审计**（决定性）：逐条复跑 P1/P2/P3 实验后判定——**P1 真证据**（ESS 校正 power 方法学干净、可复算）；**P2 循环论证**（`mu_priv = final_public_score + N(0,σ)` 而特征第 0 维就是 `final_public_score`，AUC≈0.997 是代数恒等式）；**P3 合成演示**（注入模拟，真实 lineage ablation 仍 pending）。另发现头表 AUC 口径分叉、§A 6 个 companion 死链。
3. **回灌修订 + 三闸通过**：在稿件里披露 P2 循环机制、接入 NC 前置 model-free 审计闸设计、统一 AUC 口径、从真实 JSON 重建 2 个 companion 表；`manuscript/evidence/` 里三闸（integrity / provenance / figure-trace）**首闸 + 终闸（--final --prior）全部 PASS**，38 页重编译通过。

## 六路深挖的关键判读

- **NC（Non-Circular）威胁最大**：四篇攻击都不经过循环（乐观 verifier、判据文本泄漏、提示诱导、上游权重污染）——密封集只断算术环，断不了"评分器本身被骗"。
- **C 线**：`paired_gate` 原只验配对差均值，不查逐实例回归 → 均值过闸但少数实例被压垮也能放行（Artifact-to-Capability 属性 4）。已在 rsi-agent 里加回归负担闸修补。
- **文献正面线**：ELITE 零下游评估仍 60→85.6%、RSI Tax verification 占税 74.8–93.6%——说明"audit 成本本身"是 RSI 的一等公民。

## 诚实边界

- 稿件是**未投稿**的研究稿，作者署名保留；P2 实证仍是 synthetic，原文自述 "awaits AIDE85's lineage release"。
- 文献/项目/数据的数字**大部分是正文/摘要自报，本实验室未复算**（笔记里逐条标注证据等级）；arXiv id、GitHub repo、官方 URL 已逐一验号/验活。
- 端到端 ALE-Bench / SWE-bench 实验缺外部资源（HF 数据、docker、LLM key、GITHUB_TOKEN），本机三通道实测 BLOCKED；配套 agent 里的 pilot 用确定性替身验证闸行为，诚实边界写进产物。
- 已脱敏：宿主路径、DSH 会话 UUID、临时目录、邮箱占位符均替换为 `<...>` 占位符；未纳入第三方源码副本（184 文件，许可证归属不清）。

## 复现

科研三闸的判据与声明在 `manuscript/evidence/`（integrity_declaration.json / provenance_declaration.json / figure_trace_manifest.json + config_*.json + exit_contract_p2.md）。三闸脚本是本工作区的内部工具，未随本仓库发布；声明与报告本身自含 verdict / block_reasons / unresolved_from_first_gate 字段，可直接审阅。

可运行的验证在配套仓库：`git clone https://github.com/aidless/rsi-agent.git && cd rsi-agent && python3 run_demo.py --self-test`（41 项全过）。
