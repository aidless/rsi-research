# RSI 深挖：Auditable Recursive Self-Improvement 的 claim-level 审计（2026-11-18）

> 用户指令链：m00002「rsi什么情况了」→ m00046「全做然后深入研究一下rsi」→ m00340「继续深挖」。
> 本轮把之前基于摘要的 P1/P2/P3 判断，推进到「找到 37 页正文 + CLAIM_LEDGER + 真源实验产物 → 隔离复跑 → 逐条判定」。

## 结论（先给）

三套实验本体都在真源项目的兄弟目录 `experiments/`，**推翻上一轮「P1 缺噪声下界本体实验」的判断**。隔离复跑 P1/P2，数字与落盘产物逐位一致。但深挖坐实四条硬判定：

1. **P1 成立且干净**：ESS 校正 ignition-test power 在 N_inner=5 全 ρ < 0.8，是真实的方法学结果。
2. **P2 是循环论证**：合成生成器里 `mu_priv = final_public_score + N(0,0.025)`，而特征向量第 0 维就是 `final_public_score`。AUC≈0.99 是把答案写进特征再预测出来，不构成关于 AIDE² 的事实。
3. **P3 是合成注入模拟 + 一项预注册偏差**：需要 DeepSeek API 的 800 次 judge 调用，本轮未复跑；结果里 hardcoded/crypto α 完全阻断、prompt-α 被绕过，有工程价值但仍是合成底物。
4. **投稿完整性有硬缺口**：论文「证据链」章节引用的 6 个 companion 文件在真源树上**全部不存在**（死链）；头表 AUC 0.9925 来自已丢失的 v0.4 归档会话，与 canonical 脚本复跑的 0.9970 第三位小数不一致。

作者自己的 `CLAIM_LEDGER.md` 把 C01–C03 全标 `inconclusive`、C04–C05 标 `blocked`，「What We Have Not Shown」明写无真实数据验证、无 ablation、无跨系统——**这篇稿件在口径上比摘要诚实，科学贡献被合成代理严重稀释**。

## 一、复现实测表（verifier = 隔离目录复跑，原件零改动）

复跑命令在 `<SCRATCH>/...`（脚本从真源读取，输出重定向到隔离目录，未覆盖任何原始产物）。

| 实验 | 预注册状态 | 复跑命令/入口 | 关键数字（复跑） | 与落盘产物 | 判定 |
|---|---|---|---|---|---|
| P1 | EXECUTED 2026-08-09 | `outer_loop_correlation.py --out <iso>` | verdict=P1 supported；N_inner=5 ESS power ρ=0.5→0.047、0.7→0.015、0.9→0.000；naive ρ=0.9→0.662 | 与 `p1_power_results.json` / REPRO 表逐位一致 | ✅ 数字可信 |
| P2 | 2026-08-08 复跑记录 | `mu_priv_meta_predictor.py --backend linear --n_agents 1000 --seed 0` | test_auc=0.9970；rank_corr=0.9908；test_mse=0.0007 | 与 `result_linear_seed0.json` / REPRO §2 逐位一致 | ✅ 数字可信；⚠ 口径循环 |
| P2 fold-CI | 2026-08-08 | `p2_fold_ci.py` → `p2_fold_ci.json` | Linear 0.9957±0.0027 / GB 0.9951 / MLP 0.9535；ρ=0.5→0.8499 [0.7959,0.8990] | 与正文 §4.4 / REPRO §5 一致 | ✅ 数字可信；⚠ 同源合成 |
| P3 | EXECUTED 2026-08-09（1 偏差） | `audit_channel.py`（需 DEEPSEEK_API_KEY，约 800 judge 调用） | 本轮未复跑 | REPRO 表：no-α +9.30、prompt-α +9.30、hardcoded/crypto 0.00 | ⚠ 未独立复跑 |

## 二、逐条 claim 判定

### P1 — 噪声功率下界（成立）
- 主张：N_inner≈5 的 per-candidate ignition test 在 AR(1) 噪声下，无论 ρ≥0 都欠功效。
- 证据：30 cells × 1000 MC，seed 42，d=0.5，α=0.05 双侧。ESS 校正 power 在 N_inner=5 全 ρ < 0.8（0.130→0.000）；naive Welch 随 ρ 从 0.106 虚高到 0.662，量化了「忽略自相关虚高 ignition power」的陷阱。
- 判定：**方法学上成立、可复算、干净**。n_eff 与论文解析表一致（ρ=0.7→0.88，ρ=0.9→0.26）。这是三条里唯一不依赖合成「把答案写进特征」的一条。

### P2 — μ_priv 可预测性（数字可信，论证循环）
- 主张：若 μ_priv 可由公开历史多项式时间算出，则 Level-2 RSI 不可能；AUC>0.7 即在合成数据上 falsify P2。
- **循环论证证据（本次新挖出的决定性一条）**：`mu_priv_meta_predictor.py` 生成器 `build_synthetic_dataset()` 里 `mu_priv = h.final_public_score + rng.normal(0, noise_sigma)`；而 `PublicHistory.to_feature_vector()` 第 0 维 = `self.final_public_score`。标签由特征直接加噪构造，AUC≈0.997 是同义反复，测的是「线性模型能拟合自己的输入」，不是「真实 AIDE² 私有分可被公开历史推断」。
- 诚实的边界：正文 §4.4 明写 synthetic / "directional, not confirmatory"；§7「What We Have Not Shown」明写真实 μ_priv 可预测性未证；ρ 敏感性、noise 扰动、5 模型类稳健都只在同一合成生成器内部变换。
- 判定：**作为管线自检成立，作为关于 AIDE² 的经验主张循环论证**。ρ=0.5 时 AUC 降到 0.8499 仍是同一生成器的产物。

