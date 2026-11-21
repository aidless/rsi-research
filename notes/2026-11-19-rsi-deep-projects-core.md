# RSI 四项目源码级深读（2026-11-19 实测）

> 任务：为 rsi-agent（纯 Python 双层：inner 改策略 + outer 密封确认闸；四机制 SP 密封私集 / NC 非循环评分 / ESS 自相关感知 ESS 配对闸 / HA 哈希链审计）做源码级深读，纠正上一轮只读 README 元数据的不足。
> 通道：`gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1` 取树 + `curl -sL https://raw.githubusercontent.com/<owner>/<repo>/main/<path>` 取单文件（实测 200）。未用 git clone（坏代理 127.0.0.1:7900）。4 repo 默认分支均确认为 `main`。
> 纪律：所有 `file:line` 来自本轮实际下载并阅读的源码；没读到 / 看不懂的如实标注「读取缺口」；数字可复算、绝不编造。原始源码落在 `analyses/tracks/rsi-deep/projects-core/sources/<repo>/`，原件零改动。
> 外链：agentdescent https://github.com/ThinkFlowLab/agentdescent ｜ Godel_Agent https://github.com/Arvid-pku/Godel_Agent ｜ RSIAgent https://github.com/AetherLabsAI/RSIAgent ｜ ScienceBuddy https://github.com/Gen-Verse/ScienceBuddy

## 总览：一句话结论

| repo | 源码证实的核心机制 | 与 rsi-agent 四机制的关系 |
|---|---|---|
| agentdescent | shipped gate = 全 held-out 上的 Beta 后验 P(改进) + 回归护栏；可选 FalsificationAcceptance 三分区证伪闸，评分器由 aggregator 注入、policy 不能自评 | **同构度最高**，SP/NC 都有工程实现，可直接抄 |
| Godel_Agent | 自指 agent 用 `exec`/monkeypatch 改自己 policy，`action_read_logic` 可读任意模块源码（含 reward）→ **循环论证无隔离** | 反面教材：证明 SP/NC 失效会长啥样 |
| RSIAgent | Verifier = zero-history 独立复查 + Linux namespace/setpriv 真隔离 + 私有路径强制隐藏 + fail-closed | SP/NC 的最强工程版（OS 级），纯 Python 抄不了隔离但可抄接口 |
| ScienceBuddy | 双层交替：固定 model 改 harness ↔ 固定 harness 用 GRPO 训 model，靠 sha256 + immutable 发布锁每层；val 90 排除出交互与训练、test 90 不参与选择 | SP/HA 的强工程版，不可变发布可直接抄 |

---

## 1. ThinkFlowLab/agentdescent（约 220★，2026-11-19 仍在推）

### 源码清单
下载并阅读（路径 / 行数 / 核心函数）：
- `.github/scripts/review_gate.py` 272L — `decide(head, author, reports, policy)` :124（**PR review PASS 闸，非 RSI held-out gate**）
- `.github/scripts/risk_gate.py` 219L — `Gate.judge(files, truncated)` :91（**diff 风险分级 T0–T3，非提升闸**）
- `.github/scripts/controller.py` 458L — label 控制器（只打 `risk:T*` / `agent-pr` 标签，从不合代码）
- `.github/agent/review-policy.json` 6L — `reviewers=["Birfy"], selfReview=["Birfy"]`
- `agentdescent/acceptance.py` 246L — **`FalsificationAcceptance.accept(ctx)` :157（真正的证伪闸）**，`ProbeContext` :74
- `agentdescent/defaults.py` 456L — **`DefaultAcceptance.accept(ctx)` :359（shipped gate）**
- `agentdescent/evolution.py` 4687L — `held_out_size()` :2507、切分 :2645-2646、test 尾部切 :2654-2667、`propose()` 调用 :4320、`_build_engine` 传 train/held_out :3107
- `agentdescent/aggregator.py` 1864L — `_probe_context()` :1662、`evaluate` 闭包 :1698
- `tests/test_held_out_note.py` 186L（切分契约）、`pyproject.toml` 76L、`README.md`
- 下载但未逐行深读：`agentdescent/policies.py` 799L、`agentdescent/evolvespec.py` 1575L、`agentdescent/audit/gate.py` 512L、`tests/test_gate_reads_the_full_set.py` 121L

