# RSI / self-improving agent 开源项目调研（2026-11-19 实测）

> 任务：为 rsi-agent（纯 Python 双层：inner 改策略 + outer 密封闸）找可对标的开源项目与论文实现。
> 方法：platform_search(github) 召回候选 → `gh api repos/<owner>/<name>` 读结构化元数据（star/pushed_at/language/archived/description，读数时间 2026-11-19 08:0x UTC，标"约"）+ README 全文 → web_fetch 逐条验活 HTTP 200。
> 纪律：所有 star 数来自 GitHub API/页面实测；验不到的如实标。单一来源标"未验证"。

## 结论（最值得对标的 4 个）

1. **ThinkFlowLab/agentdescent**（约 220★，2026-11-18 仍在推）——「diff 即梯度、aggregator 即优化器」的代理级梯度下降，带 **held-out gate** 判提升、离线 demo 10 秒可跑。这是全表里与 rsi-agent 的"非循环评分 + 确认闸"设计**同构度最高**的一个，最值得直接对表。
2. **Arvid-pku/Godel_Agent**（约 240★）——学术上最直接的 RSI 参考实现：自指 Agent 通过 self-awareness→self-modification 循环改自己的 policy 代码，reward 由外部函数给。rsi-agent 的 outer 闸可直接对标它的"外部 reward + 自改代码"结构。
3. **AetherLabsAI/RSIAgent**（约 610★，2026-10 仍活跃）——训练无关的多 agent RSI 框架，**Verifier 独立于 Actor**（不能读 Actor 私有推理/记忆），BRS→DRS→冻结记忆测试期复用。它的 Actor/Verifier 隔离恰是 rsi-agent 密封闸的工程化大尺度版。
4. **Gen-Verse/ScienceBuddy**（约 430★，2026-09 发布）——"递归套递归"：inner 改 harness + outer 训模型（GRPO），双层循环与 rsi-agent 的 inner/outer 双层结构直接对应，且有冻结 train/val/test 划分、verifier 计分，可借鉴其"两层各固定一层的交替更新"协议。

## 实测表（13 个验活 repo + 3 个未验证）

