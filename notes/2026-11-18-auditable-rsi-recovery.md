# Auditable RSI 论文正文 `main.tex` 找回报告

> 2026-11-18 由 lead 委派 rsi-recover 子代理执行。任务：找回 `Auditable Recursive Self-Improvement: A PDP-style Formal Framework for Bi-Level Agent Systems` 的正文 `main.tex`，核实身份，判断是否可用于重启研究。
> 身份锚点：review-packet 记录 `primary_source = <WORKSPACE_ROOT>/.research/bi-level-pepa/main.tex`，`sha256 = a40d6df56db984b9178bab76d70d90fea36a71d441896273d1b1ee795be44dab`。

## 结论（先给答案）

**正文完整找回，hash 完全匹配，身份确认，可直接用于重启研究。**

- 恢复位置：工作区内相对路径 `recovered/auditable-rsi/`（47 个文件，整投稿包逐字拷入，未改写任何字节）。
- `recovered/auditable-rsi/main.tex` sha256 = `a40d6df56db984b9178bab76d70d90fea36a71d441896273d1b1ee795be44dab`，与目标**逐字相同**（`cmp` PASS）。
- 真源：`<DATA_ROOT>`；`$HOME/.research/bi-level-pepa/main.tex` 与 `$HOME/.research/imported/bi-level-pepa/main.tex` 都是它的软链视图，三份哈希一致。
- 不是扩展摘要骨架：这是一份 **37 页完整投稿包**（`main.pdf` 37 页，7 正文节 + 7 附录节 + 5 图 + 24 条文献 + CLAIM_LEDGER + evidence/ + pre_registration/ + 全部 section `.tex`）。grade55 的 `words=507` 是**单文件 wrapper 的口径**，不是全文词数（详见第五节）。
- 会话记录恢复路线（dsh skill）**失败**：689 个会话记录里 5 个含目标关键词，但没有任何 `path` 指向 `bi-level-pepa/main.tex` 的 diff——正文从未在会话里被编辑过。最终在整机备份搜索阶段于 `$HOME/.research`（主目录，非工作区）定位到真迹。

## 一、实测表

| 检查项 | 判据 | 实测 | 判定 |
|---|---|---|---|
| 工作区内 `.research/` | 用户称不存在 | `ls .research` → No such file or directory | 确认 |
| 工作区内同名残留 | find 结果 | 只有 `aidless-maintenance/patches/H1-seed-readme/bi-level-pepa/README.md`（Status: reserved 空槽）与 `ci-delivery/workflows/bi-level-pepa.yml` | 均非正文 |
| 会话记录扫描 | dsh skill 步骤 1–2：zstd 流式全扫 | 689 个 `session*.jsonl.zstd`；5 个命中关键词会话（7e6711d6 / cf26fca2 / 6dffe4f1 / 9af32b87 / ddf54f92）；递归找 `{path,oldText,newText}` diff → **0 条**指向 main.tex；0 条 `\documentclass`+Auditable 全文 | 会话恢复失败 |
| 早先找回会话 7e6711d6 | 其结论 | 只找到占位 README，未找到正文 | 与本次一致 |
| 备份根 1：`/data2/backups/` | 两个 `.tar.zst`（dsh 配置备份） | 1448 / 1591 条目，无任何 `.tex` / bi-level / auditable 条目 | 无正文 |
| 备份根 2：`dsh-*-backup-*` | 两个目录 | 只有 `cordis.patch.yml` / `package.json` / `pnpm-workspace.yaml` | 无正文 |
| 备份根 3：`bundle-snapshot/` | 标题锚点 grep | 0 命中（8fam 迁移包，与本稿无关） | 无正文 |
| 关键定位：`$HOME/.research/bi-level-pepa/` | sha256 比对 | `main.tex` → 真源软链 `data3/.../tmlr_submission/main.tex`，hash `a40d6df5...` | **命中** |
| 真源 hash | review-packet 目标 hash | `a40d6df56db984b9178bab76d70d90fea36a71d441896273d1b1ee795be44dab` | 完全一致 |
| 恢复件 hash/cmp | 与真源逐字节 | `a40d6df5...`，`cmp` 无输出 | 完全一致 |
| 配套完整性 | README 文件清单 | 7 正文 section + 8 附录 + references.bib + figures/ + evidence/ + pre_registration/ + 4 份 `.bak` + main.pdf/bbl/blg 全在 | 完整投稿包 |

