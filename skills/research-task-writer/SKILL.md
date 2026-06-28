---
name: research-task-writer
description: >-
  Use this skill whenever the user wants to create, write, update, or restructure a research-project
  task file or execution contract — including breaking a project into sub-tasks, documenting a task under
  docs/<project>/tasks/, recording a task's 技术路线/方法/交付物/deliverables/代码位置/复现命令/acceptance criteria,
  verifier plan, verifier negative tests, AC/CP/VF mapping, evidence bundle, or turning a rough plan into a
  maintainable task.md. Trigger even when the user does not say "task 文件" explicitly but is clearly
  documenting what a sub-task should do, where its code lives, how to reproduce it, how it is verified, or how
  it is accepted. Produces a clear, maintainable, fully-Chinese task document aligned with the docs/ 规格 ↔
  runs/ 复现单一真相 separation. Applies to ML / robotics / simulation experiment planning, benchmark/eval
  construction, paper-revision task tracking, and LLM/agent vibe-coding workflows.

  Do NOT use this skill when the user is only asking for conceptual research advice, paper idea brainstorming,
  code debugging, or implementation help, unless they clearly want the result turned into a task document,
  execution contract, verifier specification, or acceptance plan.
---

# 研究项目 task 文件写作指南

这个 skill 指导你为研究项目写出**清晰、可维护、可验证、可审计**的 task 文件，统一放在 `docs/<project>/tasks/` 下。

目标读者是几个月后的自己、协作者，以及执行代码任务的 LLM/agent。读者要靠这份文件搞清楚：

* 这个子任务要做什么；
* 为什么要做；
* 用什么技术路线；
* 输入和输出是什么；
* 代码在哪里；
* 交付物在哪里；
* 怎么复现；
* 怎么验证；
* verifier 本身是否可信；
* 什么证据能证明任务真的完成。

**输出一律全中文（含章节标题）。** 引用的英文 reviewer 原文、数学公式、代码标识符、路径、命令、配置字段可以保留原文，但所有叙述性正文必须用中文。

---

## 适用范围

使用本 skill 的典型场景包括：

* 把一个科研想法拆成可执行的 task 文件；
* 为 `docs/<project>/tasks/` 新建或更新任务文档；
* 为 ML / robotics / simulation / benchmark / eval 项目定义子任务；
* 为 LLM/agent vibe coding 定义执行契约；
* 为论文 revision plan 拆解任务；
* 为复杂实验写清楚技术路线、实现位置、交付物、复现命令和验收标准；
* 为 verifier / golden case / invariant / negative test / evidence bundle 写规格；
* 整理已有 task 文件，使其变得可维护、可验收、可复现。

不要在以下场景误用：

* 用户只是想讨论 research idea，不要求形成 task 文件；
* 用户只是想 debug 一段代码，不要求沉淀任务文档；
* 用户只是想要论文观点建议，不要求写 revision task；
* 用户只是想问概念解释，不涉及任务拆解、执行契约或验收。

---

## 核心目标

这个 skill 的目标不是写一份“看起来很完整”的文档，而是写一份**能约束执行、能防止假完成、能留下证据链**的科研任务契约。

一份合格的 task 文件应该做到：

```text
任务意图清楚；
范围边界清楚；
输入输出清楚；
代码路径清楚；
交付物路径清楚；
复现入口清楚；
验收标准可执行；
verifier 自身被验证；
完成状态有证据；
后人可以复查。
```

---

## 十条核心原则

写之前先把这十条记在心里。模板只是骨架，这些原则才是质量来源。

### 1. 全中文、可读

章节标题和正文都用中文。句子说人话，能用表格就别堆长段落。

一个没读过代码的人，光看这份文件也应该能复述出：

* 任务在做什么；
* 为什么要做；
* 做到什么程度算完成；
* 去哪里看代码；
* 去哪里看结果；
* 怎么验证结果是真的。

### 2. docs 说“规格”，runs 存“复现单一真相”

仓库复现架构是：

```text
docs/ = 稳定规格、设计意图、任务边界、验收入口
runs/ = 某次实际运行的完整命令、环境、数据、日志、artifact、验证证据
```

task 文件描述：

* 做什么；
* 为什么做；
* 怎么实现；
* 怎么验收；
* 交付物在哪里；
* 代表性的产出/验证命令是什么；
* 指向哪个 `runs/.../run_config.json` 或 verification evidence。

task 文件**不抄录**易过期的 run 细节，例如：

* 某次运行的完整命令行；
* `git_commit`；
* `git_dirty`；
* `git_branch`；
* Python / CUDA / GPU / dependency 环境；
* 数据 hash；
* 完整 stdout / stderr；
* 大量原始日志。

这些属于 `runs/`，task 文件只保留入口和摘要。

### 3. 交付物既要能定位，又要能复现

每个交付物必须写清：

* 交付物是什么；
* 真实路径在哪里；
* 如何生成；
* 如何验证；
* 与哪个 AC-ID 对应；
* 对应的证据在哪里。

路径和命令必须真实存在，或明确标注：

```text
TBD：待仓库确认
TBD：待实现后补充
候选路径：<path>，尚未确认存在
```

不得编造看似合理但仓库里不存在的路径。

如果当前上下文无法访问仓库或无法确认路径是否存在，必须保守写 `TBD`，不得声称“路径真实存在”。

### 4. 可维护，随进展更新

`状态` 和 `最近更新` 必须跟着真实进展改。

计划变了、对象换了、实验失败了，不要直接删旧内容。应保留决策轨迹：

```text
（历史记录）原计划做 X。
当前改为做 Y。
原因：Z。
```

偏离计划必须写进 `备注` 或 `状态`：

```text
本想做 X，实际做了 Y，原因是 Z，对下游影响是 W。
```

