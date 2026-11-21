# 2026-11-19 rsi-agent 候选数据集/环境深读（接入级）

> 上一轮 `notes/2026-11-19-rsi-datasets.md` 只看官网/README 元数据；本轮读到**真实加载接口、实测规模、评分接口、许可证原文条款**。
> 全部数字带一手 URL；本机实测通道：raw.githubusercontent 200 / gh api 200，**HuggingFace 全通道超时不通**（huggingface.co、hf-mirror、datasets-server 均 000），因此凡 HF 数字只引 README/官网文字，标「HF 本机不可达、需上云复核」。本机零 GPU（AGENTS.md §17），所有 GPU 项标「需上云」。
> rsi-agent 机制口径：SP=密封私集 / NC=非循环评分 / ESS=自相关感知 ESS 配对闸 / HA=哈希链审计；paired_gate 吃**逐实例配对差 + 噪声估计**。
> 抓取物留档：`<SCRATCH>/...`。

## 结论先行

- **接入优先级前三**：① SWE-bench（客观 F2P + 天然 Verified/full split + MIT，**评分侧无需 key**，最贴四标准）；② ALE-Bench（连续分数 + 框架级 public/private 隔离，**最贴 paired_gate/ESS**；代价是需 LLM key + HF 数据上云）；③ Terminal-Bench（客观通过率 + 可切 task_id_subset；需 Docker + model key）。
- **本机零 GPU 直接降级**：KernelBench、RE-Bench、MLE-Bench full、aira-dojo 全量、AIDE 全量 —— 这些要么需 GPU，要么数据 3.3TB 级，只能上云。
- **四个旧缺口本轮全补上**：Terminal-Bench core v0.1.1 = **80** 题；RE-Bench = **7 主任务 + 175 held-out**；KernelBench L4 = **20**（总 270）；MLAgentBench 许可证 = **MIT**。详见对应小节。
- **SP/NC/ESS 适配关键**：只有 ALE-Bench 原生提供密封私集（private_seeds/standings）；SWE-bench/Terminal-Bench 需我们自己在 harness 层锁私集。二值分（SWE-bench F2P）配对差噪声大，连续分（ALE-Bench、MLE-Bench medal、RE-Bench、KernelBench speedup）更适合 ESS。

---

## 实测总表

| # | 数据集 | 实测规模 | 评分信号 | 许可证（原文出处） | key/Docker/GPU | SP/NC 适配 | 适配度 |
|---|--------|---------|---------|-------------------|---------------|------------|-------|
| 1 | SWE-bench full/Verified/Multimodal | 2294 / 500 / 480 | 二值 F2P | MIT（README badge） | 评分免 key；需 Docker；CPU | 天然 split，二值噪声大 | **高** |
| 2 | MLE-Bench | 75；Lite=22 | bronze/silver/gold + any_medal% | 代码 MIT（LICENSE）；数据另受 Kaggle 规则 | Kaggle+LLM key、Docker、重 | medal 连续，22 题偏小 | 中 |
| 3 | ALE-Bench | 40（官网/README） | 连续 overall_absolute_score | Apache-2.0（LICENSE） | LLM key；CPU 可跑；HF 上云 | **框架级 SP**，最贴 ESS | **高** |
| 4 | SWE-smith | 52k 实例 / 26k 轨迹 / 250+ 环境 | 需自建 | MIT（README badge） | Docker；CPU | 私集生成源，非现成 bench | 中 |
| 5 | RE-Bench | 7 主任务 + 175 held-out | 连续（各任务异质） | MIT（README，附保密条款） | Vivaria+GPU | held-out 可当私集，但 deprecated | 低 |
| 6 | KernelBench | 100+100+50+20=270 | 连续 speedup / fast_p | MIT（LICENSE） | GPU（或 Modal）；LLM key | speedup 好，需 GPU | 低 |
| 7 | Terminal-Bench | core v0.1.1=80（tree 有 256） | 二值测试通过 | Apache-2.0（LICENSE） | Docker+model key；CPU | 可切 task_id_subset | 中 |
| 8 | MLAgentBench | 13 | 任务指标 | **MIT（LICENSE，本轮补）** | Docker+Kaggle+LLM key | 规模太小 | 中低 |
| 9 | ScienceAgentBench | 102 / 44 论文 / 4 学科 | 多指标 + GPT-4o judge | 任务 CC-BY-4.0，代码 MIT | Docker+OpenAI key；CPU | verified split 可用，获取有摩擦 | 中 |
| 10 | aira-dojo | 任务数未给（缺口） | 任务指标 | **CC-BY-NC-4.0（非商业）** | LLM key；Slurm | lite/low split，NC 许可风险 | 中 |
| 11 | AIDE (aideml) | agent 实现，复用 MLE-Bench 75 | AUROC 等 | MIT（LICENSE） | OPENAI key；CPU/GPU | 作对照基线，勿重复计数 | 中 |

