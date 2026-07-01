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

为研究项目写**清晰、可验证、可复现**的 task 文件，统一放在 `docs/<project>/tasks/`。读者是几个月后的你、协作者，以及执行任务的 agent。

**默认产出精简的 task 文件**：能说清就别堆章节。只有重型代码 / 实验 / 数据任务才展开全套 verifier 机制（见「按轻重缩放」）。输出一律全中文（含章节标题）；公式、代码标识符、路径、命令、配置字段、引用原文可保留原文。

---

## 第一步（必做）：先和用户敲定「目标 + 完成口径」

动笔写 task 文件之前，**必须先和用户交互**，由用户拍板两件事：

1. **任务目标**：这个子任务要达成什么、边界在哪（做什么、不做什么）。
2. **完成口径（验收）**：怎样才算完成？用什么客观证据 / 检查来判定？

规则：

- 完成口径**由用户定义**，不要替用户假设验收标准。你的工作是把用户给的口径细化成带编号的 AC，再补上「怎么验、证据在哪」。
- 用户说不清时，给候选方案让他选，而不是自己拍板。
- 目标或完成口径没敲定，就不要开始写正文——继续问。

交互最少要问清：

```text
- 这个 task 要解决什么问题 / 产出什么？
- 达到什么状态算「做完了」？（一两条可判断的条件）
- 有什么明确不做的（Non-goals）？
- 是重型代码 / 实验任务，还是轻量写作 / 分析任务？（决定填多详）
```

---

## 核心理念（六条）

1. **目标与完成口径由用户定。** AC 必须可客观判定、有证据；不写「工作正常 / 效果不错 / 图看着行」这种没法判真假的话。
2. **docs 写规格，runs 存复现。** task 文件只留稳定规格 + 入口 + 摘要；某次运行的完整命令、`git_commit`、环境、数据 hash、原始 `stdout/stderr` 属于 `runs/.../run_config.json`，不抄进 task 文件。
3. **先验后做。** 重型任务先写 AC 和怎么验，再写实现——别先写实现再临时补验收。
4. **Verifier 不是摆设。** 关键 verifier 必须有反例测试（注入错误 → verifier 必须失败），证明它真能抓错。
5. **三道闸门防跑偏。** AC 逐条绑验证 / Checkpoint 纵切防漂移 / Non-goals + 越界处置防过度工程。
6. **不假完成。** 标「已完成」必须有证据闭环：总验证命令一次全绿 + exit 0 + 日志 / artifact 路径 + git 状态。没证据只能是「进行中」。

真实优先：路径、命令、指标只写真实存在或已确定的；不确定就写 `TBD：待确认`（说明缺什么），**绝不编造**仓库里不存在的路径。

---

## 按轻重缩放：默认精简，重型才展开

先和用户确认任务类型，再决定填多少。

| 任务类型 | 怎么填 |
|---|---|
| 轻量写作 / 分析 / 论文修改 | 只填**精简骨架**；验收用 grep + 人工审查即可，AC 仍要编号、可判定。按需从重型补充里借用「技术路线（改思路）/ 实现计划（改哪些文件）」。 |
| 重型代码 / 实验 / 数据 / 评测 | 精简骨架 + **重型补充**（数据契约、Verifier 设计、反例测试、覆盖矩阵、Checkpoint、执行约束等）。 |

### 精简骨架（默认，所有 task 都要）

```text
状态 / 目标 / 完成口径(AC) / 交付物 / 阻塞与边界(含 Non-goals) / 复现或核对入口 / 完成证据 / 开放问题
```

其中**完成口径**合并了「验收标准 + 怎么验」：每条 AC 写清判定条件 + 验证方式（命令 / grep / 人工审查都行）+ 证据在哪。

### 重型补充（重型任务再加）

```text
背景 / 技术路线 / 数据契约与不变量 / 实现计划 / 指标与统计验证 /
Verifier 设计 / Verifier 反例测试 / AC·CP·VF 覆盖矩阵 / Checkpoint 切片 / 执行约束 / 备注
```