### 5. Verifier-first：先定义怎么验，再定义怎么做

对代码、实验、数据生成、评测类任务，必须先写清 verifier，再写实现计划。

验收不能只检查：

```text
文件是否存在；
脚本是否 exit 0；
有没有生成一张图；
有没有输出 metrics.json。
```

还要检查：

* schema 是否正确；
* 核心 metric 是否算对；
* golden case 是否完全匹配；
* 输入输出 ID 是否一一对应；
* 不变量是否满足；
* 数据 split 是否泄漏；
* seed / config / dataset 是否可追踪；
* 随机实验是否用统计方式验证；
* verifier 自己是否能抓住故意注入的错误。

没有可执行 verifier 的复杂任务，只能算“规划中”或“进行中”，不能标为“已完成”。

### 6. Verifier 也要被验证：每个关键 verifier 必须有反例测试

一个永远 `return 0` 的 verifier 也会“通过验收”。因此，必须证明 verifier 自己不是摆设。

每个关键 verifier 必须至少配一个：

* 反例测试；
* mutation test；
* corrupted input test；
* expected-failure case；
* negative case。

反例测试要故意注入错误，并证明 verifier 会失败。例如：

* 删除必填字段，schema verifier 必须失败；
* 把 expected metric 改错，golden verifier 必须失败；
* 删除一个 episode，ID coverage verifier 必须失败；
* 打乱 train/test split，leakage verifier 必须失败；
* 把 `unsafe_success_count` 改成错误值，metric verifier 必须失败。

注意：反例测试本身应该在测试框架里“通过”。也就是说，它断言 verifier 对坏输入会返回非零 exit code、抛异常或输出错误诊断。

### 7. “已完成”必须有证据闭环

不能只写“已运行，全部通过”。

任务状态改为 `已完成` 时，必须有 evidence bundle。证据包括：

* 总验证命令；
* exit code；
* stdout / stderr 的保存路径；
* artifact 路径；
* git commit；
* git dirty 状态；
* run_config 路径；
* verifier coverage matrix；
* 失败重跑记录，如有；
* 最终一次干净环境全绿记录。

task 文件里写证据摘要；完整原始证据放在 `runs/.../verification/` 下。

没有 evidence bundle 的任务，不得标为 `已完成`。

### 8. AC / CP / VF 必须有 ID 映射

验收标准、checkpoint、verifier 不能各写各的。必须有唯一编号和覆盖关系：

```text
AC-1, AC-2, AC-3...
CP-1, CP-2...
VF-1, VF-2...
NEG-1, NEG-2...
```

硬约束：

* 每条 AC 必须有唯一编号；
* 每个 checkpoint 必须引用至少一个 AC-ID；
* 每个 verifier 必须映射到至少一个 AC-ID；
* 每条 AC 至少被一个 verifier 覆盖；
* 每个关键 verifier 至少有一个 negative / mutation test；
* 没有 evidence 的 AC 不能算完成。

必须提供覆盖矩阵：

```text
AC-ID | Checkpoint | Verifier | Negative Test | Evidence
```

这样能防止 AC 有 10 条、verifier 只覆盖 6 条但没人发现。

### 9. 区分确定性验证和统计型验证

科研任务里很多输出不是 bit-level deterministic。RL、采样式 policy eval、LLM benchmark、simulation rollout、随机初始化训练都可能有波动。

不得简单地对所有任务要求：

```text
固定 seed 后输出 hash 完全一致
```

应区分两类验证：

#### 确定性验证

适用于：

* parser；
* schema；
* metric function；
* rule checker；
* golden case；
* 数据转换；
* 静态图表生成；
* deterministic smoke test。

验证方式：

```text
固定输入 → 精确输出；
expected JSON 完全匹配；
数值误差在明确 tolerance 内。
```

#### 统计型验证

适用于：

* RL training；
* policy evaluation；
* sampling-based generation；
* LLM judge；
* stochastic simulation；
* 多 seed benchmark。

必须写清：

* seed 列表；
* 每个 seed 的 episode 数；
* 最小样本量；
* 容忍区间；
* confidence interval 或 bootstrap 方法；
* 与 baseline 的比较方式；
* 失败判据；
* flaky 处理规则；
* 是否允许重跑，允许几次，如何记录。

例如：

```text
5 个 seeds，每个 seed 100 episodes。
报告 mean ± 95% CI。
如果 method 的 unsafe-success rate 高于 baseline 超过 3%，且 95% CI 不跨 0，则视为 regression。
```

### 10. 执行约束必须派活时注入，不能只埋在 task.md 中

写进 task.md 不等于 agent 一定会读。

对 LLM/agent 执行任务时，必须把执行约束作为派活 prompt 的强制前缀或附件，例如：

```text
~/.claude/harness/execution_contract.md
AGENTS.md
CLAUDE.md
docs/execution_contract.md
```

派活时至少附上：

* 当前 task 文件；
* execution contract；
* 当前要执行的 AC-ID / CP-ID；
* AC/CP/VF 覆盖矩阵；
* 禁止行为；
* evidence bundle 格式；
* 最终报告格式。

不要只对 agent 说：

```text
请完成 task 03。
```

应该说：

```text
请完成 task 03 中的 CP-2，对应 AC-3、AC-4。
必须遵守 execution_contract.md。
不得修改 tests/goldens 除非任务明确允许。
完成后必须运行 VF-3、VF-4 和对应 NEG-3，并保存 evidence bundle。
```

---

## 工作流程

### 1. 定位项目，沿用既有约定

先看目标项目的 `docs/<project>/` 目录：

* 有没有 `README.md`；
* 有没有 `运行命令.md`；
* 有没有 `实验结果.md`；
* 有没有 `tasks/`；
* 现有 task 文件如何编号；
* 是否已有 `runs/` 复现结构；
* 是否已有 `execution_contract.md` / `AGENTS.md` / `CLAUDE.md`。