---

## 1. SWE-bench（full / Verified / Multimodal）

### 核实后的规模与许可证
- **full = 2294 实例**，官网原文（实测 200）：https://www.swebench.com/original.html —— “We collect 2,294 task instances ... from 12 popular Python repositories”。
- **Verified = 500**，官网原文（实测 200）：https://www.swebench.com/verified.html —— “human-validated subset of 500”。
- **Multimodal**：官网 https://www.swebench.com/multimodal.html 原文 “originally 517 issues ... V2 retains 480 tasks”（页面标注 2026-09-01）→ 现 480。
- **许可证 MIT**：README badge，https://github.com/SWE-bench/SWE-bench （raw main 200：https://raw.githubusercontent.com/SWE-bench/SWE-bench/main/README.md）。

### 真实加载接口
- pip：`pip install -e .`（仓库根）；加载代码：
  ```python
  from datasets import load_dataset
  swebench = load_dataset('princeton-nlp/SWE-bench', split='test')   # full
  ```
- CLI v5（均已从 README 核对）：
  ```bash
  swebench eval verified --gold --task-repo ./swe-bench-tasks          # gold 自检
  swebench eval verified -p <preds.jsonl> --run-id <id> -j <n>         # 评自己的预测
  swebench report <run_id> -d verified                                 # 出分（不需容器）
  swebench infer verified -m gpt-5 -o preds -w 8                       # agent 生成（需 key）
  ```
  `DATASET` 接受别名 `full/verified/multimodal/multilingual` 或 HF id。
- **资源**：x86_64、≥120GB 空闲磁盘、16GB RAM、8 CPU；`max_workers < min(0.75*cpu, 24)`。
- **缓存陷阱**：结果按 `run_id + instance_id` 缓存；改预测必须换 `run_id`，否则复用旧结果。
- 数据 HF 页 https://huggingface.co/datasets/princeton-nlp/SWE-bench （本机不可达，README 文字引数）。

### 最小 CPU pilot（评分侧免 key）
```bash
pip install -e .                                             # 本机 CPU 装 CLI
swebench eval verified --gold --task-repo ./swe-bench-tasks  # harness 自检；需 Docker，gold 全量重，最小冒烟见 swebench eval --help 限实例
swebench report <run_id> -d verified                         # 出分，纯 CPU 免 key
```
预期：加载元数据 MB 级；gold 全量需 Docker + 数小时，pilot 只跑少量实例（用 `--help` 里的实例限制参数）。**评分/出分无需任何 key**——这是它排第一的关键。

### SP/NC 映射 + paired_gate
- split：**Verified 500 当 public**（inner 改策略调参），**full 剔除 verified 后池子当 private**（outer 密封确认）。两集同源同分布但需去重叠。
- 评分 = **二值 F2P**（每实例 resolved 与否）。逐实例配对差可做（同一 instance 两策略 resolved_A/B），但二值 → 噪声大，ESS 需更多配对实例才有把握度。
- 适配度：**高**——客观、可 split、评分免 key、CPU 可跑；唯一短板是二值噪声。

### 诚实缺口
- Multimodal V2 的 480 是官网页面文字，未逐条数 HF parquet（HF 本机不通）。
- Verified 与 full 的去重叠需自己做 instance_id 差集，官方未给「full-minus-verified」现成 split。

---

## 2. MLE-Bench（75 / Lite 22）

### 核实后的规模与许可证
- **75 个 Kaggle 竞赛**（README:139）；**Low == Lite split = 22**（README:82-84，本会话对 `experiments/splits/low.txt` grep 计数 = 22）。
- full 数据 **3.3TB** / Lite **158GB**（README:84）；`prepare --all` 约两天（README:151）；官方推荐配置 36 vCPU / 440GB 内存 / 1×24GB A10 / 24h。leaderboard 2026-04-24 起暂停新提交。
- **许可证**：代码 MIT —— LICENSE 原文 “MIT License Copyright (c) 2024 OpenAI”（README badge）。但 README:139-150 明确数据来自各 Kaggle 竞赛，需 `~/.kaggle/kaggle.json`、须接受各竞赛规则——**外部竞赛数据不受 MIT 覆盖**。
- 一手：https://github.com/openai/MLE-Bench （raw main 200）。

