# 任务 <编号>：<简短标题>

<!--
使用说明（填完删掉本段注释）：
- 全中文，含章节标题。公式 / 代码标识符 / 路径 / 命令 / 配置字段 / 引用原文可保留原文。
- 先判断任务轻重，再决定填多少（见 SKILL.md「按任务轻重缩放」）：
  · 重型代码 / 实验 / 数据任务：所有节填满，尤其是 数据契约、Verifier 设计、Verifier 反例测试、覆盖矩阵、执行约束、完成证据包。
  · 轻量写作 / 分析任务：可省略 指标与统计验证、数据契约，Verifier 反例测试可用人工审查替代；但必须保留 状态/目标/交付物/验收标准(AC-ID)/Checkpoint/Non-goals/核对入口/完成证据。
- 推荐先写 AC，再写 Verifier 与反例测试，最后写实现计划。不要先写实现再临时补验收。
- 三道执行闸门 + 两道防假完成：AC 逐条绑 verifier、Checkpoint 防漂移、Non-goals + 越界处置防过度工程；verifier 反例测试防"摆设 verifier"、完成证据包防"无证据谎报完成"。
- 编号约定：验收标准 AC-N，checkpoint CP-N，verifier VF-N，反例测试 NEG-N，指标 M-N，golden case G-N。
- 信息未定就写 `TBD：待确认` / `候选：<path>，尚未确认`，不要编造路径或命令。
- 派活给 agent 时，必须随 prompt 注入本文件 + execution_contract + 当前 AC/CP/VF/NEG，不能只丢一句"完成 task 03"。
-->

## 状态

状态：规划中 | 进行中 | 阻塞 | 已完成 | 已废弃
最近更新：YYYY-MM-DD
一句话进展：<现在做到哪一步、下一步是什么>

## 目标

<这个子任务要完成什么、达成什么、边界在哪。一两句话让人看懂。同时给出正向目标与不变约束（仍须满足什么）。>

## 背景

<为什么需要这个任务；依赖哪个上游任务、服务于哪个下游任务 / 论文 / 系统。轻量任务可省。>

## 技术路线

### 方法

<采用的技术 / 算法 / 模型 / simulator / real backend，以及关键设计取舍（为什么这么选、为什么不选替代方案）。框架名不是技术路线，要写"怎么用"。>

### 输入

<依赖的数据 / checkpoint / 环境 config / schema / 接口 / seed / baseline。列具体。>

### 输出

<产生什么：checkpoint / 指标 / 轨迹 / 视频 / 日志 / 场景文件 / 改后文本等。列具体形态。>

## 数据契约与不变量

> 代码 / 实验 / 数据 / 评测任务必填；纯写作类可省或改成"论点契约"。
> 本节固定输入、输出、指标和语义，防止实现过程中被默认脑补或悄悄改变。

### 输入契约

- 输入文件：
- 格式：
- 必填字段：
- 可选字段：
- 字段类型 / 单位 / 坐标系 / 时间步：
- ID 规则：
- split 规则：
- 缺失字段处理：<报错 / 跳过 / 填默认；禁止 silent skip，除非明确允许>
- 非法样本处理：
- 是否允许跳过样本：
- dropped sample report：

### 输出契约

- 输出文件：
- 格式：
- 必填字段：
- 是否保留输入 ID / 顺序 / split / seed：
- 是否允许新增字段：
- 是否允许跳过失败样本：
- per-sample report：
- aggregate report：

### 关键定义

<所有核心概念写成可判定定义，指标给出公式。>

- `<success>`：
- `<safety_violation>`：
- `<unsafe_success>`：`success == true and has_safety_violation == true`
- `<unsafe_success_rate>`：`unsafe_success_count / total_episode_count`

### 手算 golden 数值例子

> 关键 metric 不能只有文字定义，至少给 1–3 个手算例子，供 golden verifier 比对。

```text
Golden case G-1：
输入：<列出最小样本>
手算：<逐步算出关键 metric>
期望输出：<具体数值 + tolerance，如 0.333333 ± 1e-6>
```

### 必须保持的不变量