沿用已有命名和写法，例如：

```text
docs/<project>/tasks/01_prepare_dataset.md
docs/<project>/tasks/02_build_evaluator.md
docs/<project>/tasks/03_run_policy_eval.md
```

如果项目目录还不存在，按如下结构建议新建：

```text
docs/<project>/
  README.md
  运行命令.md
  实验结果.md
  tasks/
    01_<task_name>.md
```

并提醒用户可能还要补：

```text
docs/<project>/README.md
docs/<project>/运行命令.md
docs/<project>/实验结果.md
docs/<project>/execution_contract.md
```

### 2. 判断任务轻重

先判断任务类型：

| 类型     | 例子                                                       | 要求                                                        |
| ------ | -------------------------------------------------------- | --------------------------------------------------------- |
| 重型代码任务 | evaluator、dataset generator、sim pipeline、training script | 必须完整填写 verifier、negative test、evidence bundle、AC/CP/VF 映射 |
| 实验任务   | policy eval、ablation、benchmark run                       | 必须填写指标、统计验证、run_config、artifact、evidence                  |
| 数据任务   | dataset conversion、split、filtering                       | 必须填写 schema、split 泄漏检查、hash、dropped sample report         |
| 论文写作任务 | revision plan、回应 reviewer、重写 introduction                | 可精简 verifier，但仍要有 AC-ID 和可检查证据                            |
| 轻量分析任务 | 文献对比、表格整理                                                | 可缩放模板，但不得失去目标、交付物、验收标准、证据入口                               |

### 3. 先写 AC，再写 verifier，再写实现计划

推荐顺序：

```text
目标
→ Non-goals
→ 数据契约 / 指标定义
→ AC-ID 验收标准
→ Verifier 设计
→ Negative tests
→ Checkpoint 切片
→ 实现计划
→ 交付物
→ 复现入口
→ Evidence bundle
```

不要先写实现计划再临时补验收标准。

### 4. 信息不全就写 TBD，不要编造

还没定的代码路径、命令、指标、artifact 路径，写：

```text
TBD：待确认
TBD：待实现后补充
候选：<path>，尚未确认
```

不要编造：

```text
src/evaluator/safety_eval.py
runs/latest/metrics.json
```

除非用户提供了文件树，或你确实看到了仓库路径。

### 5. 写完自检

写完后必须对照文末“完成前自检清单”。

---

## 标准模板：章节清单

完整 task 文件建议包含以下章节。重型任务尽量填满；轻量任务按后文规则缩放。

| 章节                           | 必填?              | 职责                                                    |
| ---------------------------- | ---------------- | ----------------------------------------------------- |
| `# 任务 <编号>：<标题>`             | 必填               | 一眼看出是第几个任务、做什么                                        |
| `## 状态`                      | 必填               | 规划中/进行中/阻塞/已完成/已废弃 + 最近更新日期 + 一句话进展                   |
| `## 目标`                      | 必填               | 任务要完成什么、边界在哪里                                         |
| `## 背景`                      | 可缩放              | 为什么需要，依赖哪个上游，服务哪个下游                                   |
| `## 技术路线`                    | 必填               | 方法、输入、输出、关键设计取舍                                       |
| `## 数据契约与不变量`                | 代码/实验/数据任务必填     | schema、字段、指标定义、非法输入处理、split/leakage/invariant         |
| `## 实现计划`                    | 必填               | 要改/新增哪些文件，代码路径在哪里                                     |
| `## 指标与统计验证`                 | 实验/评测任务必填        | 指标定义、统计验证、seed、CI、tolerance、flaky 规则                  |
| `## 交付物`                     | 必填               | artifact 表、路径、状态、生成命令、验证命令                            |
| `## 复现入口`                    | 必填               | 指向 `运行命令.md` / `runs/.../run_config.json` / `实验结果.md` |
| `## 验收标准`                    | 必填               | AC-ID 列表，每条可客观判断                                      |
| `## Verifier 设计`             | 代码/实验/数据任务必填     | VF-ID、覆盖 AC-ID、命令、期望结果                                |
| `## Verifier 反例测试`           | 关键 verifier 必填   | NEG-ID、故意注入什么错误、如何证明 verifier 会失败                     |
| `## AC / CP / Verifier 覆盖矩阵` | 必填               | AC、checkpoint、verifier、negative test、evidence 的映射     |
| `## Checkpoint 切片`           | 必填               | 2–5 个纵切片，每片引用 AC-ID 和报到点                              |
| `## 阻塞与边界`                   | 必填               | 已有底座 / 本次落地 / 仍未做 / Non-goals / 越界处置                  |
| `## 执行约束`                    | LLM/agent 执行任务必填 | 禁止行为、派活时必须注入的 execution contract                      |
| `## 完成证据包`                   | 已完成时必填           | evidence bundle 摘要，指向 runs/ 原始日志                      |
| `## 开放问题`                    | 必填               | 待决策问题，初始可空                                            |
| `## 备注`                      | 可缩放              | 历史记录、设计取舍、偏离计划                                        |

---

## 用户四要素落在哪里

这个 skill 存在的根本理由，是保证每份 task 都能回答四个问题。

### 1. 技术路线

落在：

```text
## 技术路线
```

必须写清：

* 方法；
* 输入；
* 输出；
* 关键依赖；
* 替代方案；
* 为什么这样做；
* 不这样做的原因。

### 2. 交付物

落在：

```text
## 交付物
```

建议表格：

| 交付物 | 路径 | 对应 AC | 状态 | 生成命令 | 验证命令 | 说明 |
| --- | -- | ----- | -- | ---- | ---- | -- |

### 3. 实现的代码位置

落在：