### 真实加载接口
```bash
mlebench prepare --all                       # 下载+建环境，full 3.3TB / Lite 158GB，约两天
mlebench grade --submission <sub.jsonl> ...  # 读 JSONL，每行一 submission
uv run python experiments/aggregate_grading_reports.py --experiment-id <id> --split low|medium|high|split75
```
- `grade` 产出 grading report，指标 **any_medal_percentage**，medal 分 bronze/silver/gold（比二值细腻）。
- 需 **Kaggle key + Docker/Sysbox + LLM key**；建议 ≥3 seeds 报 mean±SEM。

### 最小 CPU pilot
Lite split 单竞赛的 prepare + grade（**需 Kaggle key + 云端**；本机不跑大数据）：
```bash
mlebench prepare --split low --competition <单个比赛id>   # 只下一个比赛，约 GB 级，需 key
mlebench grade --submission sub.jsonl                     # CPU 出分
```
预期：单比赛准备分钟~小时级，体积 GB 级；全 Lite 158GB 必须上云。

### SP/NC 映射 + paired_gate
- split：Low/Lite 22 当 public，Medium/High 当 private。
- 评分 medal 三档 + any_medal% → 半连续，配对差可做；22 题偏小，ESS 功效受限。
- 适配度：**中**——信号质量好但需 key、资源重、题数少。

### 诚实缺口
- Kaggle 数据需逐比赛接受规则，MIT 不覆盖数据；3.3TB 无法本机验证规模，引 README 文字。
- 22 题是 split 文件计数，是否为官方“Lite”别名以 README 为准。

---

## 3. ALE-Bench

### 核实后的规模与许可证
- **40 题**（官网 https://sakana.ai/ale-bench/ 实测 200 + README 口径一致；AHC 历届启发式优化题）。本会话从 `src/ale_bench/constants.py` 读到题号全集（ahc001/003/008/011/013/014/016…/048/051、awtf2025heuristic、future-contest-2022-qual 等 >26 个 id）间接佐证；**repo 内未见明确「40」字面计数**，标为官网口径。
- **许可证 Apache-2.0**：LICENSE 原文 “Apache License Version 2.0”（gh api 实测）。
- 一手：https://github.com/SakanaAI/ALE-Bench （raw main 200）、https://sakana.ai/ale-bench/ 。

### 真实加载接口（SP 的关键证据）
```python
session = ale_bench.start(problem_id='ahc001', lite_version=False, ...)
public_result = session.public_eval(code, code_language=...)
score = public_result.overall_absolute_score    # 连续分数
# problem.metadata.score_type 决定 min/max
```
- **README:135-138 明确**：`session` 的 `private_seeds`、`rank_performance_map`、`standings`、`_private_inputs` 等属性 **MUST NOT be accessed** —— 框架级天然密封私集（SP）。
- `docs/session_object.md`：`lite_version=True` 用小 seeds；`maximum_num_call_public_eval` 限制调用次数。
- 评测 CLI（`docs/evaluation.md`）：
  ```bash
  uv run --extra eval -m ale_bench_eval \
      --model_config_path llm_configs/gpt-5.json \
      --problem_ids_type debug|lite|all --n_public_cases 50|5 --judge_version 202510 ...
  ```
  `public_eval` 默认 50 cases(all) / 5(lite)。
- 安装：`pip install 'ale-bench[eval,mcp]'` 或 `uv sync --no-dev --all-extras`；Docker 镜像 `yimjk/ale-bench`（`scripts/docker_build_all.sh $(id -u) $(id -g)`）；数据从 HF clone（本机不通，需上云）。**CPU 可跑，不需 GPU**，需 LLM API key；官方公平对比推荐 AWS c6i.32xlarge。

### 最小 CPU pilot
```python
pip install 'ale-bench[eval,mcp]'   # 本机 CPU
session = ale_bench.start(problem_id='ahc001', lite_version=True)  # 小 seeds
r = session.public_eval('print(sum(map(int,input().split())))', code_language='python')
print(r.overall_absolute_score)     # 连续分
```
预期：单题 public_eval 秒~分钟级；数据下载需云端（HF 本机不通），agent 侧需 LLM key，评分接口本身本地。

