# Docs 管理指南

`docs/` 目录负责管理每个项目的 specification。它不是简单的说明文档，而是项目任务边界、实现要求、验收标准和当前状态的 source of truth。

实验的复现信息（命令、代码版本、环境、数据）不写在 `docs/`，而是由脚本写入 `runs/` 下对应 run 的 `run_config.json` / `eval_config.json`（见 `runs-management.zh.md`）。`docs/` 只负责说明和**指向**这些信息。

## 核心职责

`docs/` 应该回答以下问题：

- 这个项目要解决什么问题？
- 为什么这个项目重要？
- 项目包含哪些 sub-task？
- 每个 sub-task 的输入和输出是什么？
- 用什么技术、算法、simulator 或 real-world setup？
- 代码应该实现在哪里？
- 如何发起一次新的实验 run？
- 结果在 `runs/` 的什么位置？是否达到验收标准？
- 当前 task 是否完成？如果没完成，具体 blocker 是什么？

## 推荐目录结构

每个项目直接对应 `docs/<project_name>/`，例如 `MVP`、`sim1`。项目下用 `tasks/` 逐个记录子任务，
并在项目级维护两个汇总文件：`运行命令.md` 和 `实验结果.md`。

```text
docs/
  <project_name>/          # MVP, sim1, ...
    README.md              # 项目概览 + 导航 + 状态一览
    tasks/
      01_<task_name>.md
      02_<task_name>.md
    运行命令.md             # 如何发起一次新 run 的入口指南
    实验结果.md             # 结果汇总（小表 + 指向 runs 下 config 的路径）
    decisions.md           # 可选
    changelog.md           # 可选
```

例如：

```text
docs/
  MVP/
    README.md
    tasks/
      01_collect_expert_data.md
      02_train_bc_policy.md
    运行命令.md
    实验结果.md
  sim1/
    README.md
    tasks/
      01_train_icrl_policy.md
      02_eval_in_simulator.md
    运行命令.md
    实验结果.md
```

关键约定：

- **每个项目一个目录**，名字简洁直观（`MVP`、`sim1`、`real1` 等）。
- **子任务写在 `tasks/` 里**，每个 task 一个独立 markdown 文件。
- **复现信息的 source of truth 在 `runs/`**：每次 run 的 `run_config.json` 记录了实际命令、代码版本、环境和数据。`docs/` 不重复抄这些，只指向它们。
- **`运行命令.md` 只是"怎么发起新 run"的入口指南**，`实验结果.md` 只是"结果一览 + 指针"。

## 项目 README

每个项目目录下的 `README.md` 是该项目的入口，应该让人几秒钟内知道去哪看什么，并一眼看到项目当前状态。

推荐结构：

```markdown
# <Project Name>

> 快速理解本项目：
> - 看结果和数据 → `实验结果.md`
> - 复现某次实验 → 打开对应 run 目录里的 `run_config.json`（含命令 / 代码版本 / 环境 / 数据）
> - 发起一次新 run → `运行命令.md`
> - 了解每个子任务 → `tasks/`
> - 原始产物（checkpoint / 日志 / 视频）→ `runs/`

## Status

| Task | Status | 最新结果 | 下一步 |
|---|---|---|---|
| 01 Collect expert data | Done | 见 `实验结果.md` | - |
| 02 Train policy | In Progress | TBD | 跑完 seed 0-2 |
| 03 Eval in simulator | Blocked | - | 补 eval config |

## Goal

描述这个项目的目标。

## Motivation

说明为什么要做这个项目，它验证什么假设，服务于论文、系统或产品中的哪一部分。

## Scope

说明本项目包含什么，不包含什么。

## Environment

说明项目涉及的环境类型（MVP / Simulator / Real-world）和具体 backend，例如 Isaac Gym、MuJoCo、RoboCasa、LIBERO、ABB GoFa、UMI 等。

## Methods

列出涉及的方法，例如 expert、BC、ICRL、diffusion policy、safety filter 等。

## Success Criteria

说明这个项目完成的标准（最好是可量化的指标阈值）。
```

## Task Markdown 模板

每个 sub-task 建议使用一个独立 markdown 文件，放在 `tasks/` 下。

Task 文件只描述目标、输入输出契约和验收标准；**怎么发起 run 见项目级 `运行命令.md`，本次实际命令和复现信息在对应 run 的 `run_config.json`，结果路径见 `实验结果.md`**。

推荐模板：

````markdown
# Task <ID>: <Task Name>

## Status

Status: `Planned | In Progress | Blocked | Done | Deprecated`

## Objective

说明这个 sub-task 要完成什么。

## Background

说明为什么需要这个 task，它依赖哪个上游 task，服务于哪个下游 task。

## Technical Details

### Method

说明使用的技术、算法、模型、simulator 或 real-world backend。

### Input

说明输入是什么，例如 expert trajectories、checkpoint、environment config、RGB-D image、action history 等。