### P3 — 反作弊等价于审计通道容量演化（合成 + 偏差）
- 主张：审计通道容量 C(α) 是承重机制；对 LLM-judged metric 的注入攻击应抬高 reward-hacking gap。
- DDB-Drill 结果：no-α +9.30、prompt-α +9.30（被绕过）、hardcoded-α 0.00（100/100 拒绝）、crypto-α 0.00（oracle）。
- 预注册偏差：Cohen's d 在零方差分离下未定义，主统计量改成 raw mean |judge−true| gap + bootstrap CI，已记入 `p3_results.json` 的 deviation_note 和卡片 changelog。
- 「L3 坏掉但 reward-hacking 仍从 42%→34%」被论文当作 P3 的外部呼应，但这是 Weco 博客的相关观察，作者自己在别处用 "a possible explanation" 定性，属相关非因果。
- 判定：**graded 支持（合成注入模拟），真实 lineage 的 4 条件 × 30 步 ablation 仍 pending**。本轮缺 API key 未复跑，列为未独立核验。

## 三、三个决定性发现

1. **合成循环（最重）**：P2 标签由特征构造，AUC 高是设计使然。任何在这种数据上做的模型类稳健、ρ 敏感、noise 扰动都逃不出同一生成器。真正能救 P2 的只有真实 AIDE85 lineage——而 Weco 至今未发布（论文自己承认）。
2. **证据链死链**：论文 §A「Evidence Chain」点名的 companion 文件——`p1_drill/ddc_results.txt`、`p2_drill/model_class_robustness.py`、`p2_drill/ddd_results.txt`、`p3_drill/drill_design.md`、`aide2_audit/precise_line_audit.md`——在真源树上全数不存在（本轮 find 逐一核实）。P2_REPRODUCIBILITY §3 也把这些标为 ❌「脚本缺失，待从归档恢复或重写」。这些是投稿时会被审稿人直接点破的死引用。
3. **头表数字口径分叉**：论文头表 AUC=0.9925，来源是已丢失的 v0.4 归档会话（`[archive]\2026-07-16-11-22-05\`）；仓库内 canonical 脚本复跑是 0.9970。REPRO §2 自己写「建议以 canonical 脚本重生成正式头表，统一为单一权威数字」。同一结论两个权威数字并存，是投稿前必须收口的一致性问题。

## 四、CLAIM_LEDGER 对照（作者自我定级）

| ID | Claim | Status | 本轮判定 |
|---|---|---|---|
| C01 | 框架形式化耦合双层优化 | inconclusive | 形式化自洽，无需实证；属定义不是发现 |
| C02 | AIDE² 实例化框架 | inconclusive | 映射是事后归类，缺 seal manifest |
| C03 | P1/P2/P3 泛化 | inconclusive | 本轮确认：三套实验都在但全是合成/AIDE² 单系统 |
| C04 | 建议经因果验证 | blocked | 无 intervention/ablation，尤其 P3 缺强制关 L1/L2 |
| C05 | 普遍安全保证成立 | blocked | 作者自己列为 unsupported，建议删 claim |

## 五、证据等级与边界

- **L1 本地复跑可复算**：P1（power 表）、P2（AUC 0.9970 + fold-CI 表）——本轮隔离复跑，原件零改动。
- **L2 落盘产物已核**：P3 结果 JSON / 三份 REPRODUCIBILITY / 两份预注册卡 / CLAIM_LEDGER / 证据 gaps。
- **L3 未独立核验**：P3 的 800 次 judge 调用（缺 API key）；论文头表数字 0.9925 的归档来源（文件已丢失）。
- **verifier 边界**：本轮所有「数字一致」判断来自脚本 stdout 与 JSON 字段逐位比对，不是目测；「文件不存在」来自 find 逐一返回空，不是印象。「P2 循环论证」来自直接读代码里 `mu_priv = final_public_score + noise` 这一行，是可指到行号的机制判读。
- 全程未改真源任何文件，未 git commit/push。

## 六、对既有判断的更正

- 上一轮（m00339）写过「P1 缺噪声下界本体实验；P2 只有 synthetic R²；P3 等价强于博客措辞」。本轮更正：
  - **P1 的本体实验存在且已复跑通过**（AR(1) 功率模拟），不是缺实验，是当时没找到真源兄弟目录。
  - **P2 的问题比「只有合成」更重**：是合成生成器内的循环论证，R²/AUC 高是构造使然。
  - **P3 的实验存在**（DDB-Drill 合成注入），带 1 项已申报偏差，但真实 lineage ablation 仍 pending 这一条不变。
- 一句话总结深挖增量：**从「证据够不够」升级到「证据是不是自证」——P1 是真证据，P2 是自证，P3 是合成演示，且投稿证据链有死链和头表口径分叉两处硬伤。**
