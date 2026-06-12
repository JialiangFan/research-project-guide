# Docs 管理指南

`docs/` 目录负责管理每个大实验的 specification。它不是简单的说明文档，而是项目任务边界、实现要求、验收标准和当前状态的 source of truth。

## 核心职责

`docs/` 应该回答以下问题：

- 这个实验要解决什么问题？
- 为什么这个实验重要？
- 实验包含哪些 sub-task？
- 每个 sub-task 的输入和输出是什么？
- 用什么技术、算法、simulator 或 real-world setup？
- 代码应该实现在哪里？
- 如何复现结果？
- 结果应该保存到 `runs/` 的什么位置？
- 当前 task 是否完成？
- 如果没完成，具体 blocker 是什么？

## 推荐目录结构

```text
docs/
  specs/
    <experiment_name>/
      README.md
      tasks/
        01_<task_name>.md
        02_<task_name>.md
        03_<task_name>.md
      decisions.md
      changelog.md
```

例如：

```text
docs/
  specs/
    icrl_sim_to_real/
      README.md
      tasks/
        01_collect_expert_data.md
        02_train_icrl_policy.md
        03_eval_in_simulator.md
        04_eval_in_real_world.md
      decisions.md
      changelog.md
```

## Experiment README

每个大实验目录下的 `README.md` 是该实验的总说明。

推荐结构：

```markdown
# <Experiment Name>

## Goal

描述这个大实验的目标。

## Motivation

说明为什么要做这个实验，它验证什么假设，服务于论文、系统或产品中的哪一部分。

## Scope

说明本实验包含什么，不包含什么。

## Environment

说明实验涉及的环境类型，例如：

- MVP
- Simulator
- Real-world

并写清楚具体 backend，例如 Isaac Gym、MuJoCo、RoboCasa、LIBERO、ABB GoFa、UMI 等。

## Methods

列出涉及的方法，例如 expert、BC、ICRL、diffusion policy、safety filter 等。

## Sub-tasks

| Task ID | Task | Status | Code Path | Runs Path | Notes |
|---|---|---|---|---|---|
| 01 | Collect expert data | Done | `scripts/...` | `runs/...` |  |
| 02 | Train policy | In Progress | `train/...` | `runs/...` |  |
| 03 | Eval in simulator | Blocked | `eval/...` | TBD | missing config |

## Success Criteria

说明这个大实验完成的标准。
```

## Task Markdown 模板

每个 sub-task 建议使用一个独立 markdown 文件。

推荐模板：

```markdown
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

## Reproduction

写清楚如何复现。

```bash
python ...
```

## Experiment Results

写清楚结果应该保存到哪个 `runs/` 路径。

```text
Run path:
runs/...

Eval path:
runs/.../eval/...
```

## Metrics

列出这个 task 关心的指标，例如 success_rate、mean_return、violation_rate、mean_cost、num_trials、num_episodes 等。

## Acceptance Criteria

这个 task 只有满足以下条件才算完成：

- [ ] 代码实现完成
- [ ] 可以通过命令复现
- [ ] 结果写入正确的 `runs/` 目录
- [ ] 生成必要的 config 文件
- [ ] 生成必要的 metrics 文件
- [ ] 文档中记录了代码路径和 runs 路径
- [ ] 主要结果已经检查过

## Blockers

如果没有完成，写清楚具体阻碍。

## Notes

记录其他实现细节、设计取舍或临时决定。
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

不要只写技术细节。文档应该同时包含目标、任务边界、验收标准和结果路径。

尤其需要明确：

- acceptance criteria；
- input / output contract；
- code path；
- runs path；
- blockers；
- metrics definition；
- environment / backend。

这样 AI agent 或合作者接手时，不需要猜测什么算完成。
