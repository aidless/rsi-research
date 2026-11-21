# 2026-11-19 RSI 研究：公开 benchmark 数据集/评测环境实测清单

> 负责人：rsi-data（子代理）｜日期：2026-11-19（Asia/Shanghai）｜任务：为 rsi-agent（上下文决策试验床）找可用于 RSI / self-improving / self-evolving agent 研究的公开评测源。
> 纪律：数字可复算、绝不编造、arXiv id 逐一 `web_fetch` abs 页验号、单一来源标「未验证」、查不到标 `[⚠ 未覆盖]`。一手来源优先（官方 GitHub README / 项目官网 / arXiv abs），Bing 中文结果噪声大已弃用。

## 结论（先看这里）

**最值得用的 3 个：**

1. **SWE-bench**（2,294 实例 / Verified 500 子集）——唯一同时满足「规模够大、每实例独立 Docker、F2P 二值打分、MIT、CPU 可跑」的标准集，且官方自带 Verified / full / Multimodal 多个切分，天然就是 rsi-agent 的 public/private split 来源。作为**主评测**最稳。
2. **MLE-Bench**（75 Kaggle 竞赛 / Lite 22）——任务本身就是「长程 ML 工程迭代」，medal 分级提供比二值更细腻的改进信号，与 RSI 的「递归自我改进」语义最贴；AIDE 用树搜索在此集上比线性 agent 多拿 4× medal，说明该集对「改进策略」敏感，适合当**自我改进曲线的主战场**。代价：需 Kaggle key、full 数据 3.3TB（Lite 158GB），自有 GPU 1×24GB 起。
3. **ALE-Bench**（40 个 AtCoder 启发式优化题）——**分数连续**（score-based）而非 pass/fail，正好量化「每一轮改进了多少」，是画 RSI 学习曲线的最佳信号源；CPU 可跑、不需 GPU，长程迭代友好。作为**细粒度改进度量**补充前两者。

**加分项**：SWE-smith（52k 合成任务实例 + 26k trajectories，MIT）是给 agent 自举生成 private split 的最佳数据源；RE-Bench 专为「AI 做 AI 研究」设计但已 DEPRECATED，仅作对照参考；KernelBench 需 GPU，当前本机零可用 GPU（见 MEMORY.md / AGENTS.md §17），暂不能本地跑。共验活 11 个数据集/环境，全部官方 URL HTTP 200。

## 实测表

证据等级：**A** = 官方一手页 + 多源交叉；**B** = 单一官方一手页（未交叉）；**C** = 未验证/数字未复算。