### SP/NC 映射 + paired_gate
- split：**public_eval（公开 seeds）= public**，**private_seeds / standings = private（框架隔离）**——四机制里 SP/NC 最现成的一家。
- 评分 **连续 overall_absolute_score** → 逐实例配对差天然、噪声可估 → **最贴 paired_gate + ESS**。
- 适配度：**高**——理论最贴合；扣分在数据上云 + LLM key + 40 题规模。

### 诚实缺口
- 40 题无 repo 内字面计数，仅官网/README + 源码题号间接佐证。
- HF 数据本机不可达，题面/seed 细节未逐条验；`overall_absolute_score` 的量纲按 `score_type` 各题不同，跨题配对需先归一化。

---

## 4. SWE-smith

### 核实后的规模与许可证
- README:51 `load_dataset("SWE-bench/SWE-smith", split="train")  # Loads all 52k task instances`；**52k 任务实例 / 26k SWE-agent 轨迹 / 250+ 环境**（README:64-67）。
- **许可证 MIT**（README badge）。一手：https://github.com/SWE-bench/SWE-smith 。

### 真实加载接口
```python
from datasets import load_dataset
ds = load_dataset("SWE-bench/SWE-smith", split="train")   # 52k
```
- 定位：**训练数据生成器/工具包，非现成 benchmark**；可把任意 GitHub repo 转成 SWE-gym 合成无限任务。
- 需 **Docker**（Ubuntu 22.04.4 LTS；不支持 Win/macOS）。HF 页 https://huggingface.co/datasets/SWE-bench/SWE-smith （本机不可达）。

### 最小 CPU pilot
```python
from datasets import load_dataset
ds = load_dataset("SWE-bench/SWE-smith", split="train[:100]")  # 只取 100 条看 schema，需上云/HF
```
预期：流式取 100 条 MB 级；全量 52k 上云。

### SP/NC 映射 + paired_gate
- 最佳用途：**自举生成 private split**（合成任务天然可密封、无限量）给 outer 确认闸；public 用固定样本。
- 无现成 scoring harness → 必须自建评测（可借 SWE-bench runner）。
- 适配度：**中**——私集生成源价值高，但接入工作量大。

### 诚实缺口
- 52k 是 README 文字（HF 本机不可达），未实测计数。
- 需自建评分，不能直接进 paired_gate。

---

## 5. RE-Bench（已 deprecated）

### 核实后的规模与许可证（**补上旧缺口：总数**）
- README 顶部 **# DEPRECATED** —— tasks deprecated，不再接受贡献/bug。
- `suite_manifest.yaml`（gh api 实测）列 **7 个 task family × 各 1 个 main task**：
  `ai_rd_fix_embedding`(v0.2.5)、`ai_rd_nanogpt_chat_rl`(v0.2.4)、`ai_rd_optimize_llm_foundry`(v0.2.5)、`ai_rd_restricted_mlm`(v0.2.5)、`ai_rd_rust_codecontests_inference`(v0.2.3)、`ai_rd_small_scaling_law`(v0.2.3)、`ai_rd_triton_cumsum`(v0.2.5)。
  → **总数 = 7 主任务**（+ rust codecontests 用 **175 CodeContests held-out 题**，README:73）。
- **许可证 MIT**，但 README:28-38 非正式请求勿泄入训练数据，部分任务密码 zip `resident-peacock-motif-grading`。arXiv 2411.15114（已验号）。
- 一手：https://github.com/METR/RE-Bench （raw main 200）。

### 真实加载接口
- Vivaria scaffold + GPU；任务多为 GPU 密集（微调、RL、kernel）。评分各任务 README 给 starting/official score（percentage solved、avg win%、log(loss-1.5)、log time、interpolated loss）——**连续分数**。

### 最小 CPU pilot
无——**需 GPU + Vivaria，本机不可跑**；最小验证只能读 manifest 与任务 README（已做）。标注「需上云」。

### SP/NC 映射 + paired_gate
- held-out 175 题可当 private；评分连续，理论上适合 ESS。
- 适配度：**低**——deprecated（无维护）+ GPU 依赖 + 任务异质难统一打分。