### 机制还原
**关键纠正**：任务点名的 `.github/scripts/{review_gate,risk_gate,controller}.py` 是 PR 工作流工具，不是 RSI 提升闸。`review_gate.decide()` 判的是「PR tip SHA 上有没有指定 reviewer 写的 `Verdict: PASS`」，最新报告为准（:188-201）；`risk_gate.Gate.judge()` 从 diff 路径/新增行/体积算 T0–T3，文档明说 "It never comes from PR text"（risk_gate.py:10），report-only。真正判「策略是否提升」的是核心模块。

**默认闸（DefaultAcceptance，defaults.py:359-397）** 伪代码：
```
base_s,base_f = ctx.base_counts; cand_s,cand_f = ctx.cand_counts   # 都是全 held-out eval_counts
baseline_post = BetaPosterior(prior.successes+base_s, prior.failures+base_f)   # :372
candidate_post= BetaPosterior(prior.successes+cand_s, prior.failures+cand_f)   # :373
for card in ctx.cards: candidate_post.observe_delta(card.before_after_delta)   # :376-379
delta = annealed_delta(base_delta, version, half_life)                          # :382
seed  = stable_hash((version, diff_id)) & 0x7FFFFFFF                            # :386 每个候选独立播种
p_improve = prob_improvement(candidate_post, baseline_post, samples, seed)      # :384
regressed = cand_full < base_full                                               # :388
if p_improve <= 1-delta or regressed: reject("below-threshold")                # :390
else: accept("committed")
```
类注释 :312-318 明确两个 gate 都读**全 held-out**（`eval_counts`），regression guard 曾误读 `cheap_eval` 子采样（4 样本能否决全集的 bug，已修）。

**证伪闸（FalsificationAcceptance，acceptance.py:157-221）** 三分区，全部必须非空：
- `target`：要求实测增益 `gain = cand_rate - base_rate` > min_gain（:185-198，否则 `no-target-transfer`）
- `archived`：任何逐任务回归即拒（:200-207 `archived-regression`）
- `falsification`：任何 pass/fail 判定改变即拒（:209-218 `falsification-moved`，判「过宽」）
- 评分器 `probes.evaluate` 由 aggregator 注入（`ProbeContext` :74-85），policy **不能自评**；若 `ctx.probes is None` → `AcceptDecision(..., defer=True)`（:159-166），**绝不退回 count-only 闸**。拒绝理由带具体失败 task id（`_ids()` :98-103）。

**切分与防泄漏（evolution.py）**：
```
cut = max(1, round(n*(1-frac))); held_out_size = max(1, n-cut)   # :2518-2519
cut = len(tasks) - held_out_size(...); train, held_out = tasks[:cut], tasks[cut:]  # :2645-2646
if test_frac: held_out, test = held_out[:-n_test], held_out[-n_test:]  # :2667 测试集从 held-out 尾部再切，任何 gate 都不测 test
raw_proposal = propose(mine.render(), task, output, score)  # :4320 task 来自 train → inner 只见 train
```
verifier 的 `eval_fn` 由 aggregator 注入 probe（aggregator.py:1680,1698 `{task.id: float(eval_fn(artifact,[task]))}`），策略无法给自己的候选打分。