```text
## 实现计划
```

建议表格：

| 模块/文件 | 作用 | 对应 AC | 状态 | 说明 |
| ----- | -- | ----- | -- | -- |

路径必须真实存在，或明确标 TBD。

### 4. 可运行/复现的 shell 指令

分两个层次：

#### task 文件里的代表性命令

落在：

```text
## 交付物
## Verifier 设计
## 验收标准
```

作用是让读者知道如何生成/核对 artifact。

#### runs/ 里的精确复现命令

落在：

```text
runs/.../run_config.json
runs/.../verification/
```

记录某一次真实运行的完整命令、环境、git、数据、日志、artifact。

---

## 数据契约与不变量

代码、实验、数据、评测任务必须写本节。它用于固定输入、输出、指标和语义定义，防止实现过程中被默认脑补或悄悄改变。

### 输入契约

必须写清：

* 输入文件路径；
* 文件格式；
* required fields；
* optional fields；
* 字段类型；
* 单位；
* 坐标系；
* 时间步定义；
* ID 规则；
* split 规则；
* 缺失字段处理；
* 非法样本处理；
* 是否允许跳过样本；
* dropped sample 是否必须报告。

模板：

```text
### 输入契约

- 输入文件：
- 格式：
- 必填字段：
- 可选字段：
- 字段类型：
- 单位 / 坐标系 / 时间步：
- ID 规则：
- split 规则：
- 缺失字段处理：
- 非法样本处理：
- 是否允许跳过样本：
- dropped sample report：
```

### 输出契约

必须写清：

* 输出文件路径；
* 输出格式；
* required fields；
* 是否必须保留输入 ID；
* 是否必须保留输入顺序；
* 是否允许新增字段；
* 是否允许缺失样本；
* 是否必须输出 dropped-count；
* 是否必须输出 per-sample report。

模板：

```text
### 输出契约

- 输出文件：
- 格式：
- 必填字段：
- 是否保留输入 ID：
- 是否保留输入顺序：
- 是否允许新增字段：
- 是否允许跳过样本：
- per-sample report：
- aggregate report：
```

### 关键定义

所有核心概念都要写成可判定定义，不要写模糊描述。

例如：

```text
- `success`：
- `safety_violation`：
- `unsafe_success`：
- `episode_failure`：
- `rule_violation_count`：
- `valid_episode`：
- `dropped_episode`：
```

指标定义必须明确公式。例如：

```text
unsafe_success = success == true and has_safety_violation == true
unsafe_success_rate = unsafe_success_count / total_episode_count
```

### 手算 golden 数值例子

关键 metric 不能只有文字定义，必须有至少 1–3 个手算例子。

示例：

```text
Golden case G-1：

输入：
- episode_001: success=true, safety_violation=true
- episode_002: success=true, safety_violation=false
- episode_003: success=false, safety_violation=true

手算：
- total_episode_count = 3
- success_count = 2
- violation_count = 2
- unsafe_success_count = 1
- unsafe_success_rate = 1 / 3

期望输出：
- unsafe_success_count 必须等于 1
- unsafe_success_rate 必须等于 0.333333，允许误差 1e-6
```

### 通用不变量

常见不变量包括：

```text
- 输入中的每个 sample_id / episode_id 必须在输出中有对应记录，除非明确记录 dropped reason。
- 总数类指标必须满足：0 <= unsafe_success_count <= success_count <= total_count。
- violation_count 不得小于 unsafe_success_count。
- 输出中的 sample 数必须等于输入有效 sample 数。
- 同一个输入和配置下，确定性 verifier 的输出必须稳定。
- 缺失关键字段必须显式报错，不能 silent skip。
- 所有过滤逻辑必须输出 dropped-count 和原因。
```

### ML / Eval / Benchmark 专用不变量

ML 和 benchmark 任务必须重点检查以下问题：

```text
- train / val / test split 必须互斥。
- 同一个 scene_id / task_id / trajectory_id / episode_id 不得跨 split 泄漏，除非任务明确允许。
- evaluation 不得读取 ground-truth answer、ground-truth plan、violation label 或未来状态。
- method 和 baseline 必须使用同一 test set、同一 evaluation protocol。
- 每个 split 的样本数、scene 分布、task 分布、rule 分布必须输出到 report。
- dataset version、dataset hash、rule config hash、prompt template hash 必须进入 run_config。
- 任何样本过滤、失败跳过、timeout skip 都必须输出数量和原因。
- 不得用 test set 调参。
- 不得在 eval 阶段动态修改 benchmark rule。
- 不得把 failed episodes 从 denominator 中静默移除。
```

### Robotics / Simulation 专用不变量

Robotics / simulation 任务建议检查：

```text
- scene config 与 episode metadata 一致。
- object id、pose、scale、unit、coordinate frame 明确。
- robot joint limit / velocity limit / torque limit 不得被绕过。
- collision / contact / safety rule 的判定时刻必须明确。
- simulation seed、physics setting、controller config 必须进入 run_config。
- evaluation 不得读取 simulator hidden privileged state，除非任务明确允许。
- replay 与 online eval 的指标定义必须一致。
```

---

## 指标与统计验证

实验和评测任务必须写本节。

### 指标定义

每个指标必须写：

| 指标 ID | 名称 | 定义 | 分母 | 分子 | 单位 | 越大越好? | 对应 AC |
| ----- | -- | -- | -- | -- | -- | ----- | ----- |

不要只写：

```text
计算 success rate。
```

要写：

```text
M-1: success_rate = success_count / total_valid_episode_count。
分母包含 timeout 和 execution failure，不允许静默移除失败 episode。
```

### 确定性验证

适用于 metric function、schema、parser、golden cases。

必须写：

* 输入；
* expected output；
* tolerance；
* 验证命令；
* 对应 AC-ID。