### 诚实缺口
- 7 主任务的评分量纲完全不同，统一 paired_gate 需逐任务归一化；deprecated 后环境可能已失效。

---

## 6. KernelBench（**补上旧缺口：L4 条数**）

### 核实后的规模与许可证
- 本会话从 gh tree 直接数 `KernelBench/` 下 .py：level1=100、level2=100、level3=50、**level4=20 → 总 270**。
  README:25-32 只写 L1 100/L2 100/L3 50，L4（“Level Hugging Face”）未给数——**L4=20 为本轮 tree 实测补齐**。
- **许可证 MIT**（LICENSE 原文）。HF：https://huggingface.co/datasets/ScalingIntelligence/KernelBench （本机不可达）。
- 一手：https://github.com/ScalingIntelligence/KernelBench （raw main 200）。

### 真实加载接口
```
fast_p    = correct 且 speedup > p 的任务占比   （fast_1 / fast_2 / fast_0）
python scripts/run_and_check.py ... --eval_mode local|modal        # 单样本
uv run python scripts/eval_from_generations.py run_name=... dataset_src=local level=1 num_gpu_devices=8
```
- 安装 `uv sync` / `uv sync --extra gpu`；需 **GPU**（CUDA/DSL 在目标 GPU 测 speedup）+ litellm key。无 GPU 可配 Modal（`modal token new`）。**本机零 GPU 不可跑**。

### 最小 CPU pilot
无 GPU 跑不了 speedup；最小动作是读 level 元数据（已做）。标注「需上云 GPU 或 Modal」。

### SP/NC 映射 + paired_gate
- split：level1-3 当 public、level4 当 private（或按 level 切）。
- speedup 连续信号好，配对差质量高；但需 GPU。
- 适配度：**低**（本机维度）——信号本身优质，纯因 GPU 门槛降级。

### 诚实缺口
- L4=20 来自 tree 文件计数，未实跑确认每个 .py 都是一个有效 task。
- HF 本机不可达，未复核 parquet 与 repo 是否一致。

---

## 7. Terminal-Bench（**补上旧缺口：4.1 任务数**）

### 核实后的规模与许可证
- 真身仓库 `harbor-framework/terminal-bench-1`（原 `laude-institute/terminal-bench` 301 重定向）。
- `registry.json`（gh api 实测）：**terminal-bench-core v0.1.1 的 task_id_subset 长度 = 80**（v0.1.0 = 71）。README:77 原文 “Terminal-Bench-Core v0.1.1 is the set of tasks for Terminal-Bench's beta release and corresponds to the current leaderboard” → **leaderboard 对应 80 题**（本轮补上的精确数）。repo tree 里 task.yaml 总数 256（adapters 15 + original-tasks 241），但那是全集，不是 core 0.1.1。README:29 写「beta with ~100 tasks」（近似口径）。
- **许可证 Apache-2.0**（gh api LICENSE + meta 实测）。一手：https://github.com/harbor-framework/terminal-bench-1 、官网 https://www.tbench.ai/ 。

### 真实加载接口
```bash
uv tool install terminal-bench        # 或 pip install terminal-bench（依赖 uv + Docker）
tb run --agent terminus --model anthropic/claude-3-7-latest \
       --dataset-name terminal-bench-core --dataset-version 0.1.1 --n-concurrent 8
```
- registry 还有 mlebench-lite / swebench-verified / swesmith / usaco / quixbugs 等其他 dataset。
- 需 **uv + Docker + agent/model key**；CPU 可跑。

### 最小 CPU pilot
```bash
uv tool install terminal-bench
tb run --agent terminus --model anthropic/claude-3-7-latest \
       --dataset-name terminal-bench-core --dataset-version 0.1.1 --n-concurrent 1   # 需 Docker + model key
```
预期：单题分钟级；全 80 题需 model key + Docker，命令客观产出测试通过率。

### SP/NC 映射 + paired_gate
- split：core 0.1.1 当 public，adapters / 其他 dataset 当 private，或按 task_id_subset 切两半。
- 评分 = 二值测试通过 → 配对差可做但噪声大（同 SWE-bench）。
- 适配度：**中**——客观打分、可私有化 harness，扣在需 Docker + model key + 二值噪声。

### 诚实缺口
- 80 是 registry 的 task_id_subset，未逐题实跑确认全部可用；256 vs 80 的关系以 registry 为准。
- model key 必需（agent 侧），评分侧的「免 key」优势不如 SWE-bench。