### 对 rsi-agent 四机制对表
- **SP 密封私集**：有。`held_out` 与 `train` 位置切分，inner 的 `propose()` 只拿 train task（:4320）；额外 `test` 从 held-out 尾部切出，所有 gate 都不碰（:2654-2667）。**比我们强**：多一层 test。**比我们弱**：仍是 in-process 构造性隔离（同一函数边界），非 OS 级；靠"传什么参数"而非"物理不可达"。移植：`held_out_size()` + test 尾部切分几乎可原样搬。
- **NC 非循环评分**：有，且做得明确。`aggregator._probe_context()` 文档直说 "the evaluator comes from here, so a policy cannot score its own candidate"（:1663-1671）；`needs_probes` 且拿不到 probes 就 defer（acceptance.py:159-166）。**比我们强**：defer-on-unavailable 是 fail-closed 纪律，我们没有等价物。移植：把评分函数作为外部依赖注入 acceptance，缺测量就 `defer=True` 而非降级。
- **ESS 自相关感知配对闸**：无显式 ESS / Newey-West / 自相关校正。它用「同一 held-out 上 base vs cand 的 Beta 后验 + 每候选独立播种 Monte Carlo」（defaults.py:372-386）——是配对思想的工程近似，但无 ESS 阈值、无自相关估计。**统计严谨度比我们弱，工程复杂度也低**。可作我们 ESS 的对照臂，不能替代。
- **HA 哈希链审计**：部分。`seed=stable_hash((version,diff_id))`、`evidence_card_hash=stable_hash(tuple(card_ids))`（acceptance.py:245）、artifact digest、CAS 提交；证据存 `<ledger>/.git/provenance/` 工作树外（:44-46）。**比我们弱**：是"逐提交溯源 + 内容哈希"，不是逐决策哈希链。`audit/` 子模块下载了 `audit/gate.py` 但未深读，诚实留缺口。移植：把 provenance 存到工作树外的做法值得抄。

### 本地可跑性
真依赖：`pyproject.toml` Python≥3.9，唯一硬依赖 `numpy>=1.20`（PPI 估计器），CLI 与 gate 纯标准库；可选 dev/mcp/otel。`requirements.txt` 404（README 用 pyproject）。`README.md:24`：`agentdescent demo` = "a complete evolution, offline, no API key, ~10 s"；:57 "no API key, no dependencies"；:134-135 测试离线确定性、`pytest -q` 无需 key。**零 GPU 可跑，且有无 key 离线路径**——4 个 repo 里唯一能在本机直接验证的。

### 诚实的读取缺口
- `evolution.py` 4687L 仅读 split / propose / gate / engine 构造相关段落，未全文通读；`propose()` 的 task 来自 train 是据 :4320 + `_build_engine` 传参 :3107 推断，未追到 worker task sampler 的每一行。
- `policies.py` / `evolvespec.py` / `audit/gate.py` / `test_gate_reads_the_full_set.py` 已落盘未深读，HA 判断只基于 acceptance.py + defaults.py 可见部分。
- 未实际运行 `agentdescent demo`（任务要求只读源码判断、不跑长任务；demo 虽短也无必要跑）。

---

## 2. Arvid-pku/Godel_Agent（约 244★，2025-09-17）

### 源码清单
- `src/agent_module.py` 723L — `evolve()` :589、`execute_action()` :524、`action_evaluate_on_task()` :299、`action_read_logic()` :77、`action_adjust_logic()` :109、`action_run_code()` :193、`action_environment_aware()` :33、`solver()` :279
- `src/task_mgsm.py` 238L — `MGSM_Task.evaluate()` :113、`real_evaluate()` :77、`score_mgsm()` :156、`threshold=0.94` :14、`last_test_acc=0.` :15
- `src/logic.py` 181L — `get_source_code()` :20、`store_all_logic()` :103
- `src/goal_prompt.md` 41L、`src/main.py` 7L、`src/wrap.py` 11L、`README.md` 90L、`requirements.txt` 4L（下载未细读）