| 数据集 | URL（web_fetch 验活） | 规模（出处） | 本地可跑性 | 许可证 / 获取 | 与 rsi-agent 契合度 | 等级 |
|---|---|---|---|---|---|---|
| **SWE-bench** | https://www.swebench.com/ ; https://swebench.com/original.html | full **2,294** 实例（12 个 Python 仓库 PR/Issue）；Verified **500**；Multimodal v2 **480**（官网 news 2026-09-01） | CPU+Docker 可跑；每实例一个 Docker 环境；生成预测需 LLM/API key | MIT（README）；HF `princeton-nlp/SWE-bench` | ★★★★★ 天然多 split，直接做 public/private；F2P 打分客观 | A |
| **MLE-Bench** | https://github.com/openai/MLE-Bench | **75** Kaggle 竞赛；Low/Lite split **22**；full 数据 **3.3TB**、Lite **158GB**（README） | Docker+Sysbox；需 Kaggle key (`~/.kaggle/kaggle.json`)；推荐 36 vCPU/440GB/1×24GB A10/24h；`prepare --all` 约两天 | 代码 MIT（LICENSE，明确不含外部下载数据）；数据受各 Kaggle 竞赛规则约束 | ★★★★★ 长程迭代+medal 分级；用 Lite 22 做 CPU/小资源 pilot | A |
| **ALE-Bench** | https://sakana.ai/ale-bench/ ; https://github.com/SakanaAI/ALE-Bench | **40** 个 AHC 历届优化题（官网 + README 一致） | CPU 可跑（本地代码执行 + test-run 反馈），不需 GPU；需 LLM API；Docker `yimjk/ale-bench`；HF `SakanaAI/ALE-Bench` | Apache-2.0（LICENSE 已读）；arXiv 2506.09050（abs 验号，NeurIPS 2025 D&B） | ★★★★★ 连续分数 → 最能量化「每轮改进幅度」 | A |
| **SWE-smith** | https://github.com/SWE-bench/SWE-smith | **52k** task instances（README 注释「Loads all 52k」）；26k SWE-agent trajectories；250+ environments | Docker（Ubuntu 22.04，明确不支持 Win/macOS）；合成数据阶段 CPU 可 | MIT；arXiv 2504.21798（NeurIPS 2025 D&B Spotlight） | ★★★★☆ 自举生成 private split 的最佳源；非现成评测 benchmark | B |
| **RE-Bench** | https://github.com/METR/RE-Bench | 7 个 task family（README 列出），**任务总数未给** `[⚠ 未覆盖]`；含 175 CodeContests held-out 题 | 需 GPU（nanogpt RL / triton / scaling-law 等）；Vivaria scaffold；部分任务密码 zip（`resident-peacock-motif-grading`） | MIT + 非正式请求勿泄入训练数据；arXiv 2411.15114（abs 验号） | ★★★☆☆ 专为「AI 做 AI 研究」设计，最贴主题；但已 **DEPRECATED** | B |
| **Terminal-Bench** | https://www.tbench.ai/ ; https://www.tbench.ai/benchmarks ; https://www.tbench.ai/news/terminal-bench-4-0 | v4.1；4.0 删 8 题、修 19 题；统一 8h timeout。**精确任务数未取到**（Hub 页 JS 渲染）`[⚠ 未覆盖]` | Docker+Modal；`uv run harbor run -d terminal-bench/terminal-bench@latest --agent oracle`；需 agent/model | Apache-2.0（LICENSE 已读） | ★★★★☆ 终端任务、harness 可私有化；数字待补 | B |
| **MLAgentBench** | https://github.com/snap-stanford/MLAgentBench | **13** 个 ML 工程任务（README） | Docker 镜像 `qhwang123/researchassistant:latest`；需 Kaggle key + LLM key（OpenAI/Claude/CRFM/Gemini/HF） | 许可未在抓取片段确认 `[⚠ 未覆盖]`；arXiv 2310.03302 | ★★★☆☆ 自带 starter code，适合测「改进幅度」；规模小 | B |
| **ScienceAgentBench** | https://github.com/OSU-NLP-Group/ScienceAgentBench | **102** tasks，来自 44 篇论文、4 学科、9 位领域专家验证（README） | CPU 可（102 实例/8 线程/≤30min）；完整数据需 SharePoint + 密码 `scienceagentbench`，要求勿在线重分发；HF 仅 annotation sheet | 任务 CC-BY-4.0、代码 MIT（保留 rasterio/matminer 原许可）；arXiv 2410.05080（abs 验号） | ★★★★☆ 数据科学任务、有 verified split；获取有摩擦 | B |
| **aira-dojo**（Meta） | https://github.com/facebookresearch/aira-dojo | 实现了 MLE-bench 任务；官方称曾支持 1,000 agent 并行/120h。**独立任务数未给** `[⚠ 未覆盖]` | 支持 Slurm；需 LLM API；可扩展自定义 task | CC-BY-NC-4.0（badge，非商业）；arXiv 2507.02554（abs 验号） | ★★★★☆ 专为「hill-climbing 研究能力」设计，最贴 RSI；但 NC 许可需注意 | B |
| **KernelBench** | https://github.com/ScalingIntelligence/KernelBench | L1 **100** + L2 **100** + L3 **50** = **250**（README）；L4 整模型条数未给 `[⚠ 未覆盖]` | **需 GPU**（CUDA/DSL + 目标 GPU 上测 speedup）；本机零 GPU 暂不可跑 | MIT（LICENSE 已读）；HF `ScalingIntelligence/KernelBench`；ICML'25，arXiv 2502.10517 | ★★☆☆☆ 改进信号清晰（speedup）但需 GPU，当前环境受限 | B |
| **AIDE (aideml)** | https://github.com/wecoai/aideml ; https://arxiv.org/abs/2502.13138 | agent 实现，非独立数据集；自带 example_tasks（house prices）；评测复用 MLE-Bench 75 竞赛 | `pip install aideml`；需 OPENAI_API_KEY；CPU/GPU 取决于任务 | MIT（badge）；arXiv 2502.13138（abs 验号，2025-02-18 v1） | ★★★☆☆ 作为「树搜索式自我改进」对照基线；评测集= MLE-Bench，勿重复计 | B |

## 逐条判断