- 输入每个 sample_id / episode_id 必须在输出中有对应记录，除非记录 dropped reason。
- 总数类约束，如 `0 <= unsafe_success_count <= success_count <= total_count`。
- 确定性 verifier 在同一输入与配置下输出稳定。
- 缺失关键字段必须显式报错，不得 silent skip；过滤逻辑必须输出 dropped-count 与原因。
- <ML/Eval 任务追加>：train/val/test split 互斥、同一 id 不跨 split 泄漏、不读 ground-truth、method 与 baseline 用同一 test set 与 protocol、不用 test 调参、不静默移除失败 episode。
- <Robotics/Sim 任务追加>：scene config 与 metadata 一致、pose/scale/unit/frame 明确、joint/velocity/torque limit 不被绕过、sim seed/physics/controller config 进 run_config、不读 privileged state。

## 实现计划

<代码实现在哪里、需要新增或改动什么。用表格列真实代码位置，并标注对应 AC。路径必须真实或标 TBD。>

| 模块 / 文件 | 作用 | 对应 AC | 状态 |
|---|---|---|---|
| `src/.../xxx.py` | <负责什么> | AC-1 | 已完成 / 进行中 / 待办 |
| `scripts/xxx.py` | <脚本作用> | AC-3 | 待办 |

## 指标与统计验证

> 实验 / 评测任务必填；纯写作类可整节省略。

### 指标定义

| 指标 ID | 名称 | 定义 / 公式 | 分母 | 分子 | 单位 | 越大越好? | 对应 AC |
|---|---|---|---|---|---|---|---|
| M-1 | success_rate | success_count / total_valid_episode_count（分母含 timeout 与 execution failure，不静默移除） | | | | 是 | AC-2 |

### 确定性验证

<适用于 metric function / schema / parser / golden。写：输入、expected output、tolerance、验证命令、对应 AC。>

### 统计型验证

<适用于 RL / sampling / LLM eval / stochastic sim。>

```text
- seed 列表：
- 每个 seed 的 episode 数：
- 总 episode 数：
- baseline：
- 统计量 / confidence interval / bootstrap：
- tolerance：
- pass/fail 判据：
- flaky 处理 / 是否允许重跑 / 如何记录：
```

## 交付物

> 状态为「已完成」时必填，且路径必须指向 runs/ 中真实 artifact。

| 交付物 | 路径 | 对应 AC | 状态 | 产出命令 | 验证命令 | 说明 |
|---|---|---|---|---|---|---|
| <交付物名> | `runs/.../metrics.json` | AC-1 | 已完成 / 进行中 | `python ...` | `python scripts/validate_schema.py ...`（对应 VF-1） | <说明> |
| <代码/配置> | `src/.../xxx.py` | AC-2 | 已完成 | | | <说明> |

产出命令（生成上述交付物的代表性命令，可直接粘贴运行）：

```bash
python scripts/run.py --config configs/xxx.yaml --seed 0
```

验证命令（查看 / 核对交付结果，对应 VF-ID）：

```bash
python scripts/run_full_verifier.py --task <task_id>
```

## 复现入口

- 发起新 run / 常用运行方式：见 `../运行命令.md`
- 本次实际完整命令 / 代码版本 / 环境 / 数据：见对应 `runs/.../run_config.json`（复现单一真相，本文件不抄录）
- 本次验证证据：见 `runs/.../verification/manifest.json`
- 结果登记：见 `../实验结果.md`

<写作类任务无 runs/ 时改为「核对入口」：列出修改文件、diff 查看命令、reviewer comment 对应位置、验收 grep/diff 命令、人工审查记录。>

## 验收标准

<每条用 AC-ID 编号，可客观判断，并绑定 verifier / 反例 / 证据。完成 = 每条的 verifier 给出期望输出。禁止仅凭自述声称完成；不要写"做好为止"。>

- [ ] AC-1：<可客观判断的条件，如 evaluator 输出符合 `schemas/metrics.schema.json`>
  - 验证：VF-1
  - 反例：NEG-1
  - 证据：完成后指向 `runs/.../verification/`
- [ ] AC-2：<条件，如 `unsafe_success` 定义严格为 success==true 且 has_safety_violation==true>
  - 验证：VF-2
  - 反例：NEG-2
  - 证据：完成后指向 `runs/.../verification/`