---

## 8. MLAgentBench（**补上旧缺口：许可证**）

### 核实后的规模与许可证
- **13 个 ML 工程任务**（README:12）。
- **许可证 MIT** —— LICENSE 原文 “MIT License”（gh api 实测，上一轮笔记标「未覆盖/缺口」，本轮补上）。arXiv 2310.03302（已验号）。
- 一手：https://github.com/snap-stanford/MLAgentBench （raw main 200）。

### 真实加载接口
```bash
pip install -e .
docker run -it --user root -v "$(pwd)":/MLAgentBench -w /MLAgentBench qhwang123/researchassistant:latest
python -u -m MLAgentBench.prepare_task <task> $(which python)
python -u -m MLAgentBench.runner --python $(which python) --task cifar10 --device 0 \
       --log-dir <dir> --work-dir workspace --llm-name gpt-4 ...
python -m MLAgentBench.eval --log-folder <folder> --task <name> --output-file <out>
```
- 需 **Kaggle key**（`~/.kaggle/kaggle.json`）+ **LLM key**（`openai_api_key.txt` 格式 `org:key` / `claude_api_key.txt` / `crfm_api_key.txt`；另支持 gemini/HF）。

### 最小 CPU pilot
```bash
pip install -e .
python -m MLAgentBench.eval --log-folder <已有示例log> --task cifar10 --output-file out.json   # 仅评分，需 Docker 环境
```
预期：评分秒级；产生轨迹需 Docker + 多 key + device。

### SP/NC 映射 + paired_gate
- 13 任务切分；规模太小，ESS 功效不足。
- 适配度：**中低**——许可证缺口已补（MIT），但规模小、需多 key。

### 诚实缺口
- 13 任务每任务指标不同，统一打分需归一化；规模限制统计功效。

---

## 9. ScienceAgentBench

### 核实后的规模与许可证
- **102 tasks / 44 论文 / 4 学科 / 9 专家**（README:36）；容器化 harness 102 实例 / 8 线程 / 30min（README:14）。
- **许可证**：任务 **CC-BY-4.0**（rasterio Instance 32/46/53/54/84、matminer Instance 3 保留原许可），代码 MIT（LICENSE 原文）。
- 一手：https://github.com/OSU-NLP-Group/ScienceAgentBench （raw main 200）。

### 真实加载接口
```bash
# dockerized eval
python -m evaluation.harness.run_evaluation --log_fname <f> --run_id <id> \
       --cache_level base|instance --max_workers 4 --instance_ids <ids...>
# direct eval（需 OpenAI key，GPT-4o judge 可视化）
python -u run_eval.py --log_fname <f>
```
- 指标：examined programs / execution / cost 多指标。
- **获取摩擦**：HF 只给 annotation sheet（防污染），完整 benchmark 需 **SharePoint + 密码 `scienceagentbench`**，要求勿在线重分发；verified split 2026-04-30 发布（`benchmark_verified.zip`）。

### 最小 CPU pilot
```bash
python -m evaluation.harness.run_evaluation --log_fname <example> --run_id pilot \
       --cache_level base --max_workers 1 --instance_ids <单个id>   # 需先拿到 SharePoint 数据
```
预期：单实例分钟级；数据获取是主要摩擦（HF 本机不通 + 密码包）。

### SP/NC 映射 + paired_gate
- split：official verified split vs 原始。
- 多指标 + GPT-4o judge（judge 有噪声）；配对差可做但 judge 噪声需纳入 ESS。
- 适配度：**中**——CPU 可跑但获取有摩擦、judge 引入额外噪声。

### 诚实缺口
- 数据需 SharePoint + 密码，HF 本机不通，未实测题面；102 是 README 文字。
- GPT-4o judge 需 key 且判分有随机性，需多次采样估噪声。

---

## 10. aira-dojo（Meta）

### 核实后的规模与许可证
- 实现 MLE-Bench 任务；官方称曾支持 1000 agent 并行 / 120h；**独立任务数未给（缺口）**。
- **许可证 CC-BY-NC-4.0（非商业）** —— README:3 badge + README:173 正文；THIRD_PARTY_LICENSES.md 另列。arXiv 2507.02554（已验号）。
- 一手：https://github.com/facebookresearch/aira-dojo （raw main 200）。