### 1. SWE-bench — 主评测首选 [A]
- 规模可复算：官网 original.html 明写「2,294 task instances，来自 12 个流行 Python 仓库」；Verified 页 500 条人工验证子集；官网 news 页 2026-09-01 的 Multimodal v2 = 480 tasks 全开源。三处数字均取自一手页。
- 环境：每实例一个 Docker 镜像、以 Fail-to-Pass 测试判定，打分客观、无 judge 主观性——对 RSI 的「改进是否真实」最关键。
- 契合：Verified(500)/full(2294)/Multimodal(480)/Lite 多个官方切分，可直接把 Verified 当 public、full 中剔除后的池子当 private，避免自我改进「记住测试题」。

### 2. MLE-Bench — 长程改进主战场 [A]
- README 明写 75 竞赛评测集；Low complexity split = 22（Lite）；full 3.3TB / Lite 158GB，数字均可复算。
- 环境：`mlebench prepare --all` 从零约两天；官方推荐 36 vCPU/440GB/1×24GB A10/24h——**本机不满足**，建议只用 Lite 22 + 云通道跑，或仅用其竞赛定义/评分脚本。
- 许可陷阱：代码 MIT 明确「不适用于外部下载的数据集」，数据受各 Kaggle 竞赛规则约束——落笔记为「获取方式受限」，不写成无条件 MIT。
- 契合：medal 分级（bronze/silver/gold）比二值更适合画改进曲线；AIDE 论文/README 实测树搜索赢线性 agent 4× medal，证明此集对「改进策略」敏感。注意 leaderboard 自 2026-04-24 暂停新提交（v2 开发中），不影响本地评。
- arXiv 2410.07095 abs 页验号一致（v1 2024-10-09，v6 2025-02-26）。

### 3. ALE-Bench — 细粒度改进度量 [A]
- 官网 + README 一致给出 40 题（历届 AHC 启发式优化题）；arXiv 2506.09050 abs 页验号一致（NeurIPS 2025 Datasets & Benchmarks Track）。
- 环境：本地代码执行 + test-run 反馈 + 可视化，支持交互式 agent；CPU 可跑、不需 GPU；Docker `yimjk/ale-bench` + HF `SakanaAI/ALE-Bench`。需 LLM API。
- 契合：**score-based 连续分数**是它相对 SWE-bench（二值）的核心优势——RSI 每一轮「改进了多少」可直接读分数差，而不是只有过关/不过关。长程迭代题型也贴 self-improvement。

### 4. SWE-smith — private split 自举源 [B]
- README 注释「Loads all 52k task instances」可复算；另有 26k trajectories、250+ environments。
- 它是「把任意 GitHub 仓库转成训练环境并合成无限任务」的工具，本质是**训练数据生成器**而非现成评测 benchmark。
- 契合：最适合给 rsi-agent 自动扩充 private split（合成任务 + 真实仓库），避免公开集污染；需要 Docker（Ubuntu 22.04，明确不支持 Windows/macOS）。
- 单一官方源（README），标 B；arXiv 2504.21798 未在本次会话单独 web_fetch abs 验号 → 号码来自 README，标「未验号」。

### 5. RE-Bench — 贴主题但已弃用 [B]
- 7 个 task family（rust codecontests / nanogpt chat RL / fix embedding / optimize LLM foundry / triton cumsum / small scaling law / restricted MLM）；**任务总数 README 未给** → `[⚠ 未覆盖]`。
- README 顶部明确 **DEPRECATED**，不再接受贡献/bug 报告——作为活跃评测源风险高，仅作历史对照。
- 多数任务需 GPU（RL finetune、triton kernel、5e17 FLOPs scaling law）；MIT + 非正式请求勿泄入训练数据；部分任务 zip 密码 `resident-peacock-motif-grading`。
- arXiv 2411.15114 abs 页验号一致。

### 6. Terminal-Bench — 终端评测，数字待补 [B]
- 当前 v4.1（连续 tagged releases）；4.0 新闻页明写删 8 题/修 19 题、统一 8h timeout——但**精确任务总数在抓取的 Hub 页未取到**（JS 渲染 `Loading dataset details…`）→ `[⚠ 未覆盖]`，不可凭印象填数字。
- Apache-2.0；Harbor framework + Modal。契合度好（终端任务、harness 私有化），但落地前须补任务计数。

### 7. MLAgentBench — 小规模 ML 任务 [B]
- README 明写 13 个 ML 工程任务，自带 starter code。
- 需 Kaggle key + 多 LLM key + Docker 镜像；许可证在抓取片段未见 → `[⚠ 未覆盖]`。arXiv 2310.03302 未在本次会话 abs 验号（来自 README）。

