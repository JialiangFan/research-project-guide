# Research Project Guide / 科研项目结构管理指南

This repository collects practical guidance for organizing research-oriented AI projects.

这个仓库用于整理科研项目管理的一些 guidance，重点是让 AI / robotics / machine learning 项目更容易维护、复现、协作和交给 AI agent 实现。

## Core Idea / 核心思想

Separate a research project into three clear responsibilities:

把科研项目拆成三个清晰职责：

```text
docs/   specifications, task breakdowns, decisions, and status tracking
code/   implementation, scripts, configs, and reusable modules
runs/   experiment outputs, training results, evaluation results, and metrics
```

中文解释：

- `docs/` 负责说明要做什么、为什么做、怎么验收；
- `code/` 负责具体实现；
- `runs/` 负责保存实验执行后的证据和结果。

## Guidance Documents / 指南文档

### Chinese / 中文版

- [Docs 管理指南](docs-management.zh.md)
- [Runs 管理指南](runs-management.zh.md)

### English / 英文版

- [Docs Management Guide](docs-management.en.md)
- [Runs Management Guide](runs-management.en.md)

## Skills / 技能

Reusable skills for AI coding agents (Claude Code / Codex) live under `skills/`.

供 AI 编码 agent（Claude Code / Codex）复用的 skill 放在 `skills/` 下。

- [research-task-writer](skills/research-task-writer/) — guides agents to write clear, maintainable, all-Chinese task files under `docs/<project>/tasks/`. See its [README](skills/research-task-writer/README.md) for install and usage.

  指导 agent 在 `docs/<project>/tasks/` 下写出清晰、可维护的**全中文** task 文件。安装与使用见其 [README](skills/research-task-writer/README.md)。

## Recommended Project Layout / 推荐项目结构

Minimal recommended layout:

最小推荐结构：

```text
project/
  README.md
  docs/
  src/
  scripts/
  configs/
  runs/
  tests/
```

Full recommended layout:

完整推荐结构：

```text
project/
  README.md
  docs/
  src/
  scripts/
  configs/
  runs/
  data/
  tests/
  tools/
  notebooks/
  assets/
```

## Directory Responsibilities / 目录职责

- `README.md`: project overview, quick start, main commands, and links to important docs.
- `docs/`: experiment specifications, task breakdowns, design decisions, status tracking, and blockers. See [Docs Management Guide](docs-management.en.md) / [Docs 管理指南](docs-management.zh.md).
- `src/`: reusable source code, including methods, models, environment adapters, training logic, evaluation logic, metrics, and utilities.
- `scripts/`: thin command-line entry points, such as training, evaluation, leaderboard generation, and maintenance scripts.
- `configs/`: reusable configuration files for training, evaluation, models, environments, and backends.
- `runs/`: experiment outputs, including configs, checkpoints, logs, metrics, raw evaluation results, and artifacts. See [Runs Management Guide](runs-management.en.md) / [Runs 管理指南](runs-management.zh.md).
- `data/`: dataset metadata, manifests, download scripts, small examples, and format documentation. Large datasets should usually not be committed to git.
- `tests/`: focused tests for configs, path conventions, metrics, data formats, checkpoint loading, and smoke runs.
- `tools/`: utility scripts for result aggregation, validation, conversion, cleanup, and project maintenance.
- `notebooks/`: exploratory analysis and visualization. Stable logic should be moved into `src/`, `scripts/`, or `tools/`.
- `assets/`: figures, diagrams, demo images, GIFs, and README or paper-facing visual assets.

中文说明：

- `README.md`：项目概览、快速开始、主要命令和重要文档链接。
- `docs/`：实验 specification、任务拆分、设计决策、完成状态和 blocker。具体规范见 [Docs 管理指南](docs-management.zh.md) / [Docs Management Guide](docs-management.en.md)。
- `src/`：可复用核心代码，包括方法、模型、环境 adapter、训练逻辑、评测逻辑、metrics 和工具函数。
- `scripts/`：轻量命令行入口，例如训练、评测、leaderboard 生成和维护脚本。
- `configs/`：训练、评测、模型、环境和 backend 的配置文件。
- `runs/`：实验输出，包括 config、checkpoint、日志、metrics、原始评测结果和 artifacts。具体规范见 [Runs 管理指南](runs-management.zh.md) / [Runs Management Guide](runs-management.en.md)。
- `data/`：数据集元信息、manifest、下载脚本、小样例和格式说明。大数据通常不应该直接提交到 git。
- `tests/`：针对 config、路径规范、metrics、数据格式、checkpoint loading 和 smoke run 的重点测试。
- `tools/`：结果汇总、校验、格式转换、清理和项目维护工具。
- `notebooks/`：探索性分析和可视化。稳定逻辑应该沉淀到 `src/`、`scripts/` 或 `tools/`。
- `assets/`：图片、架构图、demo 图、GIF，以及 README 或论文中使用的视觉素材。

Avoid creating separate top-level `outputs/`, `results/`, `logs/`, or `checkpoints/` directories unless there is a strong reason. These usually belong under `runs/`.

除非有明确原因，不建议额外创建顶层 `outputs/`、`results/`、`logs/` 或 `checkpoints/` 目录。这些内容通常应该归到 `runs/` 下面。

## Workflow / 工作方式

Recommended workflow:

1. Write the experiment specification in `docs/`.
2. Break the experiment into concrete sub-tasks.
3. Describe each task in a standalone markdown file.
4. Implement the code against the specification.
5. Run experiments; the scripts write each run's full reproduction info (command, code version, environment, data) into `run_config.json` under `runs/`.
6. Update `docs/` to point at the results: register run directories and metrics in `experiment_results.md`, and update the project README status.

推荐工作方式：

1. 先在 `docs/` 中写清楚 experiment specification；
2. 把大实验拆成多个 sub-task；
3. 每个 task 用独立 markdown 文件描述；
4. 根据 specification 实现代码；
5. 跑实验；脚本把每次 run 的完整复现信息（命令、代码版本、环境、数据）写入 `runs/` 下的 `run_config.json`；
6. 回填 `docs/`：在 `实验结果.md` 登记 run 目录和指标，并更新项目 README 的状态。

## Design Goal / 设计目标

The goal is to make research projects easier to inspect, reproduce, extend, and hand off.

这套结构的目标是让科研项目像产品管理一样可追踪：

- goals are clear / 目标清楚；
- scope is clear / 范围清楚；
- task boundaries are clear / 任务边界清楚；
- acceptance criteria are clear / 验收标准清楚；
- result paths are clear / 结果路径清楚；
- blockers are explicit / 当前阻碍清楚。

This structure is useful for research codebases involving training, evaluation, multiple seeds, simulators, MVP environments, real-world validation, ablation studies, and leaderboard generation.

这套结构适合有训练、评测、多 seed、多 simulator、MVP、real-world validation、ablation study 和 leaderboard 汇总需求的 research codebase。
