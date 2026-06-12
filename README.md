# 科研项目结构管理指南

这个仓库用于整理我们对科研项目管理的一些 guidance，重点是让 AI / robotics / machine learning 项目更容易维护、复现、协作和交给 AI agent 实现。

核心思想是把项目拆成三个清晰职责：

```text
docs/   管 specification、任务拆分、设计决策和完成状态
code/   管具体实现、脚本、配置和可复用模块
runs/   管实验产物、训练结果、评测结果和指标文件
```

其中：

- `docs/` 负责说明要做什么、为什么做、怎么验收；
- `code/` 负责实现；
- `runs/` 负责保存实验执行后的证据和结果。

## Guidance 文档

- [Docs 管理指南](docs-management.md)
- [Runs 管理指南](runs-management.md)

## 推荐项目结构

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

## 基本原则

科研项目不应该只靠代码目录表达项目状态。每个重要实验都应该有清晰的 specification，每个 sub-task 都应该有明确的输入、输出、实现路径、复现方式、实验结果路径和完成状态。

推荐的工作方式是：

1. 先在 `docs/` 中写清楚 experiment specification；
2. 把大实验拆成多个 sub-task；
3. 每个 task 用独立 markdown 文件描述；
4. 根据 specification 实现代码；
5. 实验结果统一写入 `runs/`；
6. 最后把实际代码路径、运行命令和结果路径回填到文档中。

这样做的目标是让项目像产品管理一样可追踪：

- 目标清楚；
- 范围清楚；
- 任务边界清楚；
- 验收标准清楚；
- 结果路径清楚；
- 当前阻碍清楚。

这套结构适合大多数 research codebase，尤其适合有训练、评测、多 seed、多 simulator、MVP、real-world validation、ablation study 和 leaderboard 汇总需求的项目。