### 机制还原
自指循环伪代码（`agent_module.py`）：
```
reinit(): optimize_history = ["The logic of solver:\n"+action_read_logic("agent_module","solver")]  # :513-522
loop: evolve()
  messages = [system Principles=goal_prompt, system Environment=action_environment_aware(agent), *optimize_history[-10:]]  # :609-611
  response = LLM(o3, tools=action_functions, tool_choice=required)  # :613
  execute_action(response)  # :621, :524 分派 tool_call
    - action_adjust_logic()  → monkeypatch 内存中的 policy 源码  # :109, :544-545
    - action_run_code()      → exec(code, globals(), locals); globals().update(locals)  # :193, :222-223；bash 用 subprocess.run(shell=True) :239-245
    - action_evaluate_on_task() → feedback,acc = task.evaluate(solver); 若 acc>last_test_acc 存快照  # :299, :306-309
  optimize_history.append(tool result)  # :577-579 feedback 回到同一上下文
  若 action_counter["evolve"]>=100: sys.exit(1)  # :583-584
```
reward 实现（`task_mgsm.py`）：`evaluate()` seed0 洗牌取前 128 作 valid，valid_acc≥0.94 才跑 `real_evaluate()`（:147-148）；`real_evaluate` 用不相交 `examples[128:928]` 作 test（:81）；`score_mgsm` = 答案字符串归一后相等（:156-163）。改进信号 = 外部 accuracy，feedback 只回数字（info_list 被注释，:149/:152）。

### 对 Q2 的直接回答：policy 能否读到 reward 源码？**能，循环论证风险真实存在**
- `action_read_logic(module_name, target_name)` 直接 `importlib.import_module(module_name)`（:90）再 `logic.get_source_code(...)`（:102）——可 import **任意**模块、读任意函数源码，包括 reward/scorer 所在的 `task_mgsm.py` 与 `score_mgsm`。
- `action_run_code()` 用 `exec(code, globals(), locals_dict)`（:222-223）+ `subprocess.run(shell=True)`（:239-245），无沙箱。
- `action_environment_aware()` 枚举模块 globals / 实例属性方法（:58-73），agent 对自身与 reward 全局有完全自观。
- `action_adjust_logic` 的 guard（:128-138）只禁改 `action_call_llm`/`action_call_json_format_llm`、禁 solver 用 `time.sleep`、禁 `logging`——**没有任何一条禁止读或改 reward/scorer**。
- `goal_prompt.md:1,5` 明说 agent 有 "unrestricted access to modify logic, run code and manipulate environment"。
结论：外部 reward 虽在进程外"定义"，但源码与执行环境对自改 policy 完全透明，feedback 又回灌同一 optimize_history → 这是循环论证的教科书反面样本。

### 对 rsi-agent 四机制对表
- **SP 密封私集**：弱。有 valid(前128)/test(128:928) 不相交数据切分，但 policy 可读 scored 数据路径、可读 grader 源码，"密封"仅是数据切分不是信息隔离。**比我们弱很多**。
- **NC 非循环评分**：无。`feedback` 文本直接回同一 `optimize_history`（:577-579）+ evaluator 源码对 policy 可见 = 完全循环。**反面教材**，正是 rsi-agent NC 要防的失效模式。
- **ESS**：无。只有 accuracy 均值 + bootstrap CI（`real_evaluate` :107），无配对、无自相关、无 ESS。
- **HA**：无哈希链。有 `results/mgsm_<acc>` 源码快照（logic.store_all_logic :103, agent_module.py:307-309），是"按分数存档"不是审计链。
- **可移植思路**：正面只有「valid 达阈值才跑 test」的节流（:147-148）与不相交切分；其余作为 SP/NC 的反例说明——外部 reward 必须连同其源码对 inner 不可见。

### 本地可跑性
依赖 `openai`, `numpy`, `tqdm`, `bs4` 等（requirements 4L 未细读）；需 OpenAI API key 写在 `src/key.env`（README:61，`Agent.__init__` :317-320 与 `task_mgsm.py:17` 均读该文件）；模型 gpt-4o-mini / o3-2025-04-16。**无离线无 key 路径**。纯 Python、无 GPU 依赖，但需付费 LLM → 本机零 GPU 也跑不动（卡在 key）。

### 诚实的读取缺口
- `requirements.txt` 4L 未逐行读；`results/mgsm/agent_module/Agent.py` 418L 已下载未细读（是自优化产物样例，非机制核心）。
- `logic.get_source_code` 依赖的 `__source__` monkeypatch 由谁写入、何时写入未逐行追；`action_adjust_logic` :139-189 的 modify/add/delete 实现只读了开头 guard。
- 未实际运行（需 key，任务也只要求源码判断）。

---