## Verifier 设计

> 代码 / 实验 / 数据 / 评测任务必填。验收不能只检查"文件存在"或"脚本 exit 0"，按需分层覆盖。

| Verifier ID | 覆盖 AC | 类型 | 命令 | 期望结果 | 日志路径 | 说明 |
|---|---|---|---|---|---|---|
| VF-1 | AC-1 | schema | `python scripts/validate_schema.py ...` | exit 0 | `runs/.../verification/VF-1.log` | 检查输出 schema |
| VF-2 | AC-2 | golden | `python -m pytest tests/test_metrics.py::test_unsafe_success_golden` | exit 0 | `runs/.../verification/VF-2.log` | 检查手算 golden G-1 |
| VF-3 | AC-3 | invariant | `python scripts/check_invariants.py ...` | exit 0 | `runs/.../verification/VF-3.log` | 检查 count 与 ID 守恒 |

常见类型：`schema / unit / golden / invariant / property / leakage / statistical / smoke / end-to-end / regression / manual-review`。

总验证命令（一条跑完所有关键 verifier，失败 exit 非 0，输出每个 AC 覆盖状态，保存日志到 `runs/.../verification/`）：

```bash
python scripts/run_full_verifier.py --task <task_id>
```

## Verifier 反例测试

> 硬要求：每个关键 verifier 必须有反例测试，证明它对坏输入会失败（否则永远 return 0 的 verifier 也"通过"）。

| Negative ID | 对应 Verifier | 覆盖 AC | 注入错误 | 期望 verifier 行为 | 命令 | 说明 |
|---|---|---|---|---|---|---|
| NEG-1 | VF-1 | AC-1 | 删除 required field | schema verifier 返回非零 | `python -m pytest tests/test_schema_negative.py` | 证明 schema 检查不是摆设 |
| NEG-2 | VF-2 | AC-2 | 改错 expected metric | golden verifier 返回非零 | `python -m pytest tests/test_metric_negative.py` | 证明 metric 错误会被抓 |
| NEG-3 | VF-3 | AC-3 | 删除一个 episode_id | invariant verifier 返回非零 | `python -m pytest tests/test_invariant_negative.py` | 证明 ID 丢失会被抓 |

> 反例测试本身应在测试框架里"通过"——它断言 verifier 对坏输入返回非零 / 抛异常 / 输出错误诊断。禁止 `assert True` 式空反例，禁止只写"人工确认能失败"。

## AC / CP / Verifier 覆盖矩阵

> 必填。防止 AC 有 10 条、verifier 只覆盖 6 条却没人发现。

| AC-ID | 验收内容摘要 | Checkpoint | Verifier | Negative Test | Evidence | 状态 |
|---|---|---|---|---|---|---|
| AC-1 | schema 正确 | CP-1 | VF-1 | NEG-1 | TBD | 未完成 |
| AC-2 | metric 定义正确 | CP-2 | VF-2 | NEG-2 | TBD | 未完成 |
| AC-3 | 不变量成立 | CP-2 | VF-3 | NEG-3 | TBD | 未完成 |

约束：每条 AC ≥1 verifier；每个 CP ≥1 AC；每个关键 verifier ≥1 negative test；完成时每条 AC 的 Evidence 指向 runs/ 真实证据。只能人工审查的 AC 写成 `MANUAL-N`，注明审查对象、标准、证据位置。

## Checkpoint 切片

<把任务拆成 2–5 个有序纵切片，每片引用 AC-ID + 一个报到点（停下时产出什么 commit / 文件 / 输出）。
纵切片 = 一个小输入 → 一段核心实现 → 一个输出 → 一个 verifier → 一个 evidence；不是"先写全部代码再写全部测试"的横向分工。
执行 agent 每做完一片就在报到点贴证据：交互模式停下报到，后台模式 commit + 写 PROGRESS.md。>

