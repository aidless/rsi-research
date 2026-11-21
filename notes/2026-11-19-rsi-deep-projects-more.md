# rsi-agent 源码级深读：9 个其余开源项目（2026-11-19）

> 上一轮 `notes/2026-11-19-rsi-projects.md` 是 README 级调研；本轮对 9 个 repo **读真实源码**，重点找「接受闸 / 反馈闭环 / 归档隔离 / 审计」的实现。
> 通道：`gh api repos/<o>/<r>/git/trees/HEAD?recursive=1` 取树 + `curl -sL https://raw.githubusercontent.com/<o>/<r>/<branch>/<path>`（实测 200）取单文件，未 `git clone`。
> 源码落盘：`analyses/tracks/rsi-deep/projects-more/sources/<owner>__<repo>/`（文件名把原路径的 `/` 换成 `__`，同目录存 `.branch` 与 `TREE.txt`）。原件零改动。
> 对表口径：SP=密封私集，NC=非循环评分，ESS=自相关感知 ESS 配对闸，HA=哈希链审计。

## 总览表

| # | repo（分支 / blob 数） | 机制一句话 | 对 rsi-agent 的价值 |
| --- | --- | --- | --- |
| 1 | [WenyiWU0111/RSIGame](https://github.com/WenyiWU0111/RSIGame)（main / 168） | 全局循环：最优 checkpoint 采纳闸 + 饱和停止 + 回归 checklist | 补强 NC/ESS：多 pass 中位数、噪声阈、champion 连续不变停止 |
| 2 | [mmeyerlein/meclaw](https://github.com/mmeyerlein/meclaw)（main / 4013） | 统一变更门 + 每 cell 内核沙箱 + append-only receipt | 补强 SP：统一变更门、失败即杀的沙箱、ledger 不可写 |
| 3 | [Clawland-AI/Geneclaw](https://github.com/Clawland-AI/Geneclaw)（master / 145） | GEP 协议 + 5 层 gatekeeper + pytest 验证回滚 | 补强 SP：路径白/黑名单 + secret 扫描 + 分支回滚 |
| 4 | [lemoz/darwin-godel-machine](https://github.com/lemoz/darwin-godel-machine)（main / 785） | DGM 群体演化 + 冻结突变 held-out 重放 | 补强 ESS/NC：per-benchmark 非回归过滤、验证闸与选择闸分离 |
| 5 | [MineDojo/Voyager](https://github.com/MineDojo/Voyager)（main / 457） | 技能库 + 自动课程 + critic 反馈 | 反馈闭环参考（无接受闸） |
| 6 | [noahshinn/reflexion](https://github.com/noahshinn/reflexion)（main / 434） | 失败轨迹→反思→重试 | 反馈闭环参考（无评分阈） |
| 7 | [aiwaves-cn/agents](https://github.com/aiwaves-cn/agents)（master / 12649） | loss→language gradient→更新 prompt/节点 | 非循环评分参考（无密封闸） |
| 8 | [madaan/self-refine](https://github.com/madaan/self-refine)（main / 186） | 最简自反馈改进 | **反面对照基线** |
| 9 | [mmtmn/Darwin-Godel-Machine](https://github.com/mmtmn/Darwin-Godel-Machine)（main / 20） | 本地无 key 群体自改循环 | 零 GPU/无 key 可跑路径参考 |

**跨 9 repo 的共同空白**：grep `prev_hash|hash_chain|merkle|block_hash` 全部为空。**没有任何一个 repo 实现 HA（哈希链审计）**。最接近的是 meclaw 的 append-only receipt 与 lemoz 的 `_tree_sha256`（Merkle 式树哈希，但不链式）。rsi-agent 的 HA 在这 9 个开源项目里找不到对标实现，是真实差异化。

---

## 1. WenyiWU0111/RSIGame（详读）

**源码清单**（`sources/WenyiWU0111__RSIGame/`）：`src/rsigame/monitor/value_stop.py`、`monitor/live.py`、`monitor/vm_replay.py`、`monitor/comparator.py`、`training/checkpoints.py`、`checklist/checklist_state.py`、`checklist/task_checklist.py`、`checklist/bootstrap_verifier.py`、`controller/verify.py`、`controller/rank.py`、`eval/{score_game,score_rounds,rebuild_round,stability_extract}.py`、`review/audit_director.py`、`agent/evolve/verifier_agent.py`、`loop.py`、`config.py`。

**机制还原**：

- **饱和停止**：`monitor/value_stop.py:46-47` `def stop_k() -> int: return int(os.environ.get('RSIGAME_MONITOR_STOP_AFTER_K') or 3)`；`:71-79` `def value_stop(trace, k=None, game='') -> Stop` 中 `for i in range(k, len(champs)): window = [c for _,c in champs[i-k:i+1]]; if window[0] is not None and all(c == window[0] for c in window): return Stop(game, i, champs[i][0], int(window[0]), True)`——champion 连续 K 个 checkpoint 不变即 `saturated=True`。K=3 × checkpoint every 3 ≈ 9 rounds。
- **采纳闸**：`monitor/vm_replay.py:80-81` `def gain(): return float(os.environ.get('RSIGAME_MONITOR_CHAMPION_GAIN') or 0.05)`；`:374-432` `def decide_checkpoint(g, r, champ, rb, *, rerecord_ok=True)`：build_failed/crashed/no_recording/parse_error 永不采纳；`:418` `if ov['delta'] is not None and ov['delta'] >= gain(): rec['event']='replaced'; champ=r`；`:421-428` 特例（老 champion 不可评分、新可评分）也替换，否则 kept。
- **多次评分取中位数（ESS 核心）**：`vm_replay.py:309-352` `compare()` 对同一记录跑 `RSIGAME_MONITOR_PASSES>1` 次取中位数，doc 原文说明理由——同一记录两次评分 champion 会落在 R6 vs R12（brief half −0.10 vs +0.17），"temperature=0 but provider 有方差，median of three 是最便宜、又不至于把掷硬币变成交付构建的做法"；`:339-344` `by_origin` 各字段逐 pass 取 `_median`，防止某 pass 缺 origin 抛 `KeyError:'task'`（曾杀 maze supervisor）。
- **checkpoint 持久化**：`training/checkpoints.py:48` `MARKER='.complete'`；`:70-88` `sync_once` 拷贝返回后才 touch marker，partial copy 先 rmtree；`:64-67` `newest_complete` 只认带 marker 的——marker 是整个设计，避免 resume 到残缺 checkpoint。
- **回归 checklist**：`checklist/checklist_state.py:55` `STATUSES=('satisfied','confirmed_gap','unverified')`；`:64-69` `_ALLOWED` 转移表含 `('satisfied','confirmed_gap')=regressed`，无回到 unverified 的路径；`:114-162` `update()` 沉默不写（unverified 读数不改写已有状态），只证据移动状态，`edited=True` 把旧 satisfied 标 stale。
- **审计**：`review/audit_director.py:76-123` `shell_ok()` 白名单命令/路径/FILE_RE 检查；`:134-155` violations→`<session_dir>/audit.json`，字段 `clean: not violations`——**目录审计，非哈希链**。散点 sha256：`checklist/task_checklist.py:286` `hashlib.sha256(f'v{SCHEMA_VERSION}\n{raw}'.encode()).hexdigest()[:24]`（缓存键）；`eval/rebuild_round.py:101` 逐文件 sha256 摘要。
- **loop 集成**：`loop.py:2244-2270` 监控在 round 落盘后运行；monitor 抛异常则记日志并把 `mon=None`，本轮不再 early stop——doc `:2248-2249` 明确 "estimator outage is not a reason to end a run early, and silently stopping would look exactly like saturation"。

**对表**：
- **SP（密封私集）≈ 有**：champion 持守 + checkpoint marker + replay 只回放固定输入脚本的同一 replay，champion 只决定交付哪个，绝不回滚/编辑。
- **NC（非循环评分）≈ 有（部分）**：`comparator` 不知 round/意图/预算，只判 "what changed"；judge 看固定输入脚本的同一 replay，非循环复评。
- **ESS（自相关感知 ESS 配对闸）≈ 有**：`compare()` 多 pass 中位数 + 噪声阈。噪声阈 `monitor/comparator.py:57` `CHANGED_AT=3.0`（同行为重放差 0.26–1.91，真实变化 7.5–10.8）。
- **HA（哈希链审计）≈ 无**：仅 append champion trace + 散点 sha256，无链式。**可移植点**：`decide_checkpoint` 的 gain 阈值 + `Stop(saturated)` 字段 + `checkpoints.py` 的 `.complete` marker，都是 rsi-agent outer 闸可直接对照的形态。

**本地可跑性**：纯 Python，依赖 godot/游戏侧（非 rsi-agent 目标），`monitor` 默认关（`RSIGAME_MONITOR_LIVE` off by default，保论文每轮同预算），可零 GPU 跑单测级。**缺口**：仅读了本轮下载的 monitor/checklist/eval 子集，未通读 168 个 blob 的全部 `loop.py` 115KB 与 agent/evolve 目录；`value_stop` 的 K 默认 3 与 checkpoint 周期耦合的实测墙钟未复核。

---

## 2. mmeyerlein/meclaw（详读）

**源码清单**（`sources/mmeyerlein__meclaw/`）：`crates/meclaw-cells/src/sandbox/mod.rs`、`sandbox/linux.rs`、`crates/meclaw-colony/src/mutation/{validate,dry_run,apply}.rs`、`crates/meclaw-api/src/handlers/ledger.rs`、`docs/why/rsi.md`、`templates/argus/charter/seed/rules.jsonl`、`templates/argus/template.json`。

**机制还原**：

- **每 cell 内核沙箱（fail-closed）**：`sandbox/mod.rs` 文件头：bash/code/harness cell 以 daemon 权限起子进程，此模块在 spawn 时剥夺权限；四机制 fail-closed——Landlock 文件系统视图 / `unshare(CLONE_NEWUSER|CLONE_NEWNET)` 网络拒绝 / cgroup v2 资源上限 / seccomp-bpf 系统调用过滤；原文 "A profile that cannot be applied makes the spawn fail; there is no path on which a restricted profile quietly does nothing"，每机制有 probe（靠 TRY 而非读旋钮）。`sandbox/linux.rs:6-20` 顺序：① join cgroup ② `PR_SET_NO_NEW_PRIVS` ③ `unshare` ④ Landlock ⑤ seccomp（必须最后，否则 filter 要 allow 前面所有 syscall）；用 Landlock 而非 mount namespace，因 unprivileged daemon 无 `CAP_SYS_ADMIN`（mount(2) 全失败）。
- **统一变更门**：`mutation/validate.rs:15` `pub const DIFF_OPERATIONS: [&str;9]` = add_templates / add_nodes / remove_nodes / swap_nodes / replace_nodes / move_nodes / add_edges / remove_edges / seed_rows；diff 中无操作读取的 key 拒而非忽略（`refuse_unknown_diff_keys`），稳定 28 个 error_code 不新增。`mutation/dry_run.rs:72-149` `dry_run_manifest` 在 temp scratch 用同一 `substitute_mutation_diff + build_staging_tree_into` 预检，不碰 colony root / 无 db write / 无 spawn；区分 `DryRunFinding::Refused` vs `NotChecked`。`mutation/apply.rs:55-114` `apply_mutation` stage→`atomic_rename_or_overwrite_all`。
- **自改循环与诚实结论**：`docs/why/rsi.md`——mutation = `POST /colony/mutations` 的 diff，agent 也走同一门；builder 无到 mutation door 的边，只有 submit 节点有（builder 不能自己加边）；argus = 8-cell 控制循环（读 charter → 从 ledger 测量 → model judge → 发 params update → 测效果 → keep or revert）；cycle 无 pre-authored revert plan 被拒 `no_revert_plan`；每 tick append-only receipt。原文关键句："Does a colony improve itself? The primitives for it are here and tested, and nothing closes them into an unattended loop. That is a decision, not a gap in the schedule."（`goals.jsonl` 全 `enabled=0`，#304 无节点写 diff）。
- **显著性闸**：`templates/argus/charter/seed/rules.jsonl`：`rule:revert-plan require_revert_plan=1`；`quality_floor_pct=0`（质量下降即回滚）；`probe_timeout_ms=30000`；`eval_min_batch=30`（"十轮对话判文本=带收据的抛硬币，会震荡"）；`never=identity`（argus 不许碰身份文本，先于一切检查）。`template.json` description：每 tick 留 append-only receipt（空 tick 也写，否则"没运行"与"没决定"从外看一样）；quality is a gate；significance `min_delta_pct/min_samples`（样本不足 skipped，"we did not look" vs "we looked and saw nothing" 是不同事实）；两时钟 probe(秒) vs window(小时)。
- **审计日志**：`handlers/message_log.rs:31-41` `DEFAULT_SCAN_BUDGET=5000 / MAX=50000 / RESOLVE_BLOB_MAX=25`；secret hygiene 错误只回字段名不回值（`:76-91`）。

**对表**：
- **SP ≈ 有（强）**：沙箱 + 统一变更门 + 身份 `never`；变更走单一 door，builder 无自改边。
- **NC ≈ 有（强）**：measurement drawn from a ledger the loop cannot write（循环不写测量账本，只读）。
- **ESS ≈ 有**：`min_samples=30`、`min_delta_pct`、`quality_floor_pct` 显著性闸，样本不足 skipped 而非判负。
- **HA ≈ 无**：append-only receipt，**无 prev_hash 链式**（grep templates/argus + handlers 无结果）。**可移植点**：`eval_min_batch` + `min_delta_pct` 的显著性闸直接对应 rsi-agent ESS；ledger 不可写（"the loop cannot write"）是 NC 的强形态。

**本地可跑性**：Rust 项目，需 cargo + Linux（Landlock/seccomp/cgroup 为内核特性）；`docs/why/rsi.md` 明确 self-improving loop 未闭环（goals 全 disabled），零 GPU 不涉及。**缺口**：未读 handlers/ledger.rs 与 colony 状态机全文，4013 blob 只读了沙箱+mutation+argus 配置；"ledger the loop cannot write" 的写权限边界在 handler 层的具体实现未逐行核。

---

## 3. Clawland-AI/Geneclaw（详读）

**源码清单**（`sources/Clawland-AI__Geneclaw/`）：`geneclaw/gatekeeper.py`、`geneclaw/apply.py`、`geneclaw/event_store.py`、`geneclaw/models.py`、`docs/specs/GEP-v0.md`。

**机制还原**：

- **GEP 协议**：`docs/specs/GEP-v0.md:14` `Observe → Diagnose → Propose → Gate → Execute → Evaluate → Record`。
- **5 层 gatekeeper**：`geneclaw/gatekeeper.py:17-80` `def validate_proposal(proposal, *, allowlist_paths=None, denylist_paths=None, max_patch_lines=500) -> tuple[bool, list[str]]`：denylist（`:46-52`）→ allowlist（`:54-61`）→ `len(proposal.unified_diff.splitlines()) > max_patch_lines`（`:63-68`）→ `redact_secrets(diff) != diff` 即拒（`:70-73`）→ `_scan_suspicious_deps`（`:75-77`）。`:87-104` `_SUSPICIOUS_PATTERNS` 正则 `^\+.*(subprocess\.call|os\.system|eval|exec)\(`。
- **apply 安全**：spec §5.2（`:90-94`）：默认 dry-run；需 `--apply` 且 `allow_apply_default=true`；建分支 `evo/<timestamp>-<slug>` 跑 pytest；失败自动回滚。
- **execute**：`geneclaw/apply.py:60-144` `apply_unified_diff`：gatekeeper reject 直接返回 `(False, reasons)`；dry_run 通过后不改文件；`git apply --check`（`:87-95`）；建 evo 分支（`:78-83`）；`git apply`；`pytest -q` timeout=300（`:114-127`）；失败 `_rollback_branch`（`:129-131`）；成功 `git add -A` + commit，msg 带 Evo-Event-ID / Risk-Level / Tests（`:133-142`）。
- **record**：`geneclaw/event_store.py:18-77` `EventStore` append-only JSONL 落在 `<workspace>/geneclaw/events/events.jsonl`，record 先 `redact_secrets` 再 `open(...,'a')`（`:27-33`），`iter_events(since_hours)` 过滤——**纯 append，无 hash chain**。
- **模型**：`geneclaw/models.py` `EvolutionProposal(id,title,objective,evidence,risk_level,files_touched,unified_diff,tests_to_run,rollback_plan)`；`EvoEvent.event_type Literal[evolve_generated,apply_attempted,apply_succeeded,apply_failed]` + `parent_event_id`（有父子关系，无加密链）。

**对表**：
- **SP ≈ 有**：路径 allow/denylist + secret 扫描 + 可疑依赖正则；patch 上限 500 行。
- **NC ≈ 弱**：pytest 是外部测试而非评分器，但反馈来源是模型自生，无独立密封评分。
- **ESS ≈ 无**：无 min batch、无多次评分、无显著性阈。
- **HA ≈ 无**：append-only JSONL + `parent_event_id`，无链式哈希。**可移植点**：5 层 gatekeeper 的「denylist → allowlist → 大小 → secret → 可疑模式」顺序可直接映射 rsi-agent 的密封闸前检查；`apply_unified_diff` 的 git 分支 + pytest + 回滚是 SP 的可执行形态。

**本地可跑性**：纯 Python，`pip install` + 本地 git 仓库即可跑 dry-run，零 GPU/无 key（pytest 本地）。**缺口**：`Evaluate` 阶段只有 pytest，无 held-out 泛化测量，未读 README 之外的 CLI 入口与 `--apply` 默认值的实际配置；未读 `docs/specs/GEP-v0.md` 全文（只取了 §5.1/§5.2 行号）。

---

## 4. lemoz/darwin-godel-machine（详读）

**源码清单**（`sources/lemoz__darwin-godel-machine/`）：`dgm_controller.py`、`agent/agent.py`、`archive/agent_archive.py`、`archive/parent_selector.py`、`evaluation/{agent_validator,benchmark_runner,scorer}.py`、`sandbox/sandbox_manager.py`、`self_modification/{implementation,modification_proposal,mutation_guard,performance_diagnosis}.py`、`config/livecodebench_segment_heldout12.yaml`、`config/livecodebench_heldout_transfer_matrix.yaml`、`docs/live-runs/lcb-heldout-transfer-20260716-1/{README.md,summary.json,aggregate.json}`。

**机制还原**：

- **验证闸与选择闸分离**：`dgm_controller.py:442` 注释 `# 7. Archive ALL agents that passed validation (paper-faithful — no score gate)`；`:443-452` `archive.add_agent(...)` 把通过 `validator.validate_agent` 的子体全部入档（不按分数筛）；`:458-460` `if total_score > parent.average_score: self.successful_improvements += 1`（改进只作计数，不阻断入档）。即：**接受闸=结构/语法/沙箱验证，选择闸=父选择时的非回归过滤**。
- **非回归过滤（ESS/NC 核心）**：`archive/parent_selector.py:26-32` docstring：`require_non_regression` 排除平均分回归的子体；`require_per_benchmark_non_regression` 排除任一 benchmark 回归的子体；`regression_tolerance` 浮点容差；`reject_score_ties` 排除与父打平的子体。`:207-243` `_passes_regression_filter`：`if not self.require_non_regression: return True`；`:223-227` `require_per_benchmark_non_regression` 时逐 benchmark `if child_score < parent_score - tolerance: return False`；`:229-233` 平均分 > 父 +容差 → True，< 父 −容差 → False，打平则按 `reject_score_ties` 或再逐 benchmark 查（`:238-241`）。父选择公式（`:4-9`）为论文 sigmoid `s_i=1/(1+exp(-λ(α_i−α_0)))` × `h_i=1/(1+n_i)` × 归一化。
- **冻结突变 held-out 重放（本项目对表核心）**：`docs/live-runs/lcb-heldout-transfer-20260716-1/README.md`：从 broad self-elicitation 实验冻结 generation-zero 的 agent，在**不相交**的 12 题 AtCoder segment（`release_v6_atcoder_heldout12`）上重放，**transfer run 期间禁止任何新突变**（`mutation_generations: 0`）。结果表 5 条 lineage：`+1.0 / +1.0 / +1.5 / +1.0`（4 条选中），其中 Gemini solver-policy 为负对照（`−0.5`，丢了 easy 题，`easy_preserved: false`）。协议：source commit `38fdabed…`，4 easy / 5 medium / 3 hard，`public_test_count=27 / private=480 / total=507`，`agent_problem_evaluations=216`，`replicates_per_agent=2`，4 个并行 `e2-standard-2` VM。`summary.json` 字段 `native_solved / mutated_solved / delta / mutated_difficulty_scores / easy_preserved`。
- **held-out 段选择门槛**：`config/livecodebench_segment_heldout12.yaml` `gates:` `min_problem_count: 12`、`required_difficulties: [easy,medium,hard]`、`min_total_tests: 400`、`min_private_tests: 350`、`require_stdin_only: true`、`prompt_policy: public_examples_only`；数据源 `sha256: bb4c364f…`，`approval_required: true`。
- **沙箱**：`sandbox/sandbox_manager.py:31-34` `memory_limit: "2g" / cpu_limit: "1" / timeout: 300 / network_mode: "none"`；`:121-131` `_require_docker`（docker SDK 缺失则报错，非静默降级）；`:141+` `execute_in_sandbox` 一次性 Docker 容器执行。
- **树哈希（最接近 HA 的地方）**：`dgm_controller.py:900-911` `_tree_sha256(manifest)` 对排序后的 `path/size/sha256` 逐项 `update` 求 sha256——**Merkle 式树哈希，非链式账本**；`:922-933` before/after manifest diff 出 added/removed/modified 文件；`:1000` `metadata["patch_sha256"] = hashlib.sha256(...)`。
- **变更防护**：`self_modification/mutation_guard.py:58` `class ConstrainedMutationGuard`，`:80` `inspect()`；`self_modification/implementation.py:27-96` `ImplementationManager`：`:59-61` 非 dry_run 先 `_create_backup`，`:84-85` 应用失败 `_rollback_changes`，`:284-302` `_rollback_changes` 从备份 `copytree` 恢复。

**对表**：
- **SP ≈ 有**：沙箱隔离（docker，network none）+ 变更前备份回滚 + 验证闸（结构/语法/沙箱）。
- **NC ≈ 有**：benchmark 分数在 `benchmark_runner` 内独立于自改循环测量；held-out 段与搜索段**不相交**，防止过拟合原题。
- **ESS ≈ 有（强）**：`require_per_benchmark_non_regression` 逐 benchmark 非回归 + `regression_tolerance` + `reject_score_ties` + held-out 段门槛（≥12 题、≥400 测试、≥350 私有、stdin-only）。
- **HA ≈ 无**：`_tree_sha256` 是树哈希非链式；无 prev_hash。**可移植点**：`_passes_regression_filter` 的「逐 benchmark 非回归 + 容差 + 打平即拒」可直接补强 rsi-agent ESS 的配对闸；`easy_preserved` + 不相交 held-out 重放是 NC 的可执行形态（`mutation_generations: 0` 冻结重放 = rsi-agent 密封确认闸的实证）。

**本地可跑性**：Python + Docker（沙箱需 docker）；`agent_validator`/`benchmark_runner` 在 docker 缺失时有 `is_docker_available()` 回退（`sandbox_manager.py:17-24` docstring 说明 docker SDK 可选，缺失回退直接 subprocess——**回退路径的安全边界需注意**）。零 GPU，held-out 重放需 LiveCodeBench 数据。**缺口**：785 blob 只读了 dgm_controller/agent/archive/evaluation/sandbox/self_modification 核心 + held-out 文档，未通读全部 runner/scorer；`_tree_sha256` 的 manifest 生成（`_hash_manifest`）实现未逐行核；negative control 的统计显著性未做（作者自述 small-sample）。

---

## 5. MineDojo/Voyager（快速核）

**源码清单**（`sources/MineDojo__Voyager/`）：`voyager/voyager.py`、`voyager/agents/skill.py`、`voyager/agents/critic.py`、`voyager/agents/curriculum.py`、`voyager/agents/action.py`。

**机制还原**：
- **技能库**：`agents/skill.py:13+` `SkillManager`，`:28-30` 建 `skill/code`、`skill/description`、`skill/vectordb` 三目录；`add_new_skill` 生成描述 → `vectordb.add_texts` → 写 `skills.json` + `.js`/`.txt`，带 `assert self.vectordb._collection.count() == len(self.skills)`（`:48-63` 附近）保证向量库与 skills.json 同步。
- **反馈闭环**：`voyager.py:203+` `step()`：action agent 产代码 → `self.env.step(code, programs=self.skill_manager.programs)` → `self.critic_agent.check_task_success(...)`（`:221-225`）；critic 判定成功/失败后决定是否把任务写入技能库。
- **自动课程**：`agents/curriculum.py` `CurriculumAgent.propose_next_task`（`:240-276`）维护 `completed_tasks / failed_tasks`，`warm_up` 分段决定何时展示可选库存/ chest 上下文，模式 `auto`/`manual`。

**对表**：SP≈无（无接受闸，技能库只增不判）；NC≈弱（critic 反馈是环境观察，非密封评分）；ESS≈无；HA≈无。**可移植点**：技能库的「代码+描述+向量库三件套同步 assert」可作 rsi-agent 归档的形态参考。**本地可跑性**：需 Minecraft/Mineflayer + OpenAI key，零 GPU 但**非零 key**，且游戏环境重。**缺口**：未读 `action.py` 全文与 prompts 模板，critic 判定逻辑只取到 `check_task_success` 调用点。

## 6. noahshinn/reflexion（快速核）

**源码清单**（`sources/noahshinn__reflexion/`）：`programming_runs/reflexion.py`、`programming_runs/immediate_reflexion.py`、`hotpotqa_runs/prompts.py`、`alfworld_runs/generate_reflections.py`。

**机制还原**：
- **失败→反思→重试**：`programming_runs/reflexion.py:33-52` `while cur_pass < pass_at_k and not is_solved:` 首试 `gen.func_impl(item["prompt"], model, "simple")` → `exe.execute(cur_func_impl, tests_i)` 得 `(is_passing, feedback, _)` → 若 `is_passing` 走 `exe.evaluate(..., item["test"], timeout=10)` 定 `is_solved`；`:55-79` `while cur_iter < max_iters:` `gen.self_reflection(cur_func_impl, cur_feedback, model)` → `gen.func_impl(strategy="reflexion", prev_func_impl=..., feedback=cur_feedback, self_reflection=reflection)` → 再 `exe.execute(...)`。反思记忆 `reflections` 累积，实现 `implementations` 累积。

**对表**：SP≈无；NC≈弱（执行器是模型/测试反馈，非密封评分）；ESS≈无（无显著性阈、无多次评分）；HA≈无。**可移植点**：`self_reflection` + 重试循环是 rsi-agent inner 策略更新的反面对照——它不判断「是否改进」，只判断「是否通过」。**本地可跑性**：纯 Python + 本地测试执行器，零 GPU/无 key 可跑（模型后端自配）。**缺口**：只读 `reflexion.py` 主循环，`immediate_reflexion.py`、prompts 模板与执行器 `executors/` 未读。

## 7. aiwaves-cn/agents（快速核）

**源码清单**（`sources/aiwaves-cn__agents/`）：`src/agents/optimization/loss.py`、`optimization/prompt_optimizer.py`、`optimization/prompt_formatter.py`、`task/node.py`、`task/sop.py`、`task/solution.py`。

**机制还原**：
- **loss 计算**：`optimization/loss.py:30` `class LossCalculator`，`:72` `calculate_loss(case)` 走 4 步（构 prompt → 调大模型 → 抽结果 → 存结果）；`:84-109` `case_data = case.get_dict_for_loss_calculation(...)`、`formulate_prompt(self.meta_loss, case_data)`、`OptimUtils.extract_data_from_response(content, self.meta_loss["extract_key"])`、`case.loss.update(**loss_data_dict)`；`:57-60` `meta_loss` 按 `has_ground_truth / has_eval_score` 分四种 prompt（`get_prompt_config_order`，`:141-156`）。
- **language gradient → 更新 prompt/节点**：`optimization/prompt_optimizer.py:73` `self.meta_backward = config.meta_prompt["backward"]`；`:130-184` `backward(case)`：构 backward prompt → 取 `suggestion` + `requirement_for_previous` → `state.backward.update(**backward_info_dict)`；`:212-247` `optimize_prompt(case_list)` **从最后一个 state 反向**（`for state_idx in range(case0_state_num - 1, -1, -1)`）构 `optim_prompt` → 抽 `["new_prompt","analyse"]` → 累积 `optim_info_list.insert(0, cur_optim_info)`，得到每个节点的新 prompt。
- **SOP 节点**：`task/sop.py:140+` `class SOP`，`:183` `init_name_role_hash_for_nodes`、`:194` `init_node_prompts`、`:198` `update_nodes_from_case`、`:205` `transit`、`:292` `route`。

**对表**：SP≈无（无密封闸，prompt 更新直接作用于节点）；NC≈弱（loss 由 LLM 评，无独立评分器）；ESS≈无（无显著性/配对）；HA≈无。**可移植点**：`optimize_prompt` 的「从后往前逐 state 重算新 prompt」可作 rsi-agent inner 策略更新的对照（它是纯 prompt 层梯度，无接受闸）。**本地可跑性**：Python + LLM 后端，零 GPU 可跑结构（需配 API）；`requirements.txt` 重。**缺口**：12649 blob，只读了 optimization/task 子集；`optimizer.py` 基类与 `evaluation/` 未读；"language gradient" 是 LLM 语义建议，非数学梯度，术语需注意。

## 8. madaan/self-refine（快速核，反面对照基线）

**源码清单**（`sources/madaan__self-refine/`）：`src/gsm/feedback.py`、`src/gsm/feedback_no_update.py`、`src/acronym/{feedback,task_iterate}.py`、`src/pie/feedback.py`。

**机制还原**：
- **最简自反馈**：`src/gsm/feedback.py` `GSMFeedback.__call__(solution)`：`entire_output = ...` → `improved_soln = entire_output.split("def solution():")[1]`、`feedback = entire_output.split("def solution():")[0]` → `update_prompt(solution, improved_soln, feedback)`；`update_prompt`（同文件下方）：把 `prefix + feedback + improved_soln + inter_example_sep` 拼成 `new_example`，`self.prompt = f"{self.prompt}{new_example}"`——**直接把 (feedback, improved_soln) 追加进 prompt 的 in-context 示例**。
- **对照变体**：`src/gsm/feedback_no_update.py:25-41` 同样拆 `feedback`/`solution` 但**不调用 update_prompt**，只 `return {"solution","feedback"}`——这是无更新基线。

**对表**：SP≈无（无任何接受闸，只要模型自评即采纳）；NC≈无（反馈即模型自评，无独立评分）；ESS≈无（无多次/显著性）；HA≈无。**这是 rsi-agent 的反面对照**：证明「无密封闸的自反馈」在原理上可运行但无审计/无显著性。**本地可跑性**：纯 Python + `prompt_lib` + OpenAI 后端，零 GPU、结构可跑（需 API key）。**缺口**：只读 gsm/acronym/pie 三个任务，未读全部 6 个任务的 `task_iterate` 与 `commongen`/`readability`。

## 9. mmtmn/Darwin-Godel-Machine（快速核，本地无 key 自改）

**源码清单**（`sources/mmtmn__Darwin-Godel-Machine/`）：`dgm.py`、`coding_agent.py`、`llm_withtools.py`、`tools/edit.py`、`utils/eval_utils.py`、`utils/git_utils.py`、`README.md`。

**机制还原**：
- **群体自改循环**：`dgm.py:18` `class DarwinGodelMachine`；`:49-64` `run()`：`select_parents()` → 对每个 parent `self_modify_agent(parent)` → `archive.append(child_record)`；`:68-95` `select_parents()` 按 `performance` 加权随机采样（`total_perf = sum(r.performance ...)`，`pick_val = random.random() * total_perf`，前缀和命中）。
- **自改 + 验证 + 归档**：`:97-143` `self_modify_agent(parent)`：`tempfile.mkdtemp` → `copy_codebase(parent)`（跳过 `.git`）→ 建 `AgenticSystem(self_improve=True, max_retries=3, num_candidates=1)` → `child_agent.forward()` → `child_score = evaluate_agent(child_agent, benchmark)` → `if not child_agent.is_valid(): shutil.rmtree(temp_dir); return None`（`:133-136`）→ 否则 `save_agent_code(...)` 入档。
- **patch 校验与回归测试**：`coding_agent.py:61-134` `forward()`：`regression_tests_summary = self.get_regression_tests()` → `while retry_count < max_retries ...:` `patch = self.get_current_edits()` → `is_valid, reason = self.check_patch_validity(patch)`（不通过则重试）→ `test_report = self.run_regression_tests(...)` → `test_score = get_report_score(test_report)` → 按 `test_score` 取 best patch。
- **评分**：`utils/eval_utils.py:9` `evaluate_agent(agent, benchmark_name) -> float`，`:21-22` 默认实现 `score = random.uniform(0.0, 1.0)`（**占位实现**）；`:24-41` `get_report_score(report)` `score = float(num_ok) / (num_ok + num_fail)`。
- **编辑工具**：`tools/edit.py:50` `validate_path(path, command) -> Path`、`:100` `str_replace_in_file`。

**对表**：SP≈弱（有 `check_patch_validity` + `is_valid` 功能校验 + 归档隔离 temp 目录，无路径白名单/沙箱）；NC≈弱（`evaluate_agent` 默认随机分数，真实评分需自接）；ESS≈无（无显著性/非回归）；HA≈无。**可移植点**：`copy_codebase` 跳过 `.git` + temp 目录隔离 + 失败即 `rmtree` 丢弃，是轻量 SP 形态；`is_valid()` 功能保持检查对应 rsi-agent 的密封确认闸的「功能不破」维度。**本地可跑性**：20 blob 的极小项目，纯 Python，`llm_withtools.py` 走本地 Ollama 无 key，零 GPU 可跑结构（需本地 LLM）。**缺口**：`evaluate_agent` 默认随机分（占位），真实评估实现未在仓库中；`coding_agent.py` 的 `check_patch_validity` 具体规则未逐行核；未读 `tests/`。

---

## 跨 repo 对表汇总

| 机制 | 有近似实现的 repo | 实现落点 | 可移植到 rsi-agent 的思路 |
| --- | --- | --- | --- |
| **SP 密封私集** | RSIGame、meclaw、Geneclaw、lemoz、mmtmn | RSIGame champion+marker；meclaw 统一变更门+沙箱；Geneclaw 5 层 gatekeeper；lemoz 验证闸+docker 沙箱；mmtmn patch 校验+temp 隔离 | ① RSIGame `decide_checkpoint` 的 gain 阈值 + `.complete` marker；② Geneclaw 的「denylist→allowlist→大小→secret→可疑模式」顺序 |
| **NC 非循环评分** | RSIGame、meclaw、lemoz | RSIGame 固定 replay 判 what changed；meclaw ledger 不可写；lemoz 不相交 held-out 段 + 独立 benchmark | ① meclaw「测量账本循环不可写」；② lemoz held-out 段与搜索段不相交 + `mutation_generations: 0` 冻结重放 |
| **ESS 自相关感知配对闸** | RSIGame、meclaw、lemoz | RSIGame 多 pass 中位数 + CHANGED_AT；meclaw `eval_min_batch`/`min_delta_pct`/`quality_floor`；lemoz 逐 benchmark 非回归 + `regression_tolerance` + `reject_score_ties` | ① RSIGame 多 pass 中位数应对 provider 方差；② lemoz `_passes_regression_filter` 的逐 benchmark 非回归 + 容差 + 打平即拒 |
| **HA 哈希链审计** | **无**（9 个全部缺失） | 最接近：meclaw append-only receipt；lemoz `_tree_sha256`（树哈希，非链式） | rsi-agent HA 无对标，是真实差异化；若需对标可参考 lemoz `_tree_sha256` 的排序树哈希做账本条目摘要，但需自行补 prev_hash 链接 |

## 本地可跑性汇总（零 GPU / 无 key）

- **零 GPU、纯结构可跑**：RSIGame（monitor 默认关）、Geneclaw（dry-run + pytest）、mmtmn DGM（本地 Ollama）、self-refine / reflexion / aiwaves（需自配 LLM 后端，结构可跑）。
- **需内核特性/Docker**：meclaw（Rust + Landlock/seccomp/cgroup）、lemoz DGM（Docker 沙箱，docker 缺失回退 subprocess）。
- **需重环境/外部 key**：Voyager（Minecraft + OpenAI key）。

## 诚实读取缺口

1. 所有源码经 `gh api trees` + `curl raw` 通道获取，**非 git clone**，未取 LFS / 历史版本 / 未合并 PR；只读默认分支 HEAD。
2. **meclaw 4013 blob、aiwaves 12649 blob 只读了机制相关子集**，未通读全库；`handlers/ledger.rs`、`optimizer.py` 基类、`evaluation/` 等未逐行核。
3. **RSIGame `loop.py`（115KB）未通读**，只核了 monitor/checklist/eval/checkpoint 子集与 `loop.py:2244-2270`。
4. **lemoz 负对照的统计显著性未做**：作者自述 small-sample（1 个 held-out 段、2 replicate），本轮未复算其 delta 的置信区间。
5. **HA 判定基于 grep 负面证据**（`prev_hash|hash_chain|merkle|block_hash` 全空），非逐文件人工审计，存在漏检小写变体的风险（已用 `-i` 覆盖）。
6. 各 repo 的 star 数、README 定位沿用上一轮 `notes/2026-11-19-rsi-projects.md`，本轮未重取（避免重复取证）。

## 外链

- 源码树：`https://github.com/WenyiWU0111/RSIGame` / `https://github.com/mmeyerlein/meclaw` / `https://github.com/Clawland-AI/Geneclaw` / `https://github.com/lemoz/darwin-godel-machine` / `https://github.com/MineDojo/Voyager` / `https://github.com/noahshinn/reflexion` / `https://github.com/aiwaves-cn/agents` / `https://github.com/madaan/self-refine` / `https://github.com/mmtmn/Darwin-Godel-Machine`
- 单文件：`https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<path>`
- 树接口：`https://api.github.com/repos/<owner>/<repo>/git/trees/HEAD?recursive=1`

> 本笔记仅源码阅读，未 commit / 未 push；源码原件落盘于 `analyses/tracks/rsi-deep/projects-more/sources/`，未做任何改动。