## 3. AetherLabsAI/RSIAgent（约 614★，2026-10-02）

### 源码清单
- `core/verifier.py` 814L — `_opening()` :329、`VerifierSession(list)` :180、`build_verifier_system()` :146、`VERIFIER_SYSTEM` :177、`acquire_executor()` :221
- `core/verifier_runtime.py` 1345L — `_normalize_private_paths()` :297、`validate_verifier_look_path()` :320、`AgenticVerifierExecutor` :338、`_run_in_private_mount_namespace()` :699
- `env/verifier_control.py` 211L — `DEVICE` :26、`_DAEMON` :47、`VerifierControl` :130、`run_script()` :169、`provision_verifier_control()` :186
- `core/loop.py` 1805L — `run_attempt()` :546、`verify_independent(...)` 调用 :776、导入 :207-209
- `core/self_evolving_loop.py` 411L — `run_self_evolving_loop()` :166、`VERIFIER_PASS` :34、fresh verifier :200 / verify :231
- `core/actor.py` 716L — `EPISTEMIC_CORE` :14（audited surface 声明 :3-6）
- `core/checks.py`（下载，未细读）、`README.md`、`requirements.txt`

### 机制还原
`core/verifier.py` 文件头 :1-15 直接给出设计动机：
> "The semantic-entanglement failure ... showed that an actor's own checks inherit the actor's wrong assumptions, so more prompting cannot catch them. The structural fix is INDEPENDENCE: a zero-history model call that sees only the task instruction and the live machine — none of the actor's reasoning, evidence or checks — re-derives the requirements ... probes the machine read-only."
> :13-14 "THIS FILE IS AN AUDITED SURFACE (Constraint #0): tests/test_no_leakage.py scans it against the task blocklist on every change."

伪代码：
```
# prompt 层隔离
opening = f"TASK (as originally given):\n{instruction}{ctx}\n\n{budget} The machine is live. Begin."  # :347
system  = VERIFIER_SYSTEM  # "You have deliberately NOT been shown their work, their reasoning, or their assumptions" :59-62
verdict ∈ {pass, wrong, unverified}  # :92-106
# 执行层隔离（AgenticVerifierExecutor, :338-348）
private tmpfs workspace + drop all capabilities
effect_isolated: visible FS read-only + private PID/network/IPC/UTS namespaces
rollback_mirror: 在 QEMU transaction 内暴露 live namespaces，关闭时回滚
若保护原语不可用 → fail-closed as infrastructure, 绝不降级成无保护执行  # :347-348
# 私有路径隐藏
_normalize_private_paths: hide_actor_memory → append "/home/user/.memory"  # :299-300
validate_verifier_look_path: 命中私有路径 → (False,"That path is private to the Actor Agent.")  # :334
# 真隔离命令（_run_in_private_mount_namespace :699-747）
setpriv --reuid=... --regid=... --clear-groups --bounding-set=-all --inh-caps=-all --ambient-caps=-all --no-new-privs  # :725-728
nsenter --target {pid} --mount  # :740；private scratch 在 /dev/shm :720；helper "never exposed as an Agent action" :703
# 控制通道（env/verifier_control.py）
root-only virtio port /dev/virtio-ports/org.rsiagent.verifier.control (:26), chown 0:0 + chmod 0600 (:49-51)
QEMU socket 127.0.0.1:7101 "never published, nor bound to the guest's container-gateway interface" (:28-29)
model programs start under setpriv and env -i (:109-111)
```

### 对 Q3 的直接回答：进程隔离还是仅 prompt 约定？**两者都有，以真实 OS 级隔离为主**
- prompt 层：zero-history + "deliberately NOT been shown their work"（:5-6, :59-62）+ 只喂原始 instruction 与 live machine（:347）。
- 系统层：Linux mount/PID/network/IPC/UTS namespace + setpriv 丢全部 capability + 只读 FS + 私有 tmpfs + root-only virtio 控制通道 + fail-closed。
- 所以不是"仅 prompt 约定"；它是 rsi-agent 纯 in-process 构造性隔离的工程化大尺度版，边界从"约定"升级为"内核强制"。