## 二、证据分级（按 `recovered-artifact-lineage-proof` 五级阶梯）

本任务沿用该 skill 的阶梯顺序，把「哈希匹配」当作最强一级（原始副本对账）。到达第 5 级，停在第 5 级，`PROVEN`。

| 级 | 判据 | 本案读数 | 是否单独定身份 |
|---|---|---|---|
| ① 包含关系 | 消费方按声明的路径找到候选 | `extract_meta.py:5` 写死 `IMP = "<HOME>/.research/..."`；review-packet `primary_source = <WORKSPACE_ROOT>/.research/bi-level-pepa/main.tex`；两处都指向 `$HOME/.research` 下的软链视图，真源同一 | 能：这是消费方实际读的那一份 |
| ② 血缘自述 | 包内 README 自述 | `00README.md`：「Title: Auditable Recursive Self-Improvement...」「24 references」「7 main body sections」「AIDE85 lineage not yet released」——与 grade55 摘要末尾逐字吻合 | 只到「同一课题」 |
| ③ 接口闭合 | 候选字段 ⊇ 消费方读取 | `extract_meta.py` 读 `title`/`abstract`/`sections`/`words`/`bibitems`/`figs` 六项；恢复件里六项全可复算（见第五节） | 加分，不单独定身份 |
| ④ 实跑 | 编译产物对账 | 同一目录 `main.pdf` 37 页（Oct 6 编译），`main.bbl`/`main.blg`/`main.log`/`main.fdb_latexmk` 齐全 | 加分 |
| ⑤ 原始副本 sha256 对账 | 与 review-packet 记录的 hash 逐字比 | 三份（真源、两个软链视图）+ 恢复件，四份 sha256 全为 `a40d6df5...`；`cmp` 无输出 | **能，最硬证据：逐字节相同** |

循环证据闸检查：①的前提「锚点上游 mtime 早于追回动作」成立——真源 mtime = `2026-10-23 09:40:44 -0700`，早于 review-packet 生成时间 `2026-10-28T12:14:02Z` 和本次恢复（11-18）；恢复件是新目录写入，未被任何会话重复指派来源；无双向追回记录，无 `applied=false` 计划当事实。判决 `PROVEN`，非 `CIRCULAR`。

## 三、meta.json 摘要能重建到什么程度

| 内容 | 状态 | 证据 |
|---|---|---|
| title（111 字符 LaTeX 原文） | 逐字可恢复 | meta.json 里保存；恢复的 `main.tex:34-35` 同串 |
| abstract（1581 字符去 TeX 命令后的纯文本） | 逐字可恢复 | meta.json / digest.md 里保存；P1/P2/P3、AIDE、0.9957±0.0027、0.9951±0.0031、0.7、1000 等全部数字 token 与恢复正文一致 |
| sections（7 个标题） | 逐字可恢复 | meta.json 里保存；`main.tex` 里 7 个 `\section{}` 同名 |
| 正文各 section 的实际段落 | 原本只能靠摘要间接重建 | 现已由真源直接恢复（`section_1_formatted.tex` … `section_11_*.tex`），无需重建 |
| 论文的数字/图表/证明/文献 | 摘要只能给出少数数字，图表与证明全丢 | 现已由真源直接恢复（`figures/`、references.bib、附录证明 `.tex`） |

结论：即使没有真源，title/abstract/sections 可逐字重建，但正文段落、图、证明、文献表会永久丢失——摘要骨架无法替代正文。本次真源找回后此项仅作盘点，不需重建。

## 四、元数据一致性核实（回应 lead 的补充线索）

用 grade55 生成时的同款脚本 `.agent-memory/grade55/extract_meta.py:24-52` 的口径，对恢复的 `main.tex` 现算：