| repo | URL | star（约） | 最后 push | 活跃度 | 核心机制 | 本地可跑 | 可借鉴点 |
|---|---|---|---|---|---|---|---|
| Voyager | https://github.com/MineDojo/Voyager | 7333 | 2024-04 | 停更（35 commits） | 自动课程 + 技能库 + 环境反馈迭代改进代码 | Python+JS；需要 Minecraft/游戏环境与 GPT-4 API key | 技能库/失败经验沉淀；可借鉴其"错误反馈→程序改进"闭环 |
| Reflexion | https://github.com/noahshinn/reflexion | 3435 | 2025-01 | 低（学术归档） | 口头强化学习：失败轨迹→反思记忆→重试 | Python；需要 OpenAI key | 反思记忆是 rsi-agent 公开反馈的互补形式 |
| aiwaves-cn/agents | https://github.com/aiwaves-cn/agents | 5967 | 2024-09 | 停更 | 符号学习：语言 loss→反向传播→语言梯度更新 prompt/工具/节点 | Python，pip install；需要 LLM API | 把"策略更新"形式化为可回传的 loss/gradient 语言；rsi-agent 的权重更新可类比 |
| self-refine | https://github.com/madaan/self-refine | 851 | 2024-10 | 停更（学术） | 自我反馈→自我改进→迭代 | Python；需要 OpenAI key | 最简 RSI 基线，适合当对照实验的最小实现 |
| RSIAgent | https://github.com/AetherLabsAI/RSIAgent | 614 | 2026-10-02 | 活跃 | 训练无关：Actor 探索 + Verifier 独立验证 + Curriculum 提项目 + 冻结记忆 | Python（需 FreeCAD 等软件环境）；需要 LLM API | **Actor/Verifier 隔离** = rsi-agent 密封闸的工程版；BRS/DRS 两阶段探索协议 |
| ScienceBuddy | https://github.com/Gen-Verse/ScienceBuddy | 426 | 2026-09-17 | 活跃 | 递归套递归：inner 改 harness 程序 + outer GRPO 训模型 | Python；实验用 Qwen3.5-4B + 冻结 715/90/90 划分；需要 GPU/训练 | 双层交替更新协议；冻结 val/test + verifier 计分的评估纪律 |
| Godel_Agent | https://github.com/Arvid-pku/Godel_Agent | 243 | 2025-09 | 中 | 自指 Agent：self-awareness→self-modification→改自己 policy 代码 | Python；需要 OpenAI key（key.env） | 与 rsi-agent 同构度最高的学术参考实现 |
| agentdescent | https://github.com/ThinkFlowLab/agentdescent | 220 | 2026-11-18 | **非常活跃** | diff=梯度，N 个 worker 并行演化 prompt/skill/代码，aggregator 合并，held-out gate 判提升 | Python≥3.9；`agentdescent demo` 离线无 key 10s；支持 Claude Code/Codex/DSH 等宿主 | held-out gate 与 rsi-agent 密封确认闸同构；无 API key 的离线 demo 设计 |
| RSIGame | https://github.com/WenyiWU0111/RSIGame | 173 | 2026-10-30 | 活跃 | 局部 explore-diagnose-improve 循环 + 全局质量监控/最优 checkpoint/饱和检测 | Python；需要模型 API key + 图像生成 key；Godot/Phaser 游戏引擎 | 内外双循环 + 演化 checklist；饱和检测与最优 checkpoint 保留 |
| meclaw | https://github.com/mmeyerlein/meclaw | 45 | 2026-11-17 | **非常活跃** | Rust 框架让 agent 构建 agent：ontology 基元 + 审计 + 内核沙箱 | Rust；`curl start.sh` 一键；需要 OpenRouter key | agent 可自审计的结构化基元设计 |
| Geneclaw | https://github.com/Clawland-AI/Geneclaw | 46 | 2026-02 | 低 | 自演化协议 GEP：记录失败→提出修复→gatekeeper→应用；5 层安全闸 | Python≥3.11；pip install -e；需要 LLM API | 五层 gatekeeper 是 rsi-agent outer 闸的直接对标物 |
| mmtmn/Darwin-Godel-Machine | https://github.com/mmtmn/Darwin-Godel-Machine | 28 | 2026-01 | 低 | DGM 实现：本地 Ollama 提议自改代码→测试评估→保留改进 agent | Python3.10+；Ollama 本地运行，**无 API key** | 离线无 key 的自改进循环，适合本机直接复现 |
| lemoz/darwin-godel-machine | https://github.com/lemoz/darwin-godel-machine | 16 | 2026-07 | 中 | DGM 论文实现：多 LLM 支持 + 沙箱执行 + 群体演化 + 基准 | Python3.8+；需要 LLM API；沙箱 | 群体演化 + 沙箱 + 冻结突变重放验证（held-out） |

**未验证（平台搜索结果，未做 HTTP 200/API 核验）**：lobehub/awesome-rsi（约 1k★，2026-10-15 活跃，README 已读：RSI 研究地图/分类 taxonomy）、ebiose-ai/ebiose（108★，Inria 衍生，Genomic Engine 代码未公开，2026-11 才发布）、sqliteai/adam（126★，C 库，self-evolving loops 仅作 feature）、sentrux/sentrux（3.3k★，Rust 架构传感器，非 agent 自身改进而是给 agent 当反馈回路）、keskival/recursive-self-improvement-suite（55★，任务套件，非可跑实现，偏向理论/任务设计）。

## 逐条展开

### 1. Voyager（MineDojo）— 7.3k★
- **验活**：web_fetch https://github.com/MineDojo/Voyager HTTP 200（2026-11-19）。
- **核心**：三类组件——自动课程（最大化探索）、持续增长的技能库（可执行代码存储/检索）、迭代 prompting（环境反馈+执行错误+自验证改进程序）。黑盒 GPT-4，无参数微调。
- **本地**：Python + JS；需 Minecraft 服务器/游戏 + OpenAI key。重环境，本机只适合读代码。
- **差异/借鉴**：技能库和失败经验存储可移植到 rsi-agent 的 memory 设计；但 Voyager 无任何"提升判定"纪律——它只有继续跑，没有确认闸。这恰是 rsi-agent 的差异点。