### 对 rsi-agent 四机制对表
- **SP 密封私集**：强。Actor 私有路径（`.memory`）对 Verifier 强制隐藏（:299-300, :334）；verifier scratch 在独立 mount namespace，"impossible for the ordinary Actor namespace to resolve scratch contents between Verifier turns"（:703-707）。**比我们强**：OS 级 vs 构造级。移植：纯 Python 抄不了 namespace，但可抄"私有路径显式登记 + look 路径校验拒绝"的接口。
- **NC 非循环评分**：强。零历史 verifier 独立重导需求 + 只读探针，评分者与被评者的推理链物理/上下文双重分离。**但注意**：verifier 看的是 live machine（candidate 产物状态）而非 actor reasoning——它隔离的是"推理过程"不是"结果"，这点与我们看 artifact 一致。移植："audited surface + blocklist 扫描"防知识泄漏（actor.py:3-6）值得抄。
- **ESS**：无显式 ESS。它是 verdict + 多轮 probe 的判定式闸，非统计配对闸。
- **HA**：有审计痕迹（ArtifactSink 存 transcript、actionless fingerprint sha256、audited surface 扫描），但非哈希链。
- **可移植思路**：validate_verifier_look_path 的拒绝模式；"保护原语不可用即 fail-closed、不降级"的纪律；zero-history verifier 接口（只吃 instruction + artifact）。

### 本地可跑性
`README.md:183-184`：Python 3.12 + uv + Linux host + Docker + `/dev/kvm`；:193 需 `OPENROUTER_API_KEY`；:205-206 需 pinned OSWorld 镜像；:246-247 Linux 镜像约 167 GiB、Windows 约 157 GiB。**零 GPU 可跑（CPU VM），但不满足无 key / 离线，且重资产（KVM + 百 GiB 镜像）——本机控制台不可跑**。requirements.txt 349B 已下载。

### 诚实的读取缺口
- `verify_independent` 在 `core/verifier.py` 被 loop.py :207-209 import，但在已下载的 verifier.py 中 grep **未找到定义**（可能在未下载的模块或 re-export），其零历史调用的确切实现未逐行读到；结论基于 verifier.py 文件头声明 + `build_verifier_system` + `_opening` + loop.py 调用点，属"声明 + 接口级"证据，非函数体级。
- `tests/test_verifier_isolation.py`、`tests/test_no_leakage.py`、`config/roles/{actor,verifier}.yaml` 下载返回 HTTP 200 但 **0 字节**（代理异常），未读到——blocklist 具体内容、角色配置未知。
- `core/checks.py` 已下载未细读；`AgenticVerifierExecutor` 的 `_run_isolated`（model-authored 代码实际执行路径）与 rollback_mirror 细节未全读。

---

## 4. Gen-Verse/ScienceBuddy（约 426★，2025-09-17）

### 源码清单
- `src/simple_scibuddy/coevolve/loop.py` 241L — `run()` :50、双层交替 for :131-223、合约 :177-193
- `src/simple_scibuddy/coevolve/protocol.py` 305L — `atomic_json()` :26、`read_request()` :48、`publish_request()` :59、`claim()` :115、`checkpoint_manifest()` :128、`publish_result()` :165、`validate_result()` :208
- `src/simple_scibuddy/coevolve/phase.py` 320L — `harness_evolve()` :113、validation 排除声明 :125-131、baseline 测 test :200-205
- `src/simple_scibuddy/coevolve/selection.py` 62L — `select_candidates()` :8、防泄漏注释 :13
- `src/simple_scibuddy/data/dataset.py` 73L — `TaskDataset` :42、校验 split_counts+payload_hashes :54-58、`verify()` :38
- `src/simple_scibuddy/data/verifier.py` 28L — `grade_answer()` :8（只评 `<answer>` 提交）
- `src/simple_scibuddy/training/data.py` 84L — `prepare()` :9、`assigned_split()` :77、`split_policy="frozen_train_val_test"` :63
- `configs/train.toml`、`README.md`
- 下载未细读：`coevolve/{improver,program,worker,feedback,phase,evidence}.py`、`training/generator.py`

