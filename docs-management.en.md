# Docs Management Guide

The `docs/` directory manages specifications for each project. It is not just a place for notes. It should act as the source of truth for task boundaries, implementation requirements, acceptance criteria, and current status.

Reproduction information (command, code version, environment, data) is not written in `docs/`. It is written by the scripts into the `run_config.json` / `eval_config.json` of the corresponding run under `runs/` (see `runs-management.en.md`). The `docs/` directory only describes things and **points to** that information.

## Responsibilities

The `docs/` directory should answer:

- What problem is this project trying to solve?
- Why is this project important?
- What sub-tasks does it contain?
- What are the inputs and outputs of each sub-task?
- What technology, algorithm, simulator, or real-world setup is used?
- Where should the code be implemented?
- How do you launch a new experiment run?
- Where are the results under `runs/`? Did they meet the acceptance criteria?
- Is the task complete? If not, what is the concrete blocker?

## Recommended Structure

Each project maps directly to `docs/<project_name>/`, such as `MVP` or `sim1`. Sub-tasks are recorded
one per file under `tasks/`, and each project keeps two project-level summary files:
`run_commands.md` and `experiment_results.md`.

```text
docs/
  <project_name>/          # MVP, sim1, ...
    README.md              # project overview + navigation + status at a glance
    tasks/
      01_<task_name>.md
      02_<task_name>.md
    run_commands.md        # how to launch a new run
    experiment_results.md  # results summary (small table + pointers to configs under runs/)
    decisions.md           # optional
    changelog.md           # optional
```

Example:

```text
docs/
  MVP/
    README.md
    tasks/
      01_collect_expert_data.md
      02_train_bc_policy.md
    run_commands.md
    experiment_results.md
  sim1/
    README.md
    tasks/
      01_train_icrl_policy.md
      02_eval_in_simulator.md
    run_commands.md
    experiment_results.md
```

Key conventions:

- **One directory per project**, with a short and intuitive name (`MVP`, `sim1`, `real1`, ...).
- **Sub-tasks live in `tasks/`**, one standalone markdown file per task.
- **The source of truth for reproduction is in `runs/`**: each run's `run_config.json` records the actual command, code version, environment, and data. `docs/` does not duplicate these; it only points to them.
- **`run_commands.md` is only a "how to launch a new run" guide**, and `experiment_results.md` is only a "results at a glance + pointers" view.

## Project README

Each project directory should have a `README.md` as its entry point. It should let a reader know within seconds where to look for what, and show the current project status at a glance.

Recommended structure:

```markdown
# <Project Name>

> Quick orientation:
> - See results and data → `experiment_results.md`
> - Reproduce a run → open the `run_config.json` in the corresponding run directory (command / code version / environment / data)
> - Launch a new run → `run_commands.md`
> - Understand each sub-task → `tasks/`
> - Raw artifacts (checkpoints / logs / videos) → `runs/`

## Status

| Task | Status | Latest Result | Next Step |
|---|---|---|---|
| 01 Collect expert data | Done | see `experiment_results.md` | - |
| 02 Train policy | In Progress | TBD | run seeds 0-2 |
| 03 Eval in simulator | Blocked | - | add eval config |

## Goal

Describe the goal of the project.

## Motivation

Explain why this project matters, what hypothesis it validates, and which part of the paper, system, or product it supports.

## Scope

Define what is included and what is explicitly out of scope.

## Environment

Describe the involved environment types (MVP / Simulator / Real-world) and concrete backends, such as Isaac Gym, MuJoCo, RoboCasa, LIBERO, ABB GoFa, or UMI.

## Methods

List the involved methods, such as expert, behavior cloning, ICRL, diffusion policy, or safety filter.

## Success Criteria

Define what must be true for the full project to be considered complete (ideally quantifiable metric thresholds).
```

## Task Markdown Template

Each sub-task should be described in a standalone markdown file under `tasks/`.

A task file only describes the objective, the input/output contract, and the acceptance criteria; **how to launch a run is in the project-level `run_commands.md`, the actual command and reproduction info live in the corresponding run's `run_config.json`, and result paths are in `experiment_results.md`**.

Recommended template:

````markdown
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

## Metrics

List the metrics that matter for this task and **define them**, such as success_rate (successful episodes / total episodes), mean_return, violation_rate, mean_cost, or num_episodes. Keep metric definitions stable so results are comparable across runs.

## Deliverables

> **Must be filled in when Status is Done.**

Output files:

- `runs/icrl/<run_id>/checkpoints/best.pt` — best checkpoint
- `runs/icrl/<run_id>/eval/<eval_id>/metrics.json` — evaluation metrics

Commands that produced the deliverables (actual shell commands):

```bash
python scripts/train.py --config configs/icrl_pickplace.yaml --seed 0
```

Verification commands (to inspect / validate the deliverables):

```bash
python scripts/eval.py --run_dir runs/icrl/<run_id> --eval_type success
cat runs/icrl/<run_id>/eval/<eval_id>/metrics.json
```

## Acceptance Criteria

This task is complete only when:

- [ ] the code implementation exists;
- [ ] it is reproducible (how to launch is in `run_commands.md`; the actual command / code version / environment / data are written into the corresponding run's `run_config.json`);
- [ ] the result is written to the correct `runs/` directory and registered in `experiment_results.md`;
- [ ] required config files and `metrics.json` are generated;
- [ ] the main result has been checked and marked as meeting or not meeting the Success Criteria;
- [ ] the Deliverables section is filled in (output file paths, production commands, verification commands).

## Blockers

If the task is not complete, describe the concrete blocker.

## Notes

Record implementation details, design tradeoffs, and **deviations from the plan** (intended X, actually did Y, because Z).
````

> Note: reproduction commands, code version, environment, and data are not written in the task file. They are written by the scripts into the `run_config.json` / `eval_config.json` of the corresponding run under `runs/`.

## run_commands.md

`run_commands.md` describes **how to launch a new experiment run** for each sub-task: which script to call, the argument template, and prerequisites.

Note: the exact command actually executed is written by the script into the corresponding run's `run_config.json` (the `command` field). This file only gives the entry points and templates; it does not log historical commands one by one.

Recommended template:

````markdown
# <project_name> Run Commands (how to launch a new run)

How to launch one experiment run for each sub-task in this project. The `run_id` format is in
`runs-management.en.md`; the exact command actually executed is written into the corresponding
run's `run_config.json`.

## Task 01: <task_name>

Doc: `tasks/01_<task_name>.md`

```bash
# train
python scripts/train.py --config configs/icrl_pickplace.yaml --seed 0
# eval
python scripts/eval.py --run_dir runs/icrl/<run_id> --eval_type success
```

Prerequisites: required data / checkpoint / environment.

## Task 02: <task_name>

Doc: `tasks/02_<task_name>.md`

```bash
python scripts/...
```
````

## experiment_results.md

`experiment_results.md` aggregates the results of all sub-tasks so a reader sees status, key metrics, and whether targets were met at a glance, then points to the configs under `runs/` that hold the full information.

Conventions:

- numbers come from `runs/.../eval/<eval_id>/metrics.json`, and the table values stay consistent with it;
- reproduction command, code version, environment, and data live in the corresponding run's `run_config.json`; this file does not duplicate them;
- run directories follow the `run_id` / `eval_id` conventions in `runs-management.en.md`.

Recommended template:

```markdown
# <project_name> Experiment Results

Numbers come from metrics.json; reproduction info lives in each run's run_config.json.

| Task | Status | Meets Target | Key Metrics | Run Dir |
|---|---|---|---|---|
| 01 <name> | Done | Yes | success_rate=0.82 | `runs/icrl/<run_id>/` |
| 02 <name> | In Progress | TBD | TBD | `runs/icrl/<run_id>/` |

## Task 01: <name>

- Conclusion: success_rate 0.82 >= target 0.80, meets target.
- Run dir: `runs/icrl/<run_id>/` (reproduction info in its `run_config.json`)
- Metrics file: `runs/icrl/<run_id>/eval/<eval_id>/metrics.json`
- Eval dir: `runs/icrl/<run_id>/eval/<eval_id>/`
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

Do not only write technical details. Documentation should also capture goals, task boundaries, acceptance criteria, and where the results are.

Keep responsibilities clear:

- **`runs/` is the single source of truth for reproduction**: each run's `run_config.json` / `eval_config.json` must contain command, git_commit, env, and data (see `runs-management.en.md`), and `metrics.json` is the source for result numbers;
- **`docs/` only describes and points**: `experiment_results.md` gives an overview plus conclusions and points to those configs, without duplicating reproduction info; `run_commands.md` only describes how to launch a new run;
- **task files only describe** the objective, the input/output contract, metric definitions, acceptance criteria, blockers, and deviations from the plan.

This makes the whole project reproducible from one place, and lets an AI agent or collaborator continue the work without guessing what "done" means, whether the results can be trusted, or how to reproduce them.