> **重型实验 / 仿真任务的可视化（默认项）**：若任务产生可观测行为（policy rollout 视频、渲染帧序列、指标曲线图），默认在「交付物」与「完成证据」登记其**真实可视化路径**，别让 demo 散落在 runs/ 里无人指向。这是重型任务的默认项、不是强制 AC；用户对可视化另有口径时从用户，轻量写作 / 分析任务不要求。

每节怎么写见 `references/section_guide.md`；完整范例（重型 + 轻量）见 `references/examples.md`；可直接复制的模板见 `assets/task_template.md`。

---

## 编号与映射

编号约定（保持一致）：

```text
AC-N   验收标准         VF-N   verifier
CP-N   checkpoint       NEG-N  反例测试
M-N    指标             G-N    手算 golden
MANUAL-N   只能人工审查的验收
```

硬约束：

- 每条 AC 有唯一编号、可客观判定、至少被一种方式验证（VF 或 MANUAL）。
- 每条 AC 至少落到一个 Checkpoint。
- 每个关键 verifier 至少有一个反例测试。
- 没有证据的 AC 不算完成。

重型任务必须给**覆盖矩阵**，一行一条 AC：

```text
AC-ID | 摘要 | Checkpoint | Verifier | Negative Test | Evidence | 状态
```

只能人工审查的 AC 用 `MANUAL-N`，注明审查对象、审查标准、证据位置。

---

## 防假完成

- **Verifier-first（重型）**：先写 verifier 再写实现。验收别只查「文件在不在 / 脚本 exit 0」，要查 schema、核心 metric 算对、golden 精确匹配、ID 守恒、不变量、数据 split 无泄漏、seed / config 可追踪。
- **反例测试**：每个关键 verifier 配一个反例（删字段 / 改错 expected / 删 episode），证明 verifier 对坏输入会失败。反例测试本身在测试框架里「通过」（断言 verifier 返回非零 / 抛异常）。禁止 `assert True` 式空反例。
- **随机实验用统计验证**：RL / 采样 / LLM eval / stochastic sim 不要求 hash 一致；写 seed 列表、样本量、CI / tolerance、pass/fail 判据、flaky 与重跑规则。只有确定性部分（metric / schema / golden）才用精确比对。
- **完成硬条件**：状态改「已完成」前必须全部满足——所有必需 AC 有验证；关键 verifier 有反例；总验证命令在干净环境一次全绿（exit 0）；`stdout/stderr`、artifact、`run_config`、git 状态都已记录；覆盖矩阵已更新；没有未说明的 skipped test。证据摘要写进「完成证据」节，完整日志放 `runs/.../verification/`。

---

## 更新已有 task 文件的纪律

- 保持原有章节结构、编号、语气；不删历史决策和失败尝试，过时内容标「（历史记录）」。
- 推翻旧计划时写清：旧计划是什么、新计划是什么、为什么改。
- 新增 AC → 同步更新 CP、VF、NEG、覆盖矩阵；改指标定义 → 同步 golden；改 verifier → 同步反例测试。
- 没确认就不要把状态改成「已完成」。

---

## 交付前自检（必过）

```text
[ ] 目标和完成口径是和用户敲定的，不是自己假设的
[ ] 全中文（含标题）；路径真实或标 TBD，没有编造
[ ] 每条 AC 有编号、可客观判定、有验证方式和证据入口
[ ] Non-goals 写清了
[ ] 没把 runs/ 的易过期细节（完整命令 / git / env / hash / 原始日志）抄进 task 文件
[ ] 重型任务：关键 verifier 有反例；有总验证命令；覆盖矩阵完整
[ ] 标「已完成」的，有证据闭环
```

---

## 参考资料

- `assets/task_template.md`：可直接复制的模板（精简骨架 + 重型补充）。
- `references/section_guide.md`：每节「好 vs 差」对照、怎么写到位。
- `references/examples.md`：完整范例（重型评测任务 + 轻量论文修改任务）。
- `docs/<project>/execution_contract.md`：项目级 agent 执行契约。
- `runs/.../run_config.json`、`runs/.../verification/manifest.json`：某次运行 / 验证的复现单一真相。