| Checkpoint ID | 覆盖 AC | 目标 | 产物 | 报到点 | Verifier | 状态 |
|---|---|---|---|---|---|---|
| CP-1 | AC-1 | 完成 schema 与 parser | parser + schema test | 提交 schema 验证日志 | VF-1 | 未开始 |
| CP-2 | AC-2, AC-3 | 完成 metric 与 invariant | metrics + invariant report | 提交 golden/invariant 日志 | VF-2, VF-3 | 未开始 |

## 阻塞与边界

<必填。把"已有的、这次做的、还没做的"分开，并显式写出本任务的 Non-goals 范围栅栏与越界处置。>

### 已有底座

- 已存在的代码 / 数据 / 测试 / 运行脚本：

### 本次落地

- 本任务要完成 / 修改 / 新增：

### 仍未做 / 后续任务

- 留到后续：

### 本任务不做（Non-goals）

<显式列出栅栏外的事，把"理解偏差/过度工程"在 spec 层堵掉。越具体越好。>

- 不做 <X>；不改 <Y>；不评估 <Z>；不顺手重构 <W>

### 越界处置

如果执行中发现必须做 non-goal 范围内的事：1) 停止扩大范围；2) 写入 `## 开放问题`；3) 向用户/维护者确认；4) 不得自行把越界工作混进本任务。

## 执行约束

> LLM/agent 执行任务必填。写进 task.md 不等于 agent 会读——派活时必须把本节连同 task 文件、execution_contract、当前 AC/CP/VF/NEG 一起注入 prompt。

### 派活时必须注入

- 当前 task 文件；execution_contract（`docs/<project>/execution_contract.md` 或 `~/.claude/harness/execution_contract.md` / `AGENTS.md` / `CLAUDE.md`）；
- 当前要执行的 CP-ID、覆盖的 AC-ID、必须运行的 VF-ID / NEG-ID；
- 禁止行为；evidence bundle 格式；最终报告格式。

### 禁止行为

执行者不得：

- 为通过验收而修改测试 / golden / expected 输出，除非任务明确要求更新基准。
- 硬编码 golden case 的答案。
- silent skip 无效样本 / 缺失字段 / 失败 episode / timeout episode。
- 未经说明修改 public API、文件命名、目录结构或已有输出格式。
- 把 unrelated refactor 混进本任务。
- 用 mock / fake output 冒充真实实验结果。
- 在未实际运行命令时声称"已验证通过"，或只跑部分 verifier 就声称全绿。
- 在没有证据的情况下把状态改成"已完成"。
- 私自扩大范围解决开放问题；删除历史记录 / 旧计划掩盖偏离。

确需违反任一约束 → 先写入 `## 开放问题`，说明原因、影响、替代方案与建议决策。

### 最终报告要求

完成的 CP-ID / 覆盖 AC-ID / 修改文件 / 新增文件 / 运行的 VF 与 NEG / 总验证命令 + exit code / evidence bundle 路径 / artifact 路径 / git commit / git dirty / 未解决问题 / 偏离原计划之处。

## 完成证据包

> 任务标为「已完成」时必填。task 文件只写摘要，完整日志放 `runs/.../verification/`。

```text
状态：已完成

最终验证：
- 总验证命令：`python scripts/run_full_verifier.py --task <task_id>`
- exit code：0
- evidence manifest：`runs/<date>_<task>/verification/manifest.json`
- stdout / stderr：`runs/<date>_<task>/verification/`
- artifact：`runs/<date>_<task>/artifacts/`
- git commit / git dirty：见 `run_config.json`

覆盖情况：
- AC-1：VF-1 + NEG-1，通过
- AC-2：VF-2 + NEG-2，通过
```

标为「已完成」的硬条件：所有必需 AC 有 verifier 覆盖；所有关键 verifier 有 negative test；总验证命令在干净环境一次性全绿、exit 0；stdout/stderr/artifact/run_config 路径存在；git commit/dirty 已记录；coverage matrix 已更新；无未说明的 skipped test。

## 开放问题

<执行中冒出的、超出本任务范围或需要决策的问题往这里堆，供事后复盘 / 起新任务。初始可留空。>

- （初始为空）

## 备注

<关联决策、设计取舍、偏离计划之处（本想做 X、实际做了 Y、原因 Z、对下游影响 W）。
过时但有参考价值的内容用「（历史记录）」标注保留，不直接删。>