### 统计型验证

适用于 RL、sampling、LLM eval、stochastic simulation。

必须写：

```text
- seed 列表：
- 每个 seed 的 episode 数：
- 总 episode 数：
- baseline：
- 统计量：
- confidence interval：
- tolerance：
- pass/fail 判据：
- flaky 处理：
- 是否允许重跑：
- 重跑如何记录：
```

示例：

```text
统计验证规则：

- seeds: [0, 1, 2, 3, 4]
- 每个 seed 100 episodes
- 报告 mean ± 95% bootstrap CI
- method 的 unsafe_success_rate 不得高于 baseline 超过 3%
- 如果任一 seed 的 dropped_episode_count > 0，必须失败
- 不允许只挑表现最好的 seed 报告
```

---

## 验收标准

验收标准必须使用 AC-ID。

示例格式：

```text
- AC-1：evaluator 必须读取 `episodes.jsonl` 和 `safety_rules.yaml`，并输出符合 schema 的 `metrics.json`。
  - 验证：VF-1
  - 反例：NEG-1
  - 证据：完成后指向 runs/.../verification/

- AC-2：`unsafe_success` 必须严格定义为 `success == true and has_safety_violation == true`。
  - 验证：VF-2
  - 反例：NEG-2
  - 证据：完成后指向 runs/.../verification/
```

要求：

* 每条 AC 必须唯一编号；
* 每条 AC 必须可客观判断；
* 每条 AC 至少有一个 verifier；
* 每条 AC 必须能映射到 checkpoint；
* 重型任务的关键 AC 必须有 negative test；
* 没有 verifier 的 AC 必须解释为什么只能人工审查；
* 没有 evidence 的 AC 不能算完成。

避免写：

```text
- AC：代码工作正常。
- AC：结果合理。
- AC：图看起来不错。
```

应该写：

```text
- AC-3：`rule_breakdown.csv` 必须包含每条 rule 的 violation count，且所有 rule 的 violation count 之和等于 per-episode report 中 violation event 的总数。
```

---

## Verifier 设计

代码、实验、数据、评测任务必须写本节。

### Verifier 表格

| Verifier ID | 覆盖 AC | 类型        | 命令                                                                   | 期望结果   | 日志路径                             | 说明            |
| ----------- | ----- | --------- | -------------------------------------------------------------------- | ------ | -------------------------------- | ------------- |
| VF-1        | AC-1  | schema    | `python scripts/validate_schema.py ...`                              | exit 0 | `runs/.../verification/VF-1.log` | 检查输出 schema   |
| VF-2        | AC-2  | golden    | `python -m pytest tests/test_metrics.py::test_unsafe_success_golden` | exit 0 | `runs/.../verification/VF-2.log` | 检查手算 golden   |
| VF-3        | AC-3  | invariant | `python scripts/check_invariants.py ...`                             | exit 0 | `runs/.../verification/VF-3.log` | 检查 count 和 ID |

### Verifier 类型

常见 verifier 类型：

```text
schema
unit
golden
invariant
property
leakage
statistical
smoke
end-to-end
regression
manual-review
```

### 总验证命令

重型任务必须提供一个总验证入口，例如：

```bash
python scripts/run_full_verifier.py --task <task_id>
```

或者：

```bash
make verify TASK=<task_id>
```

要求：

* 一条命令能跑完所有关键 verifier；
* 失败时 exit code 非 0；
* 输出每个 AC 的覆盖状态；
* 保存日志到 `runs/.../verification/`；
* 最终 evidence manifest 可复查。

---

## Verifier 反例测试

这是硬要求：**每个关键 verifier 必须有反例测试**。

### 反例测试表格

| Negative ID | 对应 Verifier | 覆盖 AC | 注入错误                             | 期望 verifier 行为          | 命令                                                  | 说明               |
| ----------- | ----------- | ----- | -------------------------------- | ----------------------- | --------------------------------------------------- | ---------------- |
| NEG-1       | VF-1        | AC-1  | 删除 required field                | schema verifier 返回非零    | `python -m pytest tests/test_schema_negative.py`    | 证明 schema 检查不是摆设 |
| NEG-2       | VF-2        | AC-2  | 改错 expected unsafe_success_count | golden verifier 返回非零    | `python -m pytest tests/test_metric_negative.py`    | 证明 metric 错误会被抓  |
| NEG-3       | VF-3        | AC-3  | 删除一个 episode_id                  | invariant verifier 返回非零 | `python -m pytest tests/test_invariant_negative.py` | 证明 ID 丢失会被抓      |

### 反例测试写法说明

反例测试不是让整个 CI 失败，而是让测试断言“verifier 对坏输入会失败”。

例如：

```python
def test_schema_verifier_rejects_missing_required_field(tmp_path):
    bad_output = tmp_path / "metrics_bad.json"
    bad_output.write_text('{"success_rate": 0.5}')

    result = subprocess.run(
        ["python", "scripts/validate_metrics_schema.py", str(bad_output)],
        capture_output=True,
        text=True,
    )

    assert result.returncode != 0
    assert "missing" in result.stderr.lower() or "required" in result.stderr.lower()
```

这个 pytest 本身应该 exit 0，因为它成功证明 verifier 能抓错。

### 不能接受的反例测试

不要写：

```python
def test_negative():
    assert True
```

不要只写：

```text
人工确认 verifier 能失败。
```

不要把 corrupted output 放进正常 artifact 目录伪装成结果。

---

## AC / CP / Verifier 覆盖矩阵

每份 task 文件必须有覆盖矩阵。

模板：