### 机制还原（双层交替协议，`loop.py:50-223`）
```
dataset = TaskDataset(...)  # 校验 split_counts + payload_hashes（dataset.py:54-58, loop.py:72-74 冻结载荷校验）
train = sorted(tasks("train"), key=digest([seed,0,id]))  # :96
validation_task_ids = [val ids]; 写 validation-split.json "excluded_from":["harness interactions","RL training"]  # :97-107
for phase in rounds:
    harness, cursor, summary = harness_evolve(root, phase_id, model, harness, train, ...)  # :156 固定 model 改 harness
    if not summary["improved"] and trigger!="always_debug": break  # :173-175
    directory = publish_request(root/"shared", f"r{round:04d}", harness, base_model_path=model,
                                harness_sha256, eval, reference_correct, contract)  # :194-209
    run_round(..., steps=30)  # :214-220 固定 harness 用 SkyRL GRPO 训 model
    result = validate_result(directory); model = result["model_path"]  # :221-222
```
**每层固定另一层怎么落地（回答 Q4）**：靠 `protocol.py` 的不可变消息传递——
- `publish_request` 把 `harness_sha256`、`base_model_path`、`eval`、`runtime_contract` 写成 `request.json`，`atomic_json(..., immutable=True)` 用 `os.link` 硬链发布（:26-45, :111），readiness 信号最后写；
- worker `claim()` 写不可变 `accepted.json`（:115-125）；`publish_result()` 要求**新 model_id + 新目录，绝不覆盖 M0**（:188-195 "Save M1 in a NEW directory; never overwrite M0"），写不可变 `result.json`；
- `validate_result()` 逐字段校验 request_sha256 / harness_sha256 / checkpoint 逐文件 sha256（:208-260）；checkpoint_manifest :128-162 列每个权重 size+sha256。
→ harness 阶段：`AutoTokenizer.from_pretrained(model, local_files_only=True)` + `cfg=dict(cfg, model=model)`（phase.py:123,133），model 冻结；RL 阶段：harness 用 sha256 锁死，只产出新 model。

**冻结 715/90/90 划分实现**：`configs/train.toml` `selection_tasks=90`、`seed=20260911`、`max_rounds=3`、`steps=30`；README:177 "frozen 715 Train / 90 Val / 90 Test"。落地：`TaskDataset.__init__` 用 manifest 的 `split_counts` + `payload_hashes` 双重校验（dataset.py:54-58），`format=="task-index-v1"` 直接拒绝要求先冻结（:49-50）；`training/data.py:16-18` 校验 validation_task_ids 必须等于显式 val split；:43-44 只按记录的 `split` 字段分 train/test；:63 写 `split_policy="frozen_train_val_test"`；:73-74 删除 val.parquet/panel.parquet 防误用；`assigned_split()` :77-84 强制"train 阶段只给 train、eval 阶段只给 test"。

**选择纪律**：`select_candidates()` :13 "Generate every hypothesis before revealing any validation results."（先生成全部候选再揭晓 val 分）；评分由外部 `evaluate()` 在 val 90 上跑（phase.py:245-261）；赢家须 `errors==0 and correct > parent`（selection.py:49）；test 90 只在 phase 起点跑 baseline（phase.py:200-205），train.toml 注释明说 "Test scores do not control candidate selection or the fixed stage schedule."