### 2. Reflexion（noahshinn）— 3.4k★
- **验活**：HTTP 200（2026-11-19）。NeurIPS 2023，口头强化学习。
- **核心**：ReAct 轨迹失败→LLM 生成"反思"文本→存入记忆→重试，直到通过。在 HotPotQA/编程任务上显著提升。
- **本地**：Python notebooks，需要 OpenAI key。学术归档，2025-01 后基本不动。
- **差异/借鉴**：Reflexion 的"反思"是**非参数**的文本记忆，无评分纪律、无提升判定；rsi-agent 的贡献在于把"反思"变成带密封确认闸的可证伪提升判定。

### 3. aiwaves-cn/agents — 5.9k★（Agents 2.0）
- **验活**：HTTP 200（2026-11-19）。
- **核心**：把 agent pipeline 类比为神经网络计算图：语言 loss（提示词实现的损失函数）→ 反传（逐节点生成文本分析/反思，称 language gradients）→ 更新（更新各节点 prompt/工具/图结构）。
- **本地**：pip install -e .，Python；需 LLM API（OpenAI 系）。
- **差异/借鉴**：符号学习把策略更新变成可审计的梯度流，形式化程度最高；但 2024-09 停更，无"非循环"处理——它的 loss 来自训练例，可能循环论证。

### 4. self-refine（madaan）— 851★
- **验活**：HTTP 200（2026-11-19）。ICLR 2024 理论 + NeurIPS 2023 实践。
- **核心**：最简 RSI——LLM 生成反馈→自我改进→迭代。7 任务 + 平均 20% 绝对提升。
- **本地**：Python，需 OpenAI key。学术归档。
- **差异/借鉴**：适合作为 rsi-agent 的"最简对照实验"基线；它没有任何防止自欺的机制（反馈与产出同一模型），正好当反面对照。

### 5. RSIAgent（AetherLabsAI）— 614★
- **验活**：HTTP 200（2026-11-19）。
- **核心**：三 agent——Actor（Python/Bash 执行）、Verifier（独立检查，**不能读 Actor 私有推理/记忆**）、Curriculum（提项目）。BRS（并行广探）→DRS（聚焦深探）→冻结记忆测试期复用。训练无关。
- **本地**：Python；需软件环境（FreeCAD 等）+ LLM API。
- **差异/借鉴**：Verifier 独立于 Actor 是 rsi-agent"密封闸"的设计先例，但 rsi-agent 更进一步（类型屏障+种子隔离+哈希链审计）。

### 6. ScienceBuddy（Gen-Verse）— 426★
- **验活**：HTTP 200（2026-11-19）。2026-09 发布（arXiv 2609.17523），带 ScienceIDE（arXiv 2609.19134）。
- **核心**：递归套递归——inner 递归（模型固定，改 Python harness 程序，父代与候选在冻结 Val 上比较）；outer 递归（harness 固定，SkyRL GRPO 训任务模型）。三层交替：harness→RL→导出→下一轮。
- **本地**：Python；实验用 Qwen3.5-4B + 715/90/90 冻结划分 + 30 GRPO updates；需 GPU。另有简化 Simple-SciBuddy。
- **差异/借鉴**：双层交替更新协议直接对应 rsi-agent 的 inner/outer；借鉴其"每层各固定一层"的协议、verifier 计分与冻结 test 的评估纪律。

### 7. Godel_Agent（Arvid-pku）— 243★
- **验活**：HTTP 200（2026-11-19）。Gödel Agent: A Self-Referential Agent Framework for Recursive Self-Improvement。
- **核心**：自指 Agent 循环——self-awareness（读自己的 policy 代码）→ self-modification（提议修改）→ action execution；外部 reward 函数对 policy 打分反馈。
- **本地**：Python + pip install -r requirements.txt；需 OpenAI key（key.env）。2025-09 后停更。
- **差异/借鉴**：学术上最直接的 RSI 实现；rsi-agent 的 outer 闸可直接对标其外部 reward 函数——但 Godel_Agent 无任何"非循环"防护（policy 可以读到 reward 实现，易循环论证），这是 rsi-agent 的核心改进点。