| AC-ID | 验收内容摘要                | Checkpoint | Verifier | Negative Test | Evidence | 状态  |
| ----- | --------------------- | ---------- | -------- | ------------- | -------- | --- |
| AC-1  | schema 正确             | CP-1       | VF-1     | NEG-1         | TBD      | 未完成 |
| AC-2  | metric 定义正确           | CP-2       | VF-2     | NEG-2         | TBD      | 未完成 |
| AC-3  | 不变量成立                 | CP-2       | VF-3     | NEG-3         | TBD      | 未完成 |
| AC-4  | end-to-end smoke test | CP-3       | VF-4     | NEG-4         | TBD      | 未完成 |

硬约束：

```text
- 每条 AC 至少一个 verifier。
- 每个 checkpoint 至少引用一个 AC。
- 每个关键 verifier 至少一个 negative test。
- 任务完成时，每条 AC 的 Evidence 必须指向 runs/ 中的真实证据。
```

如果某条 AC 只能人工审查，必须写成：

```text
Verifier: MANUAL-1
审查对象：paper/intro.tex 第 2–4 段
审查标准：是否明确回应 Reviewer 2 关于 contradiction 的评论
证据：diff 链接 / reviewer response 摘要 / 人工确认记录
```

---

## Checkpoint 切片

Checkpoint 用来防漂移。每个 checkpoint 必须引用 AC-ID。

建议 2–5 个纵切片，每个切片都能产生可检查的中间结果。

模板：

| Checkpoint ID | 覆盖 AC      | 目标                       | 产物                         | 报到点                              | Verifier   | 状态  |
| ------------- | ---------- | ------------------------ | -------------------------- | -------------------------------- | ---------- | --- |
| CP-1          | AC-1       | 完成 schema 与 parser       | parser + schema test       | 提交 schema 验证日志                   | VF-1       | 未开始 |
| CP-2          | AC-2, AC-3 | 完成 metric 与 invariant    | metrics + invariant report | 提交 golden/invariant 日志           | VF-2, VF-3 | 未开始 |
| CP-3          | AC-4       | 跑通 end-to-end smoke test | smoke run artifact         | 提交 run_config 和 verification log | VF-4       | 未开始 |

Checkpoint 不是横向分工：

```text
先写全部代码；
再写全部测试；
最后写全部文档。
```

而应该是纵切片：

```text
一个小输入 → 一段核心实现 → 一个输出 → 一个 verifier → 一个 evidence。
```

---

## 阻塞与边界

本节用于防止 agent 理解偏差和过度工程。

必须包含：

```text
### 已有底座

- 已存在的代码：
- 已存在的数据：
- 已存在的测试：
- 已存在的运行脚本：

### 本次落地

- 本任务要完成：
- 本任务要修改：
- 本任务要新增：

### 本任务不做（Non-goals）

- 不做：
- 不改：
- 不评估：
- 不重构：

### 越界处置

如果执行中发现需要做 non-goal 里的事情，必须：
1. 停止扩大范围；
2. 写入 `## 开放问题`；
3. 向用户/维护者确认；
4. 不得自行把越界工作混进本任务。
```

---

## 执行约束

对 LLM/agent vibe coding 任务，本节必填。

### 派活时必须注入的执行契约

派活 prompt 必须附上或引用：

```text
- 当前 task 文件；
- execution_contract.md；
- 当前要执行的 CP-ID；
- 当前覆盖的 AC-ID；
- 必须运行的 VF-ID；
- 必须运行的 NEG-ID；
- 禁止行为；
- evidence bundle 格式。
```

推荐固定文件：

```text
docs/<project>/execution_contract.md
~/.claude/harness/execution_contract.md
AGENTS.md
CLAUDE.md
```

### 禁止行为

执行者不得：

```text
- 为了通过验收而修改测试、golden 文件或 expected 输出，除非任务明确要求更新基准。
- 硬编码 golden case 的答案。
- silent skip 无效样本、缺失字段、失败 episode 或 timeout episode。
- 未经说明修改 public API、文件命名、目录结构或已有输出格式。
- 把 unrelated refactor 混进本任务。
- 用 mock / fake output 冒充真实实验结果。
- 在未实际运行命令时声称“已验证通过”。
- 只运行部分 verifier 就声称全绿。
- 失败后只贴最终成功日志而不记录失败修复过程，除非 task 明确不要求。
- 私自扩大范围解决开放问题。
- 删除历史记录或旧计划来掩盖偏离。
```

### 最终报告要求

执行 agent 完成任务后必须报告：

```text
- 完成的 CP-ID：
- 覆盖的 AC-ID：
- 修改文件：
- 新增文件：
- 未修改但相关的文件：
- 运行的 verifier：
- 运行的 negative tests：
- 总验证命令：
- exit code：
- evidence bundle 路径：
- artifact 路径：
- git commit：
- git dirty：
- 未解决问题：
- 偏离原计划之处：
```

---

## 完成证据包

任务标为 `已完成` 时必须有本节。

### 证据包原则

task 文件只写摘要，不粘贴巨大日志。完整日志放在 `runs/.../verification/`。

### 推荐目录结构

```text
runs/<date>_<task_name>/
  run_config.json
  artifacts/
    metrics.json
    per_episode_report.jsonl
    rule_breakdown.csv
  verification/
    manifest.json
    full_verifier.stdout.log
    full_verifier.stderr.log
    VF-1_schema.log
    VF-2_golden.log
    VF-3_invariant.log
    NEG-1_schema_negative.log
    NEG-2_metric_negative.log
    coverage_matrix.json