### 真实加载接口
```bash
pip install -e .
# 配置：src/dojo/configs/benchmark/mlebench/{all,lite,dev,debug}.yaml
# splits：low / high / medium / low_dev / all .txt
```
- 需 LLM API（litellm，Azure/openai）；支持 Slurm。

### 最小 CPU pilot
```bash
pip install -e .
# 用 lite/dev config 起单 agent 冒烟（需 LLM key；本机 CPU 可跑编排）
```
预期：编排轻量；实跑需 key + Slurm/容器环境。

### SP/NC 映射 + paired_gate
- split：mlebench lite/low 当 public。
- 任务指标同 MLE-Bench；贴 hill-climbing RSI 场景。
- 适配度：**中**——场景贴合，但 **NC 许可风险**（非商业）需先确认我们的用途。

### 诚实缺口
- 独立任务数未给（未补上）。
- CC-BY-NC 对潜在商用/发表用途的边界需人工确认。

---

## 11. AIDE（aideml）

### 核实后的规模与许可证
- **agent 实现，非独立数据集**；自带 `example_tasks/house_prices`（test.csv + data_description.txt）；评测复用 MLE-Bench 75 竞赛。
- **许可证 MIT**（LICENSE 原文 “Copyright (c) 2024 Weco AI Ltd”）。arXiv 2502.13138（已验号）。
- README:39 “MLE-Bench (75 Kaggle comps) found the tree-search of AIDE wins 4× more medals than the best linear agent (OpenHands)”。
- 一手：https://github.com/wecoai/aideml （raw main 200）。

### 真实加载接口
```bash
pip install aideml        # 或 pip install -e .
aide data_dir="example_tasks/house_prices" goal="..." eval="AUROC"   # 需 OPENAI_API_KEY
```
- 树搜索式 ML 工程 agent。

### 最小 CPU pilot
```bash
pip install aideml
# OPENAI_API_KEY=... aide data_dir="example_tasks/house_prices" goal="predict price" eval="RMSE"
```
预期：单任务需 LLM key；house_prices 自带数据 MB 级，本机可跑但需 key。

### SP/NC 映射 + paired_gate
- split 同 MLE-Bench。
- 定位：**作对照基线**（我们的 inner 策略 vs AIDE 树搜索），**勿重复计为独立数据集**。
- 适配度：**中**（基线价值，非试验床）。

### 诚实缺口
- 无独立数据，评分完全依赖 MLE-Bench 或自带小例；作为 rsi-agent 试验床价值有限。

---

## 接入优先级排序

按用户四标准「**本机 CPU 可跑 / 无需 key / 客观打分 / 能 split**」排序（本机零 GPU + HF 不通是硬约束）：

| 排名 | 数据集 | CPU | 免 key | 客观分 | 能 split | 说明 |
|------|--------|-----|--------|--------|----------|------|
| 1 | **SWE-bench** | ✓ | ✓（评分/出分侧） | ✓ F2P | ✓ Verified/full | 四项全中；评分侧完全免 key |
| 2 | **ALE-Bench** | ✓ | ✗（agent key） | ✓ 连续分 | ✓ 框架级 SP | 最贴 ESS，但需 key + HF 上云 |
| 3 | **Terminal-Bench** | ✓ | ✗（model key） | ✓ 通过率 | ✓ task_id_subset | 客观可切分，需 Docker+key |
| 4 | SWE-smith | ✓ | ✓ | ✗ 需自建 | ✓ 合成无限 | 私集源，非现成 bench |
| 5 | ScienceAgentBench | ✓ | ✗（judge key） | ~ judge | ✓ verified | 获取摩擦大 |
| 6 | MLAgentBench | ✓ | ✗ | ✓ | ✓ | 仅 13 题 |
| 7 | MLE-Bench Lite | ✓ | ✗（Kaggle key） | ~ medal | ✓ 22 | 数据 158GB 需上云 |
| 8 | KernelBench | ✗ GPU | ✗ | ✓ speedup | ✓ level | 需 GPU/Modal |
| 9 | RE-Bench | ✗ GPU | ✗ | ✓ | ✓ | deprecated + GPU |
| 10 | aira-dojo | ~ | ✗ | ~ | ✓ | NC 许可风险 |
| 11 | AIDE | ✓ | ✗ | ~ | ~ | 对照组，非数据集 |

