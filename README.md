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

## Recommended Project Layout / 推荐项目结构

```text
project/
  docs/
  src/
  scripts/
  configs/
  runs/
  tests/
  README.md
```

## Workflow / 工作方式

Recommended workflow:

1. Write the experiment specification in `docs/`.
2. Break the experiment into concrete sub-tasks.
3. Describe each task in a standalone markdown file.
4. Implement the code against the specification.
5. Save experiment outputs under `runs/`.
6. Update the documentation with actual code paths, commands, and result paths.

推荐工作方式：

1. 先在 `docs/` 中写清楚 experiment specification；
2. 把大实验拆成多个 sub-task；
3. 每个 task 用独立 markdown 文件描述；
4. 根据 specification 实现代码；
5. 实验结果统一写入 `runs/`；
6. 最后把实际代码路径、运行命令和结果路径回填到文档中。

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