```

### evidence manifest 建议字段

```json
{
  "task_id": "03",
  "task_file": "docs/<project>/tasks/03_<task>.md",
  "status": "passed",
  "git_commit": "<commit>",
  "git_dirty": false,
  "run_config": "runs/<date>_<task>/run_config.json",
  "full_verifier_command": "python scripts/run_full_verifier.py --task 03",
  "exit_code": 0,
  "stdout_log": "runs/<date>_<task>/verification/full_verifier.stdout.log",
  "stderr_log": "runs/<date>_<task>/verification/full_verifier.stderr.log",
  "artifacts": [
    "runs/<date>_<task>/artifacts/metrics.json"
  ],
  "coverage": {
    "AC-1": ["VF-1", "NEG-1"],
    "AC-2": ["VF-2", "NEG-2"]
  }
}
```

### task 文件中的完成证据摘要

模板：

```text
## 完成证据包

状态：已完成

最终验证：
- 总验证命令：`python scripts/run_full_verifier.py --task 03`
- exit code：0
- evidence manifest：`runs/2026-xx-xx_task03/verification/manifest.json`
- stdout：`runs/2026-xx-xx_task03/verification/full_verifier.stdout.log`
- stderr：`runs/2026-xx-xx_task03/verification/full_verifier.stderr.log`
- artifact：`runs/2026-xx-xx_task03/artifacts/`
- git commit：见 `run_config.json`
- git dirty：见 `run_config.json`

覆盖情况：
- AC-1：VF-1 + NEG-1，通过
- AC-2：VF-2 + NEG-2，通过
- AC-3：VF-3 + NEG-3，通过

备注：
- 本节只保留证据入口；完整日志以 runs/ 为准。
```

### 标为已完成的硬条件

只有同时满足以下条件，才能把状态改为 `已完成`：

```text
- 所有必需 AC 均有 verifier 覆盖。
- 所有关键 verifier 均有 negative / mutation test。
- 总验证命令在干净环境一次性全绿。
- exit code = 0。
- stdout/stderr 已保存。
- artifact 路径存在。
- run_config 存在。
- git commit / git dirty 状态已记录。
- coverage matrix 已更新。
- 没有未说明的 skipped test。
```

---

## 交付物

交付物必须写成表格。

模板：

| 交付物                | 路径                                            | 对应 AC      | 状态  | 产出命令         | 验证命令                                     | 说明            |
| ------------------ | --------------------------------------------- | ---------- | --- | ------------ | ---------------------------------------- | ------------- |
| evaluator 脚本       | `TBD`                                         | AC-1, AC-2 | 未完成 | `TBD`        | `TBD`                                    | 负责计算 metrics  |
| metrics 输出         | `runs/.../artifacts/metrics.json`             | AC-1       | 未完成 | `python ...` | `python scripts/validate_schema.py ...`  | aggregate 指标  |
| per-episode report | `runs/.../artifacts/per_episode_report.jsonl` | AC-3       | 未完成 | `python ...` | `python scripts/check_invariants.py ...` | 每条 episode 记录 |

要求：

* 路径必须真实或标 TBD；
* 产出命令必须可直接粘贴运行，或标 TBD；
* 验证命令必须对应 VF-ID；
* artifact 不能只写“生成结果”；
* 已完成时必须指向 runs/ 中的实际 artifact。

---

## 复现入口

本节指向复现单一真相，不重复抄录 run 细节。

模板：

```text
## 复现入口

代表性运行入口：
- `docs/<project>/运行命令.md`：说明常用运行方式。
- `runs/<date>_<task>/run_config.json`：本次实际运行的完整命令、环境、git、数据。
- `runs/<date>_<task>/verification/manifest.json`：本次验证证据。
- `docs/<project>/实验结果.md`：汇总主要实验结果和图表入口。

注意：
- task 文件不抄录完整 run command、git commit、env、data hash。
- 这些信息以 `runs/.../run_config.json` 为准。
```

写作类任务没有 runs/ 时，可改为：

```text
## 复现入口 / 核对入口

- 修改文件：
- diff 查看命令：
- reviewer comment 对应位置：
- 验收 grep / diff 命令：
- 人工审查记录：
```

---

## 更新已有 task 文件时的规则

如果是在更新已有 task 文件：

```text
- 优先保持原有章节结构、编号和语气。
- 不要删除历史决策、失败尝试和已完成记录。
- 过时内容标「（历史记录）」。
- 新增内容尽量放在“最近更新 / 当前计划 / 偏离说明 / 新验收标准”中。
- 如果新计划推翻旧计划，必须写明：旧计划是什么、新计划是什么、为什么改变。
- 不要把未确认的完成状态改成“已完成”。
- 如果新增 AC，必须同步更新 CP、VF、NEG 和覆盖矩阵。
- 如果修改指标定义，必须同步更新 golden case 和 evidence 说明。
- 如果更新 verifier，必须同步更新 verifier negative test。
```

---

## 按任务轻重缩放

任务有轻有重。重型代码/实验任务章节填满；轻量写作/分析任务可以精简，但不能失去可验收性。

### 重型代码 / 实验 / 数据任务

必须保留：

```text
状态
目标
背景
技术路线
数据契约与不变量
实现计划
指标与统计验证
交付物
复现入口
验收标准
Verifier 设计
Verifier 反例测试
AC / CP / Verifier 覆盖矩阵
Checkpoint 切片
阻塞与边界
执行约束
完成证据包
开放问题
备注
```

### 轻量写作 / 论文修改任务

可以精简：

```text
技术路线 → 写成“修改思路 / 参考依据 / 产出形态”
实现计划 → 写成“要改哪些文件或章节”
复现入口 → 改成“核对入口”
指标与统计验证 → 可省略
数据契约与不变量 → 可省略或改成“论点契约”
Verifier 反例测试 → 可用反例审查或人工审查替代
```

但仍必须保留：

```text
状态
目标
交付物
验收标准 AC-ID
Checkpoint 或一次到位说明
阻塞与边界 / Non-goals
核对入口
完成证据
```

写作类验收标准示例：

```text
AC-1：Introduction 必须显式解释 classical planner under safety constraints 的失败模式。
Verifier：`rg -n "safety constraints|unsafe-success|constraint" paper/intro.tex`
人工审查：检查该段是否直接回应 Reviewer 2 的 contradiction comment。
Evidence：diff 链接或修改后的文件路径。
```

### 轻量任务的 negative test

轻量写作任务不一定需要程序化 mutation test，但必须写清楚：

```text
- 哪种输出会被判为失败；
- 如何发现这种失败；
- 谁来审查；
- 审查证据在哪里。
```

例如：

```text
失败反例：只删除 reviewer 提到的句子，但没有解释 contradiction。
审查方式：对照 Reviewer 2 原文和修改后的 paragraph。
证据：diff + 人工审查记录。
```

---

## 常见反例与修正

### 反例 1：只有命令，没有期望输出

差：

```text
验证：python script.py
```

好：

```text
VF-1：运行 `python scripts/validate_metrics_schema.py runs/.../metrics.json`。
期望：exit code = 0；stdout 包含 `schema validation passed`；stderr 为空。
覆盖：AC-1。
反例：NEG-1 删除 required field 后，该命令必须 exit non-zero。
```

### 反例 2：golden case 没有手算值

差：

```text
用 golden case 验证 metric。
```

好：

```text
Golden G-1 手算：
3 个 episode，其中 success=2，violation=2，unsafe_success=1。
期望 unsafe_success_rate = 1/3。
VF-2 必须检查输出数值在 1e-6 tolerance 内。
```

### 反例 3：任务完成但没有证据

差：

```text
状态：已完成。测试已通过。
```

好：

```text
状态：已完成。
证据：`runs/2026-xx-xx_task03/verification/manifest.json`
总验证命令：`python scripts/run_full_verifier.py --task 03`
exit code：0
stdout/stderr：见 runs/.../verification/
覆盖：AC-1~AC-5 全部通过。
```

### 反例 4：AC 和 verifier 脱节

差：

```text
验收标准：
- 计算 metric
- 输出 report
- 支持 visualization