| 指标 | grade55 记录 | 现算（同一口径） | 一致？ | 说明 |
|---|---|---|---|---|
| title | `\textbf{Auditable Recursive ... Bi-Level Agent Systems}` | 同串 | ✓ | |
| bibitems | 24 | 24（`references.bib` 中 24 条 `@...{`） | ✓ | |
| figs | 5 | 5（5 个 `\includegraphics`） | ✓ | |
| words | 507 | 582 | ✗（差 75） | `main.tex` 在 9/8 生成 meta 之后、10/23 被改动过（`bak_identity_20261023` 等 4 份备份可追）；当前 10/23 字节正是 10/28 review-packet 记录的 `a40d6df5` 版本，hash 匹配优先于 words 近似 |
| sections | 7（Introduction…Conclusion） | 7 个同名 + 4 个附录节 | ✓ | 附录节不进 grade55 口径（`[:20]` 但 wrapper 里附录用 `\section*`，正则不匹配） |

**这是完整正文，不是扩展摘要骨架。** `paper-portfolio.md:90` 的「0 页」标记是该稿在 portfolio 里未编译/未登记的状态，与真源 37 页 PDF 矛盾；以后者为准（live 产物 > 登记状态）。

## 五、检查过的完整路径清单（诚实留痕）

会话记录（dsh skill 全量，5 命中）：
- `<DSH_SESSIONS>/<SESSION_UUID>`
- `<DSH_SESSIONS>/<SESSION_UUID>`（本次会话及父会话，只有任务描述）
- `<DSH_SESSIONS>/<SESSION_UUID>`
- `<DSH_SESSIONS>/<SESSION_UUID>`
- `<DSH_SESSIONS>/<SESSION_UUID>`
- 其余 684 个会话记录字节扫描无关键词命中；`sessions-rescued/` `sessions-trash/` 无 `.jsonl.zstd` 文件；无 `sessions.bak-*` 目录。

备份根（三处指定）：
- `/data2/backups/dsh-config-20261118T022036Z.tar.zst`（1448 条目，无 .tex）
- `/data2/backups/dsh-config-20261118T115430Z.tar.zst`（1591 条目，无 .tex）
- `<HOME>/...`（cordis.patch.yml / package.json / pnpm-workspace.yaml）
- `<HOME>/...`（同上）
- `papers/verification_selfaudit/migrations/8fam-20261019/bundle-snapshot/`（20 个 patch + 20 条 commit message + hook，标题锚点 0 命中）

工作区与 $HOME 内候选目录：
- `<WORKSPACE_ROOT>/.research/bi-level-pepa/` —— 真迹软链视图，`main.tex` hash 匹配
- `<HOME>/.research/...` —— 真迹软链视图（grade55 实际读取处），hash 匹配
- `<DATA_ROOT>` —— **真源**（真身），hash 匹配
- `<HOME>/...` —— GitHub 仓库骨架（README/LICENSE/STATUS 等），无 main.tex
- `<HOME>/...` —— 同上，无 main.tex
- `<WORKSPACE_ROOT>/aidless-maintenance/patches/H1-seed-readme/bi-level-pepa/` —— 只有 reserved 空槽 README.md
- `<WORKSPACE_ROOT>/ci-delivery/workflows/bi-level-pepa.yml` —— CI 工作流，非正文

## 六、下一步（可执行）

1. 从 `recovered/auditable-rsi/` 直接重启研究：以 `main.tex` + 7+ 个 `section_*.tex` + `references.bib` + `figures/` 为 source of truth；`main.pdf`（37 页）作对照。
2. 用 `CLAIM_LEDGER.md` 与 `evidence/` 核对 claim 状态；P2 的 empirical claim 按原文披露 awaits AIDE85's lineage release，重启时不要把 synthetic 结果当实证。
3. 若要把恢复件登记进 portfolio：更新 `paper-portfolio.md:90` 的「0 页」为「37 页」，并附本报告与 sha256。
4. 若要把 source 放回工作区 `.research/bi-level-pepa/`：用 `cp -a` 从 `recovered/auditable-rsi/` 拷入，保持字节不变；不要改写历史版本备份。
5. 不要 git commit/push（本次未做任何提交）。

## 落盘回执

- 写前：`pw notes/2026-11-18-auditable-rsi-recovery.md` → ALLOW。
- 恢复件：`recovered/auditable-rsi/main.tex` sha256 `a40d6df56db984b9178bab76d70d90fea36a71d441896273d1b1ee795be44dab`，与真源 `cmp` 无输出。
- 未改写任何原始文件；未 git commit/push。
