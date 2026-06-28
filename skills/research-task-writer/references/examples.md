# 范例：两份写到位的 task 文件

两个完整范例，覆盖轻重两端。照着感受"必填节怎么填、AC/CP/VF/NEG 怎么编号联动、轻量任务怎么缩放"。两份都全中文。

- [范例一：重型评测 / 代码任务（safety evaluator）](#范例一重型评测--代码任务safety-evaluator)
- [范例二：轻量论文修改任务](#范例二轻量论文修改任务)

> 提示：范例一是为了演示 verifier-first、反例测试、覆盖矩阵、证据闭环，写得比较满；真实任务按 SKILL.md「按任务轻重缩放」决定填多少。范例里的路径多为示意，落地时换成真实路径或标 TBD。

---

## 范例一：重型评测 / 代码任务（safety evaluator）

> 场景：为一个 VLA 安全评测项目写"实现 safety evaluator"的 task。重点演示数据契约、手算 golden、AC/VF/NEG 编号联动、覆盖矩阵、执行约束、完成证据包。

````markdown
# 任务 03：实现 safety evaluator（episodes → metrics + per-episode report）

## 状态

状态：进行中
最近更新：2026-06-28
一句话进展：数据契约与 AC/VF/NEG 已定稿，parser+schema（CP-1）已通过 VF-1/NEG-1；metric 与 invariant（CP-2）实现中。

## 目标

实现一个 evaluator：读入 policy rollout 的 `episodes.jsonl` 与 `safety_rules.yaml`，输出聚合 `metrics.json` 和 `per_episode_report.jsonl`。核心约束：metric 定义严格、不丢样本、不读 ground-truth、可被反例测试证明有效。不做训练、不做可视化。

## 背景

下游 rebuttal 需要可信的 unsafe-success 统计。此前评测脚本把失败 episode 从分母里静默移除，导致 success_rate 虚高被 reviewer 质疑。本任务重写 evaluator 并补齐 verifier，服务 `docs/safegen/实验结果.md` 的主表。

## 技术路线

### 方法

纯函数式 evaluator：`parse → per-episode judge → aggregate`。判定逻辑全部走 `safety_rules.yaml` 配置，不在代码里 hardcode rule。metric 计算与 IO 解耦，便于 golden 单测。

### 输入

- `episodes.jsonl`：每行一个 episode rollout；
- `safety_rules.yaml`：rule id、触发条件、严重级别；
- 无 ground-truth plan / future state 输入（防泄漏）。

### 输出

- `metrics.json`：聚合指标；
- `per_episode_report.jsonl`：逐 episode 判定结果；
- `rule_breakdown.csv`：每条 rule 的 violation count。

## 数据契约与不变量

### 输入契约

- 输入文件：`episodes.jsonl`、`safety_rules.yaml`
- 必填字段：`episode_id`、`success`(bool)、`steps`(list)、`violations`(list)
- ID 规则：`episode_id` 全局唯一、不可重排
- 缺失字段处理：缺 `episode_id` 或 `success` → 报错退出，禁止 silent skip
- 非法样本处理：`steps` 为空 → 记为 `dropped_episode` 并写入 dropped report，计入分母说明

### 输出契约

- `metrics.json` 必填字段：`total_count`、`success_count`、`violation_count`、`unsafe_success_count`、`unsafe_success_rate`、`dropped_count`
- 必须保留所有输入 `episode_id`（出现在 report 或 dropped report）
- 不允许把 failed/timeout episode 从 `total_count` 静默移除

### 关键定义

```
success：episode 达成任务目标，字段 success == true
safety_violation：episode 的 violations 非空
unsafe_success = success == true and len(violations) > 0
unsafe_success_rate = unsafe_success_count / total_count
```

### 手算 golden 数值例子

```
Golden G-1：
- ep_001: success=true,  violations=[r1]
- ep_002: success=true,  violations=[]
- ep_003: success=false, violations=[r2]
手算：total=3, success=2, violation=2, unsafe_success=1, unsafe_success_rate=1/3
期望：unsafe_success_count==1；unsafe_success_rate==0.333333 ± 1e-6
```

### 必须保持的不变量

- 每个输入 `episode_id` 在 report 或 dropped report 中恰好出现一次。
- `0 <= unsafe_success_count <= success_count <= total_count`。
- `rule_breakdown.csv` 各 rule violation 之和 == per-episode report 的 violation event 总数。
- 缺失关键字段必须报错，不得 silent skip。
- evaluation 不读取 ground-truth / 未来状态。

## 实现计划

| 模块 / 文件 | 作用 | 对应 AC | 状态 |
|---|---|---|---|
| `src/eval/parse.py` | 读取并校验 episodes / rules schema | AC-1 | 已完成 |
| `src/eval/metrics.py` | 纯函数计算聚合 metric | AC-2 | 进行中 |
| `src/eval/invariants.py` | ID 守恒 / count 约束检查 | AC-3 | 进行中 |
| `scripts/run_full_verifier.py` | 总验证入口，跑全部 VF/NEG | AC-1~AC-4 | 待办 |

## 指标与统计验证

### 指标定义

| 指标 ID | 名称 | 定义 / 公式 | 分母 | 对应 AC |
|---|---|---|---|---|
| M-1 | unsafe_success_rate | unsafe_success_count / total_count | total_count（含 failure，不移除） | AC-2 |

本任务 evaluator 是**确定性**的：同一输入与 rules，输出必须逐字段稳定，用 golden 精确比对（tolerance 1e-6）。不涉及统计型验证。

## 交付物

| 交付物 | 路径 | 对应 AC | 状态 | 产出命令 | 验证命令 | 说明 |
|---|---|---|---|---|---|---|
| evaluator 代码 | `src/eval/` | AC-1~AC-3 | 进行中 | — | `python -m pytest tests/ -q` | 三模块 |
| metrics 输出 | `runs/<id>/artifacts/metrics.json` | AC-2 | 未完成 | `python -m eval.run --episodes ... --rules ...` | `python scripts/run_full_verifier.py --task 03` | 聚合指标 |
| per-episode report | `runs/<id>/artifacts/per_episode_report.jsonl` | AC-3 | 未完成 | 同上 | 同上 | 逐 episode |

## 复现入口

- 发起新 run：见 `../运行命令.md`
- 完整命令 / 环境 / 数据：见 `runs/<id>/run_config.json`
- 验证证据：见 `runs/<id>/verification/manifest.json`
- 结果登记：见 `../实验结果.md`

## 验收标准

- [ ] AC-1：parser 读取 `episodes.jsonl`/`safety_rules.yaml` 并校验 schema；缺必填字段必须报错。
  - 验证：VF-1　反例：NEG-1　证据：runs/.../verification/
- [ ] AC-2：`unsafe_success` 与 `unsafe_success_rate` 按定义计算，golden G-1 精确匹配。
  - 验证：VF-2　反例：NEG-2　证据：runs/.../verification/
- [ ] AC-3：所有输入 episode_id 守恒；count 约束与 rule_breakdown 求和一致。
  - 验证：VF-3　反例：NEG-3　证据：runs/.../verification/
- [ ] AC-4：最小数据集端到端 smoke 跑通并产出全部 artifact。
  - 验证：VF-4　反例：（复用 NEG-1~3）　证据：runs/.../verification/

## Verifier 设计

| VF-ID | 覆盖 AC | 类型 | 命令 | 期望 | 日志 |
|---|---|---|---|---|---|
| VF-1 | AC-1 | schema | `python -m pytest tests/test_parse.py -q` | exit 0 | verification/VF-1.log |
| VF-2 | AC-2 | golden | `python -m pytest tests/test_metrics_golden.py -q` | exit 0 | verification/VF-2.log |
| VF-3 | AC-3 | invariant | `python scripts/check_invariants.py --report runs/<id>/artifacts/per_episode_report.jsonl` | exit 0 | verification/VF-3.log |
| VF-4 | AC-4 | smoke | `python -m eval.run --episodes tests/data/mini.jsonl --rules tests/data/rules.yaml --out /tmp/smoke` | exit 0 + 产出 artifact | verification/VF-4.log |

总验证命令：`python scripts/run_full_verifier.py --task 03`（失败 exit 非 0，打印每条 AC 覆盖状态）。

## Verifier 反例测试

| NEG-ID | 对应 VF | 覆盖 AC | 注入错误 | 期望 verifier 行为 | 命令 |
|---|---|---|---|---|---|
| NEG-1 | VF-1 | AC-1 | 删除某 episode 的 `success` 字段 | parser 报错、exit 非 0 | `python -m pytest tests/test_parse_negative.py -q` |
| NEG-2 | VF-2 | AC-2 | 把 G-1 expected 改成 0.5 | golden 测试失败被断言捕获 | `python -m pytest tests/test_metrics_negative.py -q` |
| NEG-3 | VF-3 | AC-3 | 从 report 删一个 episode_id | invariant 检查 exit 非 0 | `python -m pytest tests/test_invariants_negative.py -q` |

反例测试本身应 exit 0（它断言 verifier 对坏输入会失败）。禁止 `assert True`。

## AC / CP / Verifier 覆盖矩阵

| AC-ID | 摘要 | Checkpoint | Verifier | Negative | Evidence | 状态 |
|---|---|---|---|---|---|---|
| AC-1 | parser + schema | CP-1 | VF-1 | NEG-1 | runs/.../verification/ | 已通过 |
| AC-2 | metric 定义 | CP-2 | VF-2 | NEG-2 | TBD | 未完成 |
| AC-3 | 不变量 | CP-2 | VF-3 | NEG-3 | TBD | 未完成 |
| AC-4 | 端到端 smoke | CP-3 | VF-4 | （复用） | TBD | 未完成 |

## Checkpoint 切片

| CP-ID | 覆盖 AC | 目标 | 产物 | 报到点 | Verifier | 状态 |
|---|---|---|---|---|---|---|
| CP-1 | AC-1 | parser + schema | parse.py + 测试 | 提交 VF-1/NEG-1 日志 | VF-1 | 已完成 |
| CP-2 | AC-2, AC-3 | metric + invariant | metrics.py/invariants.py | 提交 VF-2/VF-3 + NEG-2/3 日志 | VF-2, VF-3 | 进行中 |
| CP-3 | AC-4 | 端到端 smoke | mini run artifact | 提交 run_config + manifest | VF-4 | 未开始 |

## 阻塞与边界

### 已有底座
- 已有：rollout 产出的 `episodes.jsonl` 格式、`safety_rules.yaml` 草案、pytest 框架。

### 本次落地
- 完成 parser / metric / invariant / smoke 与对应 VF、NEG。

### 仍未做
- 多 seed 统计聚合、可视化面板、与 baseline 的显著性对比（留任务 04）。

### 本任务不做（Non-goals）
- 不改 rollout 产出格式；不引入新依赖；不做 plotting；不顺手重构 `src/train/`。

### 越界处置
- 发现 rules schema 需要扩展 → 记入 `## 开放问题`，不在本任务擅自改格式。

## 执行约束

### 派活时必须注入
- 本 task 文件 + `docs/safegen/execution_contract.md` + 当前 CP-ID + 覆盖 AC-ID + 要跑的 VF/NEG + evidence 格式。

### 禁止行为
- 不改 golden / 测试来过验收；不硬编码 metric 结果；不 silent skip 缺字段或失败 episode；不把失败 episode 移出分母；未实际运行不得声称通过；只跑部分 VF 不得声称全绿；无证据不得标已完成。

### 最终报告要求
- 完成的 CP-ID、覆盖 AC、改/增文件、跑的 VF/NEG、总验证命令 + exit code、evidence 路径、git commit/dirty、未解决问题。

## 完成证据包

> 当前未完成，本节待 CP-3 全绿后填。模板：

```
状态：已完成
最终验证：
- 总验证命令：python scripts/run_full_verifier.py --task 03
- exit code：0
- manifest：runs/2026-xx-xx_task03/verification/manifest.json
- stdout/stderr：runs/2026-xx-xx_task03/verification/
- artifact：runs/2026-xx-xx_task03/artifacts/
- git commit/dirty：见 run_config.json
覆盖：AC-1 VF-1+NEG-1 通过；AC-2 VF-2+NEG-2 通过；AC-3 VF-3+NEG-3 通过；AC-4 VF-4 通过
```

硬条件：全部 AC 有 verifier、关键 verifier 有反例、干净环境一次全绿、证据齐全，方可标"已完成"。

## 开放问题

- （初始为空）

## 备注

- 决策：rule 判定走 yaml 配置而非 hardcode，便于审计与扩展。
````

---

## 范例二：轻量论文修改任务

> 场景：回应 reviewer 对某指标的质疑，改论文若干处。没有 runs/，指标节与数据契约省略，verifier 用 grep + 人工审查，但仍保留 AC-ID 与可检查证据。即便 reviewer 意见和公式是英文，叙述正文仍全中文。

````markdown
# 任务 04：问题定义与攻击指标契约（回应 Reviewer 2010）

## 状态

状态：进行中
最近更新：2026-06-28
一句话进展：已定稿改写方案，待落到 AAAI draft 的问题定义、指标小节与 Algorithm 1。

## 目标

修正 reviewer 2010 的最高风险质疑：现稿用 `V_c(s + delta) > V_c(s)` 这类表达，会被读成"攻击直接改物理状态"。要把攻击重新定义为**观测攻击**，并把"攻击有效性"绑定到真实环境代价，消除"改物理态"的歧义。不重做实验、不改数值结果。

## 背景

reviewer 2010 指出攻击指标疑似用 `s + delta` 替换物理状态。这是可能导致拒稿的最高风险点：若指标与物理过程不符，结论站不住。服务于 AAAI rebuttal。

## 技术路线（修改思路 / 依据 / 产出形态）

- 思路：把攻击建模为观测攻击 `a_t = pi_E(o_t + delta_t)`，环境转移仍为 `s_{t+1} = f(s_t, a_t)`；有效性用真实环境代价 `sum_t c(s_t, a_t, s_{t+1})` 度量。全程区分受害观测、物理状态、学习代理状态三个对象。
- 依据：reviewer 2010 意见；现稿问题定义与指标小节。
- 产出形态：改写后的问题定义、指标小节、Algorithm 1 文本，加一个符号表。

## 实现计划（要改哪些章节 / 文件）

| 位置 | 改动 | 对应 AC | 状态 |
|---|---|---|---|
| AAAI draft §问题定义 | 攻击改为观测攻击，转移式不变 | AC-1 | 待办 |
| AAAI draft §攻击指标 | 有效性绑定真实环境代价 | AC-2, AC-3 | 待办 |
| AAAI draft Algorithm 1 | 扰动在代理模型上优化，仅经真实 rollout 评测 | AC-1 | 待办 |
| 符号表 | 新增 `s_t, o_t, delta_t, a_t, psi, c` | AC-1 | 待办 |

## 交付物

| 交付物 | 路径 | 对应 AC | 状态 | 说明 |
|---|---|---|---|---|
| 改后问题定义与指标小节 | AAAI draft（Overleaf）| AC-1, AC-2 | 待办 | 消除"改物理态"歧义 |
| 改后 Algorithm 1 与符号表 | AAAI draft | AC-1 | 待办 | 扰动优化与评测解耦 |

## 复现入口 / 核对入口

无 runs/。改动落在 AAAI draft 的问题定义、攻击指标、Algorithm 1 三处；逐条对照下面 AC 检查。diff 用 Overleaf 历史或 `git diff paper/`。

## 验收标准

- [ ] AC-1：正文无任何方程暗示攻击者直接改物理状态。
  - 验证：VF-1（grep）+ MANUAL-1（人工审查）　证据：diff + 审查记录
- [ ] AC-2：学习到的约束分被描述为攻击目标，而非最终评测指标。
  - 验证：MANUAL-2　证据：审查记录
- [ ] AC-3：每个上报的攻击结果都绑定真实环境代价、violation rate、return。
  - 验证：VF-2（grep）+ MANUAL-2　证据：diff

## Verifier 设计 / 反例审查

| VF-ID | 覆盖 AC | 类型 | 命令 / 方式 | 期望 |
|---|---|---|---|---|
| VF-1 | AC-1 | grep | `rg -n "s *\+ *\\\\delta|s_t *\+" paper/` | 无命中（除符号表定义处） |
| VF-2 | AC-3 | grep | `rg -n "real .*cost|violation rate|return" paper/sec_metric.tex` | 有命中 |
| MANUAL-1 | AC-1 | 人工审查 | 对照 Reviewer 2010 原文，确认歧义已消除 | 审查通过 |
| MANUAL-2 | AC-2,3 | 人工审查 | 确认约束分被定位为攻击目标且结果绑定真实代价 | 审查通过 |

反例审查：失败反例 = 只删了 reviewer 提到的句子但没解释 contradiction，或换了符号但指标仍隐含改物理态。审查方式：对照 Reviewer 2010 原文与修改后 paragraph；证据：diff + 人工确认记录。

## AC / CP / Verifier 覆盖矩阵

| AC-ID | 摘要 | Checkpoint | Verifier | Evidence | 状态 |
|---|---|---|---|---|---|
| AC-1 | 无"改物理态"暗示 | CP-1 | VF-1 + MANUAL-1 | diff | 未完成 |
| AC-2 | 约束分=攻击目标 | CP-1 | MANUAL-2 | 审查记录 | 未完成 |
| AC-3 | 结果绑真实代价 | CP-1 | VF-2 + MANUAL-2 | diff | 未完成 |

## Checkpoint 切片

| CP-ID | 覆盖 AC | 目标 | 报到点 | 状态 |
|---|---|---|---|---|
| CP-1 | AC-1~AC-3 | 三处改写一次到位 | 提交改后 draft + grep 结果 + 人工审查记录 | 未开始 |

## 阻塞与边界

- 已有：现稿问题定义、攻击指标、Algorithm 1。
- 本次落地：三处文本改写 + 符号表。
- Non-goals：不重做实验、不改数值、不动 related work、不重排章节。
- 越界处置：若发现需要补做实验佐证 → 记入 `## 开放问题`，另起任务。

## 完成证据

状态改"已完成"时填：改后 draft 链接 + `rg` 命中结果（VF-1 无命中、VF-2 有命中）+ 人工审查记录 + diff。

## 开放问题

- （初始为空）

## 备注

这是最高风险的 AAAI blocker；其余 reviewer 质疑围绕它展开，优先处理。
````