### 对 rsi-agent 四机制对表
- **SP 密封私集**：强。train/val/test 三层冻结；val 90 排除出 harness 交互与 RL 训练（loop.py:104）；test 90 不参与选择（train.toml 注释）；private reference 在 `evaluator/reference.json`（dataset.py:35-36），prompt 只含 public task（dataset.py:69-72），grading 在 host 侧（verifier.py:8-28 只评 `<answer>`）。**比我们强**：三层 + 不可变发布 + 载荷哈希。移植：三层冻结 + val 排除交互的声明写法。
- **NC 非循环评分**：有。候选"先全生成后揭晓 val"（selection.py:13）+ 外部 batch 评分 + host verifier；proposer 在评分前看不到 val 结果。**比我们强在工程化**。移植：把"先生成全部候选再统一评分"作为防泄漏模式。
- **ESS**：无显式 ESS。用 `correct > parent` 且 `errors==0` 的严格计数（selection.py:49），无统计配对/自相关校正。
- **HA 哈希链审计**：四者里最强。`atomic_json(immutable=True)` 的 os.link 不可变发布 + 每段 request/accepted/result 的 sha256 锁 + checkpoint 逐文件 sha256 + round 链 r0001→r0003。**但严格说是"逐段不可变 + 哈希锁"而非"逐决策哈希链"**。移植价值最高：`atomic_json` 不可变落盘 + sha256 锁可直接抄成 rsi-agent 决策记录的防篡改存储。
- **可移植思路**：`atomic_json(immutable=True)`（防覆盖）、`select_candidates` 防泄漏、三层冻结 + val 排除交互。

### 本地可跑性
完整实验需 Qwen3.5-4B + SkyRL(vLLM/Ray) + CUDA（`phase.py:124` 读 `CUDA_VISIBLE_DEVICES`）+ W&B（loop.py:51,110）+ improver 远程 API key（`train.toml` `SCIENCEBUDDY_IMPROVER_API_KEY`）。README:232 明确基于 SkyRL 扩展。**零 GPU 不可跑完整 co-evolve**；纯标准库的 protocol/selection/dataset 部分可读可单测，但训练/RL 阶段需 GPU+key。`docs/algorithm.md` 未读，本地无 key 演示路径（README:173 提到的 researcher-facing workspace demonstration）与本实验实现是两回事——诚实区分。

### 诚实的读取缺口
- `coevolve/{improver,worker,feedback,program,evidence}.py`、`training/generator.py` 下载未细读；`docs/algorithm.md` 未读（GRPO reward/credit assignment 细节未核实，只从 train.toml + protocol 推断 stages）。
- `data/verifier.py` 是 28L 简化 `grade_answer`；README:191 明说完整产品 verifier/host 源码**未包含在本仓库**，故"host verifier 独立评分"只能从 `dataset.verify()` 调用点 + 简化 grade_answer 佐证，未读完整实现。
- 715/90/90 数字来自 README:177 + configs/train.toml（`selection_tasks=90`）实测；`manifest.json` 本体是运行产物、不在 repo，split_counts 的具体 715 未从数据文件复算到（只从文档读）。
- 未运行任何实验（需 GPU+key，任务也只要求源码判断）。

---

## 横向结论：哪个最值得直接抄进 rsi-agent

**首选抄 agentdescent 的 `FalsificationAcceptance`（acceptance.py:157-221）+ `_probe_context` 注入模式（aggregator.py:1662-1701）**，理由：rsi-agent 与它同为纯 Python、无 OS 隔离、无 GPU，`FalsificationAcceptance` 的三分区（target 需增益 / archived 禁回归 / falsification 禁判定漂移）+ "评分器外部注入、policy 不能自评" + "拿不到 probes 就 defer 而非降级 count-only" 能**直接补强 outer 闸的 SP 与 NC**，且不改变 rsi-agent 的运行形态。具体抄三件：
1. `needs_probes` + `defer=True`：缺测量时明确挂起，绝不静默退回弱闸（对治循环论证的"无证据也放行"）；
2. 三分区 + 拒绝理由命名具体 task id（可证伪、可复盘）；
3. `probes.evaluate` 由外部注入、inner 无法触碰 scoring 函数。

**次选抄 ScienceBuddy 的 `atomic_json(immutable=True)` + sha256 锁（protocol.py:26-45, :208-260）** 补 rsi-agent 的 HA——把每次 outer 决策做成 os.link 不可变落盘 + 内容哈希，成本极低、纯标准库。

RSIAgent 的 namespace 隔离与 ScienceBuddy 的 GPU 训练链在零 GPU 控制台环境下不可照搬，只抄其接口纪律（zero-history verifier、先生成后揭晓、fail-closed），不抄实现。