> 权衡说明：SWE-bench 之所以压过信号更好的 ALE-Bench，是因为「评分侧免 key」+「本机 CPU 可出分」两条硬标准它全中，可直接做端到端 pilot；ALE-Bench 的连续分与原生 SP 最适合 ESS，但数据必须上云、agent 需 key，接入链更长。

### 前三名具体 pilot 接入方案

**① SWE-bench**
- 数据集 → split：`Verified 500` = public（inner 改策略/补丁）；`full − verified` = private（outer 密封确认）。
- 评分接口：`swebench eval verified -p preds.jsonl --run-id <id> -j 4` + `swebench report <id> -d verified`，每实例 resolved 二值 → resolve_rate。
- paired_gate 怎么吃：同一 instance 上策略 A/B 的 `resolved_A − resolved_B ∈ {−1,0,+1}` 逐实例配对差；因二值，靠**多实例配对 + bootstrap** 估噪声，ESS 判「inner 提升是否超自相关噪声」，过闸才用 private 池做 outer 确认（换 run_id 防缓存复用）。
- 最小 pilot 命令（一句话）：`pip install -e . && swebench eval verified --gold --task-repo ./swe-bench-tasks`（gold 自检 harness；需 Docker、`swebench eval --help` 限实例做冒烟，评分/出分免 key）。

**② ALE-Bench**
- 数据集 → split：`public_eval(公开 seeds)` = public；`private_seeds / standings`（框架禁止访问）= private（outer 密封）。
- 评分接口：`session.public_eval(code).overall_absolute_score` 连续分（`lite_version=True` 用 5 cases 冒烟）。
- paired_gate 怎么吃：逐题 A/B 策略连续分差 `score_A − score_B`（先按 `score_type` 归一化量纲）；连续差噪声可估 → ESS 直接吃配对差序列，判提升是否稳健；private_seeds 只在 outer 揭盲。
- 最小 pilot 命令（一句话）：`pip install 'ale-bench[eval,mcp]' && python -c "import ale_bench; s=ale_bench.start(problem_id='ahc001',lite_version=True); print(s.public_eval('print(sum(map(int,input().split())))',code_language='python').overall_absolute_score)"`（CPU、需 LLM key + HF 数据上云）。

**③ Terminal-Bench**
- 数据集 → split：`terminal-bench-core v0.1.1` 的 `task_id_subset` 一半 = public，另一半（或 adapters/其他 dataset）= private。
- 评分接口：`tb run --dataset-name terminal-bench-core --dataset-version 0.1.1` 输出每任务测试通过与否 → pass_rate。
- paired_gate 怎么吃：同任务 A/B 策略通过与否配对差（二值，同 SWE-bench 处理）；public 半挑策略、private 半确认。
- 最小 pilot 命令（一句话）：`uv tool install terminal-bench && tb run --agent terminus --model anthropic/claude-3-7-latest --dataset-name terminal-bench-core --dataset-version 0.1.1 --n-concurrent 1`（CPU、需 Docker + model key）。

---

## 全表诚实缺口汇总
- **HF 本机全不通**：SWE-bench Multimodal 480、SWE-smith 52k、KernelBench 270、ALE-Bench 题面、MLE-Bench 数据规模均引 README/官网文字，未逐条实测 parquet；所有 HF 下载需上云。
- **补上的 4 个精确缺口**：Terminal-Bench core v0.1.1=80（registry task_id_subset）；RE-Bench=7 主任务+175 held-out（suite_manifest + README:73）；KernelBench L4=20/总 270（gh tree 实测）；MLAgentBench 许可证=MIT（LICENSE 原文）。
- **未补上的缺口**：aira-dojo 独立任务数（官方未给）；ALE-Bench「40」无 repo 内字面计数（仅官网/README + 源码题号间接佐证）。
- **许可风险**：aira-dojo CC-BY-NC-4.0（非商业）；MLE-Bench 竞赛数据不受 MIT 覆盖；ScienceAgentBench 禁在线重分发（SharePoint+密码）。
- **评分噪声**：二值 F2P / 通过率（SWE-bench、Terminal-Bench）配对差噪声大，ESS 需更多实例；GPT-4o judge（ScienceAgentBench）有判分随机性，需多次采样。
- **本机约束**：零 GPU（KernelBench/RE-Bench/MLE-full 不可本地）；本笔记只写 pilot 命令，**未实际下载大数据集、未本地长跑**（AGENTS.md §17）。