### 8. ScienceAgentBench — 数据科学任务 [B]
- README：102 tasks、44 篇论文、4 学科、9 位领域专家验证；容器化 harness 102 实例/8 线程/30min 内跑完 → CPU 可跑。
- 获取摩擦：HF 只给 annotation sheet（防污染），完整 benchmark 需 SharePoint + 密码 `scienceagentbench`，要求勿在线重分发。
- 许可：任务 CC-BY-4.0、代码 MIT（保留 rasterio Instance 32/46/53/54/84 与 matminer Instance 3 原许可）——已从 LICENSE 段原文复算。arXiv 2410.05080 abs 页验号一致。

### 9. aira-dojo — 研究能力 hill-climbing 框架 [B]
- Meta 出的可扩展 AI research agent 框架，实现了 MLE-bench 任务；官方称曾支持 1,000 agent 并行 120h。**独立任务数未给** → `[⚠ 未覆盖]`。
- **CC-BY-NC-4.0（非商业）**——若 rsi-agent 有商业用途需换源，落笔记为许可风险。
- arXiv 2507.02554 abs 页验号一致（v1 2025-07-03，v2 2025-11-04）。

### 10. KernelBench — 需 GPU，当前受限 [B]
- README：L1=100、L2=100、L3=50 共 250（可复算）；L4 整模型未给数 → `[⚠ 未覆盖]`。
- 评分是在目标 GPU 上测 speedup，**必须 GPU**；当前本机零可用 GPU（AGENTS.md §17 / MEMORY.md），云服务亦未复核 → 暂不可跑。MIT；ICML'25；arXiv 2502.10517 未在本次会话 abs 验号（来自 README）。

### 11. AIDE (aideml) — 对照基线，非独立集 [B]
- 开源 agent（树搜索式 ML 工程），MIT，arXiv 2502.13138 abs 页验号一致（2025-02-18 v1）。
- README 明写「MLE-Bench (75 Kaggle comps) found the tree-search of AIDE wins 4× more medals than the best linear agent (OpenHands)」——证明 MLE-Bench 对改进策略敏感（支撑结论 #2）。
- **它的评测集就是 MLE-Bench 的 75 竞赛，不另算一个数据集**；作为 rsi-agent 的对照基线使用。

## 缺口清单

1. `[⚠ 未覆盖]` **Terminal-Bench v4.1 精确任务数**——Hub 页 JS 渲染未取到，需查 Harbor Hub API 或仓库 tasks 目录。
2. `[⚠ 未覆盖]` **RE-Bench 任务总数**（README 仅列 7 个 family 名）；且已 DEPRECATED，是否值得接入需 lead 定。
3. `[⚠ 未覆盖]` **KernelBench L4 条数**；**MLAgentBench 许可证**（抓取片段未见）；**aira-dojo 独立任务数**。
4. ~~**未验号的 arXiv id**~~ **已由 lead 补验（2026-11-19，全部 HTTP 200 + 号-题一致）**：SWE-smith [2504.21798](https://arxiv.org/abs/2504.21798) = "SWE-smith: Scaling Data for Software Engineering Agents" ✓；MLAgentBench [2310.03302](https://arxiv.org/abs/2310.03302) = "MLAgentBench: Evaluating Language Agents on Machine Learning Experimentation" ✓；KernelBench [2502.10517](https://arxiv.org/abs/2502.10517) = "KernelBench: Can LLMs Write Efficient GPU Kernels?" ✓。原表 §4/§7/§10 的「未验号」注记可据此升级为已验。
5. **未检索**：EuroSys / 各 systems agent bench（用户点名「自行验证，不限于此」）。Bing 中文引擎对该类查询全返回无关结果，本次用 exa + 一手页覆盖主流集，systems 类留作下一轮。
6. **环境前提**：本机零可用 GPU（AGENTS.md §17），KernelBench / RE-Bench GPU 任务 / MLE-Bench full 均需云端通道；上云前须现测 `ssh -o ConnectTimeout=5`（MEMORY.md 已记 ecs-hz / hpccube 可用但无 GPU）。
7. **单一来源条目**（标 B）：SWE-smith / Terminal-Bench / MLAgentBench / aira-dojo / KernelBench / AIDE 未做第二独立源交叉，数字为各官方 README 原文，未升级为 A 级定论。
