# Docs Management Guide

The `docs/` directory manages specifications for major experiments. It is not just a place for notes. It should act as the source of truth for task boundaries, implementation requirements, acceptance criteria, and current status.

## Responsibilities

The `docs/` directory should answer:

- What problem is this experiment trying to solve?
- Why is this experiment important?
- What sub-tasks does it contain?
- What are the inputs and outputs of each sub-task?
- What technology, algorithm, simulator, or real-world setup is used?
- Where should the code be implemented?
- How can the result be reproduced?
- Where should the result be stored under `runs/`?
- Is the task complete?
- If it is not complete, what is the concrete blocker?

## Recommended Structure

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

Example:

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

Each major experiment should have a top-level `README.md`.

Recommended structure:

```markdown
# <Experiment Name>

## Goal

Describe the goal of the experiment.

## Motivation

Explain why this experiment matters, what hypothesis it validates, and which part of the paper, system, or product it supports.

## Scope

Define what is included and what is explicitly out of scope.

## Environment

Describe the involved environment types, such as:

- MVP
- Simulator
- Real-world

Also specify concrete backends, such as Isaac Gym, MuJoCo, RoboCasa, LIBERO, ABB GoFa, UMI, or other relevant systems.

## Methods

List the involved methods, such as expert, behavior cloning, ICRL, diffusion policy, or safety filter.

## Sub-tasks

| Task ID | Task | Status | Code Path | Runs Path | Notes |
|---|---|---|---|---|---|
| 01 | Collect expert data | Done | `scripts/...` | `runs/...` |  |
| 02 | Train policy | In Progress | `train/...` | `runs/...` |  |
| 03 | Eval in simulator | Blocked | `eval/...` | TBD | missing config |

## Success Criteria

Define what must be true for the full experiment to be considered complete.
```

## Task Markdown Template

Each sub-task should be described in a standalone markdown file.

Recommended template:

```markdown
# Task <ID>: <Task Name>

## Status

Status: `Planned | In Progress | Blocked | Done | Deprecated`

## Objective

Describe what this sub-task should accomplish.

## Background

Explain why this task is needed, which upstream task it depends on, and which downstream task it supports.

## Technical Details

### Method

Describe the technology, algorithm, model, simulator, or real-world backend used by this task.

### Input

Describe the inputs, such as expert trajectories, checkpoints, environment config, RGB-D images, or action history.

### Output

Describe the outputs, such as trained checkpoints, evaluation metrics, trajectories, videos, logs, or leaderboard rows.

## Implementation Plan

Describe where the code should be implemented.

```text
Code paths:
- `train/...`
- `eval/...`
- `configs/...`
- `scripts/...`
```

If new files are needed, list them explicitly.

## Reproduction

Describe how to reproduce the result.

```bash
python ...
```

## Experiment Results

Describe where the result should be stored under `runs/`.

```text
Run path:
runs/...

Eval path:
runs/.../eval/...
```

## Metrics

List the metrics that matter for this task, such as success_rate, mean_return, violation_rate, mean_cost, num_trials, or num_episodes.

## Acceptance Criteria

This task is complete only when:

- [ ] the code implementation exists;
- [ ] the result can be reproduced with the documented command;
- [ ] the result is written to the correct `runs/` directory;
- [ ] required config files are generated;
- [ ] required metrics files are generated;
- [ ] the documentation records the code path and runs path;
- [ ] the main result has been checked.

## Blockers

If the task is not complete, describe the concrete blocker.

## Notes

Record implementation details, design tradeoffs, or temporary decisions.
```

## Status Values

Use a small and consistent set of statuses:

```text
Planned      not started
In Progress  implementation or experiment is ongoing
Blocked      blocked by a concrete missing dependency or decision
Done         implemented, reproduced, and documented
Deprecated   no longer used
```

## Important Principles

Do not only write technical details. Documentation should also capture goals, task boundaries, acceptance criteria, and result paths.

The most important fields are:

- acceptance criteria;
- input / output contract;
- code path;
- runs path;
- blockers;
- metrics definition;
- environment / backend.

This makes it possible for an AI agent or collaborator to continue the work without guessing what “done” means.
