# 任务 <编号>：<简短标题>

<!--
使用说明（填完删掉本段注释）：
- 动笔前必须先和用户敲定「任务目标」与「完成口径（怎样算完成）」，由用户拍板；不要自己假设验收标准。
- 全中文，含章节标题。公式 / 代码标识符 / 路径 / 命令 / 配置字段 / 引用原文可保留原文。
- 默认只填【精简骨架】。重型代码 / 实验 / 数据任务再展开【重型补充】；轻量任务可从重型补充里按需挑用得上的节。
- 编号：AC-N 验收，CP-N checkpoint，VF-N verifier，NEG-N 反例，M-N 指标，G-N 手算 golden，MANUAL-N 人工审查。
- 信息未定就写 `TBD：待确认` / `候选：<path>，尚未确认`，不要编造路径或命令。
- 派活给 agent 时，随 prompt 注入本文件 + execution_contract + 当前 AC/CP/VF/NEG，别只说一句「完成 task 03」。
-->

<!-- ===================== 精简骨架（默认填这些） ===================== -->

## 状态

状态：规划中 | 进行中 | 阻塞 | 已完成 | 已废弃
最近更新：YYYY-MM-DD
一句话进展：<现在做到哪、下一步是什么、证据在哪>

## 目标

<这个子任务要达成什么、边界在哪。一两句话让人看懂。这里写的是和用户确认过的目标。>

## 完成口径（验收）

<怎样算完成——由用户定义，细化成可客观判定的 AC。每条绑验证方式与证据。>

- [ ] AC-1：<可客观判定的条件，不写「工作正常」这种话>
  - 验证：<命令 / `rg ...` / VF-1 / MANUAL-1>
  - 证据：<完成后指向 runs/.../verification/，或 diff / 人工审查记录>
- [ ] AC-2：……

## 交付物

| 交付物 | 路径 | 对应 AC | 状态 | 说明 |
|---|---|---|---|---|
| <名称> | `TBD` | AC-1 | 未完成 | |

（路径必须真实或标 TBD；已完成时指向 runs/ 真实 artifact。）

## 阻塞与边界

- 已有底座：<已存在的代码 / 数据 / 测试 / 运行脚本>
- 本次落地：<这次要做 / 要改 / 要新增的>
- 仍未做：<留给后续任务的>
- 本任务不做（Non-goals）：<不做 / 不改 / 不评估 / 不重构什么>
- 越界处置：发现 Non-goal 里的事 → 记入「开放问题」，停手并向用户确认，不擅自做。

## 复现 / 核对入口

- 代表性运行：见 `../运行命令.md`
- 完整命令 / 环境 / 数据 / git：见 `runs/<id>/run_config.json`（不抄进本文件）
- 验证证据：见 `runs/<id>/verification/manifest.json`

（写作类无 runs/ 时改为：要改的文件 + diff 命令 + reviewer comment 对应位置 + 人工审查记录。）

## 完成证据

<状态 = 已完成时填；只写摘要，完整日志放 runs/.../verification/>

- 总验证命令：`...`　exit code：0
- manifest / stdout / stderr / artifact 路径：……
- git commit / dirty：见 `run_config.json`
- 覆盖：AC-1 通过(VF-1 + NEG-1)；AC-2 通过(VF-2 + NEG-2)……

## 开放问题

- （初始为空）

<!-- ===================== 重型补充（重型代码 / 实验 / 数据任务再展开；轻量任务删掉下面整块） ===================== -->

## 背景

<为什么需要这个任务；依赖哪个上游、服务哪个下游 / 论文 / 系统。>

## 技术路线

<方法 + 输入 + 输出 + 关键设计取舍 + 被否的替代方案。框架名不是技术路线——要写怎么用、为什么这么选。>
<轻量写作任务可改写成「修改思路 / 依据 / 产出形态」。>

## 数据契约与不变量

- 输入契约：文件、格式、必填 / 可选字段、类型、单位、ID 规则、split 规则、缺失 / 非法样本处理、是否允许 skip、dropped report。
- 输出契约：文件、必填字段、是否保留输入 ID / 顺序、是否允许新增字段、per-sample / aggregate report。
- 关键定义（写成可判定公式）：`unsafe_success = success==true and has_violation==true`……
- 手算 golden（G-1）：最小输入 → 逐步手算 → 期望数值 + tolerance。
- 不变量：count 守恒、ID 守恒、缺关键字段必报错不 silent skip；ML/eval 加 split 互斥 / no-leakage / 不用 test 调参 / 不静默移除失败 episode；robotics/sim 加 pose·unit·frame / limit 不被绕过 / 不读 privileged state。

## 实现计划

| 模块 / 文件 | 作用 | 对应 AC | 状态 |
|---|---|---|---|
| `src/...` | | AC-1 | |

（路径必须真实存在或标 TBD；编造路径比不写更糟。）

## 指标与统计验证

| 指标 ID | 定义 / 公式 | 分母 | 对应 AC |
|---|---|---|---|
| M-1 | `success_count / total_valid_count` | 含 timeout / failure，不静默移除 | AC-2 |

- 确定性指标（metric function / schema / golden）：精确比对 + tolerance。
- 统计型指标（RL / 采样 / LLM eval）：seed 列表、每 seed 样本数、CI / bootstrap、tolerance、pass/fail 判据、flaky 与重跑规则。

## Verifier 设计

| VF-ID | 覆盖 AC | 类型 | 命令 | 期望 | 日志 |
|---|---|---|---|---|---|
| VF-1 | AC-1 | schema | `...` | exit 0 | `verification/VF-1.log` |

总验证命令：`python scripts/run_full_verifier.py --task <id>`（失败 exit 非 0，打印每条 AC 覆盖状态）。

## Verifier 反例测试

| NEG-ID | 对应 VF | 覆盖 AC | 注入错误 | 期望 verifier 行为 | 命令 |
|---|---|---|---|---|---|
| NEG-1 | VF-1 | AC-1 | 删 required field | exit 非 0 | `...` |

反例测试本身应 exit 0（它断言 verifier 对坏输入会失败）。禁止 `assert True`。

## AC / CP / Verifier 覆盖矩阵

| AC-ID | 摘要 | Checkpoint | Verifier | Negative | Evidence | 状态 |
|---|---|---|---|---|---|---|
| AC-1 | | CP-1 | VF-1 | NEG-1 | TBD | 未完成 |

## Checkpoint 切片

| CP-ID | 覆盖 AC | 目标 | 产物 | 报到点 | Verifier | 状态 |
|---|---|---|---|---|---|---|
| CP-1 | AC-1 | | | 提交 VF-1 日志 | VF-1 | 未开始 |

（2–5 个纵切片：一个小输入 → 核心实现 → 输出 → 一个 verifier → 一个 evidence；不是「先写全部代码再写全部测试」的横向分工。）

## 执行约束

- 派活注入：本文件 + `execution_contract` + 当前 CP / AC / VF / NEG + evidence 格式 + 报告格式。
- 禁止：改测试 / golden 来过验收、硬编码答案、silent skip 缺字段 / 失败 episode、混 unrelated refactor、mock 冒充真实结果、未运行就称通过、只跑部分 verifier 就称全绿、无证据标已完成。

## 备注

<关联决策、设计取舍、偏离计划之处；过时内容标「（历史记录）」保留，不直接删。>