### Output

说明输出是什么，例如 trained checkpoint、eval metrics、trajectories、videos、logs、leaderboard row 等。

## Implementation Plan

说明代码应该实现在哪里。

```text
Code paths:
- `train/...`
- `eval/...`
- `configs/...`
- `scripts/...`
```

如果需要新增文件，也要写清楚。

## Metrics

列出这个 task 关心的指标并**给出定义**，例如 success_rate（成功 episode 数 / 总 episode 数）、mean_return、violation_rate、mean_cost、num_episodes 等。指标定义要稳定，方便跨 run 比较。

## Acceptance Criteria

这个 task 只有满足以下条件才算完成：

- [ ] 代码实现完成
- [ ] 可以复现（发起方式见 `运行命令.md`，本次实际命令 / 代码版本 / 环境 / 数据已写入对应 run 的 `run_config.json`）
- [ ] 结果写入正确的 `runs/` 目录，并在 `实验结果.md` 登记
- [ ] 生成必要的 config 文件和 `metrics.json`
- [ ] 主要结果已经检查过，并标注是否达到 Success Criteria

## Blockers

如果没有完成，写清楚具体阻碍。

## Notes

记录其他实现细节、设计取舍，以及**偏离计划之处**（本想做 X、实际做了 Y、原因 Z）。
````

> 注意：复现命令、代码版本、环境、数据都不写在 task 文件里，而是由脚本写入 `runs/` 下对应 run 的 `run_config.json` / `eval_config.json`。

## 运行命令.md

`运行命令.md` 说明本项目每个子任务**如何发起一次新的实验 run**：调用哪个 script、参数模板、前置条件。

注意：每次实际执行的完整命令由脚本写入对应 run 的 `run_config.json`（`command` 字段），这里只给入口和模板，不逐次记录历史命令。

推荐模板：

````markdown
# <project_name> 运行命令（如何发起新 run）

本项目每个子任务发起一次实验 run 的方式。`run_id` 的拼法见 `runs-management.zh.md`；
每次实际执行的完整命令会写入对应 run 的 `run_config.json`。

## Task 01: <task_name>

对应文档：`tasks/01_<task_name>.md`

```bash
# 训练
python scripts/train.py --config configs/icrl_pickplace.yaml --seed 0
# 评测
python scripts/eval.py --run_dir runs/icrl/<run_id> --eval_type success
```

前置条件：所需数据 / checkpoint / 环境。

## Task 02: <task_name>

对应文档：`tasks/02_<task_name>.md`

```bash
python scripts/...
```
````

## 实验结果.md

`实验结果.md` 汇总本项目所有子任务的结果，让人一眼看到状态、关键指标和是否达标，并指向 `runs/` 下含完整信息的 config。

约定：

- 数字以 `runs/.../eval/<eval_id>/metrics.json` 为准，表里的值与它保持一致；
- 复现命令、代码版本、环境、数据见对应 run 的 `run_config.json`，本文件不重复；
- run 目录沿用 `runs-management.zh.md` 的 `run_id` / `eval_id` 约定。

推荐模板：

```markdown
# <project_name> 实验结果

数字以 metrics.json 为准；复现信息见对应 run 的 run_config.json。

| Task | 状态 | 是否达标 | 关键指标 | run 目录 |
|---|---|---|---|---|
| 01 <name> | Done | 是 | success_rate=0.82 | `runs/icrl/<run_id>/` |
| 02 <name> | In Progress | TBD | TBD | `runs/icrl/<run_id>/` |

## Task 01: <name>

- 结论：success_rate 0.82 ≥ 目标 0.80，达标。
- run 目录：`runs/icrl/<run_id>/`（复现信息见其中的 `run_config.json`）
- 指标文件：`runs/icrl/<run_id>/eval/<eval_id>/metrics.json`
- 评测目录：`runs/icrl/<run_id>/eval/<eval_id>/`
```

## Status 规范

推荐统一使用以下状态：

```text
Planned      尚未开始
In Progress  正在实现
Blocked      被阻塞，需要解决问题
Done         已完成，并且结果可复现
Deprecated   已废弃，不再使用
```

## 重要原则

不要只写技术细节。文档应该同时包含目标、任务边界、验收标准和结果指向。

职责划分要清晰：

- **`runs/` 是复现的单一 source of truth**：每次 run 的 `run_config.json` / `eval_config.json` 必含 command、git_commit、env、data（见 `runs-management.zh.md`），`metrics.json` 是结果数字的来源；
- **`docs/` 只做说明和指针**：`实验结果.md` 给一览和结论并指向上述 config，不重复抄复现信息；`运行命令.md` 只说明如何发起新 run；
- **task 文件只描述**目标、input / output contract、metrics 定义、验收标准、blockers 和偏离计划之处。

这样一处即可复现整个项目，AI agent 或合作者接手时也不需要猜测什么算完成、结果可不可信、怎么复现。