### 8. agentdescent（ThinkFlowLab）— 220★
- **验活**：HTTP 200（2026-11-19）。README 与 API 元数据双源一致。
- **核心**：把进化算法写成梯度下降——N 个 worker 并行演化（skill/prompt/harness/code），diff=gradient，aggregator=optimizer 合并，**held-out gate 决定提交与否**。
- **本地**：`pip install agentdescent`，`agentdescent demo` 离线、无 API key、~10s；支持 Claude Code/Codex/DeepSeek Harness 等宿主集成（MCP）。
- **差异/借鉴**：**最值得对标**。held-out gate ≈ rsi-agent 密封确认闸（同一"用没见过的数据判提升"原则）；无 key 离线 demo 让本机可直接跑对照实验。

### 9. RSIGame（WenyiWU0111）— 173★
- **验活**：HTTP 200（2026-11-19）。2026-10-30 仍活跃。
- **核心**：局部 explore-diagnose-improve 循环（广探可执行游戏、诊断问题、证据驱动修订 + 演化 checklist）+ 全局循环（质量监控、最优 checkpoint、饱和/回归检测）。
- **本地**：Python≥3.11；需模型 API key + 图像生成 key；Godot/Phaser 引擎；GameCraft-Bench。
- **差异/借鉴**：饱和检测与最优 checkpoint 保留、演化 checklist 的持续积累——可移植到 rsi-agent 的 runner 编排。

### 10. Geneclaw（Clawland-AI）— 46★
- **验活**：HTTP 200（2026-11-19）。
- **核心**：基于 HKUDS/nanobot 的 GEP 协议——观察失败→LLM/heuristic 提修复→5 层 gatekeeper（含 secret scan、diff 检查）→应用。
- **本地**：Python≥3.11，pip install -e .；需 LLM API。
- **差异/借鉴**：5 层 gatekeeper 是 rsi-agent outer 闸的直接对标物，但偏向"安全"（secret scan/redaction），rsi-agent 偏向"非循环"（密封集/类型屏障），方向互补。

### 11. Darwin-Godel-Machine ×2（mmtmn / lemoz）— 28★ / 16★
- **验活**：双双 HTTP 200（2026-11-19）。
- **核心**：mmtmn 版——本地 Ollama 提议自改代码→测试评估→归档改进 agent，**完全离线无 API key**；lemoz 版——DGM 论文（arXiv:2505.22954）完整实现，多 LLM 支持、群体演化、沙箱执行、冻结突变重放验证（held-out 12 题 +507 tests），其 README 自证"模型自改 agent 代码可迁移到未见问题"。
- **本地**：mmtmn 版 Python3.10+Ollama 本机可跑；lemoz 版需 LLM API + 沙箱。
- **差异/借鉴**：群体演化+归档（保留所有有效 agent）；lemoz 的 held-out 冻结突变重放是 rsi-agent 确认闸的又一独立实现。DGM 论文是 rsi-agent 最直接的学术参照。

### 12. meclaw（mmeyerlein）— 45★
- **验活**：HTTP 200（2026-11-19）。2026-11-17 仍活跃。
- **核心**：Rust 框架"agent 构建 agent"——ontology 基元（voice cell/web cell/vault）、审计日志、每 cell 内核沙箱。
- **本地**：Rust + `curl start.sh` 一键安装；需 OpenRouter key。
- **差异/借鉴**：结构化可审计基元 + 沙箱隔离的设计哲学与 rsi-agent 的 audit 哈希链同源。

## 缺口清单（没找到 / 未验证的）

1. **AIDE/Weco 开源部分**：platform_search 两轮无结果；Weco-AI 仓库状态未验（未列入表，避免硬凑）。
2. **Self-Refine 的"更活跃"实现**：主流实现均已归档（2024 停更），无维护中的替代。
3. **DARWIN 官方代码**：DARWIN Godel Machine 论文（2505.22954）无官方 repo，只有社区实现（mmtmn/lemoz）。
4. **meta-agent / 自我评测工具专项**：搜索遇 GitHub API 403 限流，此方向未单独深挖——有 awesome-rsi 的 Self-Verification 章节可后续按图索骥。
5. **未验证的 5 个**（见实测表"未验证"节）：ebiose（代码未公开）、sentrux（定位不同）、adam（非 RSI 核心）、keskival suite（任务套件非实现）、awesome-rsi（聚合列表，非项目）。