验证：
- pytest
```

好：

```text
AC-1 → VF-1, NEG-1
AC-2 → VF-2, NEG-2
AC-3 → VF-3, NEG-3
覆盖矩阵明确列出每条 AC 的验证证据。
```

### 反例 5：随机任务要求 hash 完全一致

差：

```text
固定 seed 后输出必须完全一致。
```

好：

```text
metric function 的 golden case 要求精确一致；
policy eval 采用统计验证：5 seeds × 100 episodes，报告 mean ± 95% CI。
```

---

## 完成前自检清单

交付 task 文件前，必须逐条检查。

### 基础质量

* [ ] 全中文：标题和正文都是中文，除公式/标识符/路径/命令/引用原文。
* [ ] 任务目标清楚：读者能复述任务要做什么。
* [ ] Non-goals 清楚：知道本任务不做什么。
* [ ] 状态准确：`状态` 和 `最近更新` 反映真实进展。
* [ ] 历史保留：过时内容标「（历史记录）」而不是直接删除。

### 四要素

* [ ] 技术路线清楚。
* [ ] 交付物清楚。
* [ ] 代码位置清楚。
* [ ] 复现/验证命令清楚。

### 路径与命令

* [ ] 路径真实，或明确标 `TBD` / `候选路径`。
* [ ] 命令可直接粘贴运行，或明确标 `TBD`。
* [ ] 没有编造不存在的文件。
* [ ] 没有把 runs/ 中易过期细节抄进 task 文件。

### 数据契约

* [ ] 输入契约清楚。
* [ ] 输出契约清楚。
* [ ] 关键字段定义清楚。
* [ ] 指标公式清楚。
* [ ] 非法输入处理清楚。
* [ ] dropped sample 处理清楚。
* [ ] ML/eval 任务检查了 split 泄漏和数据分布。
* [ ] Robotics/simulation 任务检查了 scene、pose、unit、safety rule、sim config。

### 验收与 verifier

* [ ] 每条 AC 有唯一编号。
* [ ] 每条 AC 可客观判断。
* [ ] 每条 AC 至少有一个 verifier。
* [ ] 每个 verifier 映射到 AC-ID。
* [ ] 每个 checkpoint 引用 AC-ID。
* [ ] 覆盖矩阵完整。
* [ ] 关键 metric 有手算 golden 数值例子。
* [ ] 关键 verifier 有 negative / mutation test。
* [ ] 反例测试证明 verifier 会抓错。
* [ ] 总验证命令存在。
* [ ] 统计型任务区分 deterministic check 和 statistical check。

### 执行约束

* [ ] 写明禁止行为。
* [ ] 写明派活时必须注入 execution contract。
* [ ] 写明最终报告格式。
* [ ] 防止 agent 修改测试来通过验收。
* [ ] 防止 silent skip。
* [ ] 防止 unrelated refactor。
* [ ] 防止伪造 artifact 或未运行就声称通过。

### 完成证据

* [ ] `已完成` 状态有 evidence bundle。
* [ ] evidence manifest 指向 runs/。
* [ ] stdout/stderr 保存路径存在。
* [ ] exit code 记录。
* [ ] artifact 路径记录。
* [ ] git commit / git dirty 记录。
* [ ] AC coverage 记录。
* [ ] 最终一次干净环境全绿。
* [ ] 没有未说明的 skipped tests。

---

## 参考资料

* `assets/task_template.md`：直接复制去填的全中文模板。
* `references/section_guide.md`：每一节“好 vs 差”对照、常见反例、怎么写到位。
* `references/examples.md`：完整范例，包括重型代码/仿真任务、轻量论文修改任务。
* `docs/<project>/execution_contract.md`：项目级 agent 执行契约。
* `runs/.../run_config.json`：某次实际运行的复现单一真相。
* `runs/.../verification/manifest.json`：某次验证的证据入口。
