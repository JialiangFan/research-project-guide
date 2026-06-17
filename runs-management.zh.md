# Runs 管理指南

`runs/` 目录用于统一管理实验结果。它负责保存训练产物、评测结果、指标文件、日志、checkpoint 和其他实验执行后的证据。

`runs/` 不负责解释实验设计。实验设计和任务说明应该写在 `docs/` 中。

## 核心原则

每一次实验运行都对应一个唯一的 `run_id`，类似 Weights & Biases 的 run 机制。

一个 `run_id` 应该代表一次可复现的实验产物，其中包含该实验使用的方法、环境、backend、模型、seed、配置文件、训练输出以及后续评测结果。

为了避免目录结构过深，不建议把 simulator、env、model、seed 等信息全部拆成目录层级。推荐使用较扁平的目录结构，并将完整元信息写入 config 文件。

`run_config.json`（以及每次评测的 `eval_config.json`）是**复现的单一 source of truth**。复现一次实验只靠命令是不够的，因此 config 必须同时包含以下四样：

- **command**：本次实际执行的完整、可直接粘贴运行的命令；
- **git_commit**：产生该结果时的代码版本（commit 或 tag，并标记是否 dirty），这是复现的锚点；
- **env**：运行环境，例如 python 版本、关键依赖版本、GPU，或指向 run 目录下的环境快照文件；
- **data**：使用的数据集路径与版本/hash。

`docs/` 里的 `实验结果.md` 不重复抄这些信息，只指向对应 run 的 config 路径即可。

## 推荐目录结构

```text
runs/
  <method>/
    <run_id>/
      run_config.json

      train/
        train_config.json
        checkpoints/
        logs/
        artifacts/

      eval/
        <eval_id>/
          eval_config.json
          metrics.json
          raw_results/
```

其中：

- `<method>` 表示实验方法，例如 `expert`、`bc`、`icrl` 等；
- `<run_id>` 表示一次唯一的训练或实验运行；
- `<eval_id>` 表示一次唯一的评测任务；
- `run_config.json` 记录该 run 的完整元信息；
- `train/` 保存训练阶段产生的配置、模型、日志和其他产物；
- `eval/` 保存针对该 run 的一次或多次评测结果。

## run_id 规范

`run_id` 必须唯一，并建议包含少量关键可读信息，方便人工快速识别。

推荐格式：

```text
<date>_<time>_<backend>_<model>_<env>_s<seed>
```

示例：

```text
20260612_1530_isaacgym_dp_pickplace_s0
20260612_1715_mujoco_mlp_lift_s1
20260613_1015_real_gofa_dp_pickplace_s0
```

注意：`run_id` 只用于快速识别，不能作为完整元信息的唯一来源。完整信息必须记录在 `run_config.json` 中。

## run_config.json

每个 run 目录下必须包含 `run_config.json`，用于描述这次实验的完整信息，并作为复现的单一 source of truth。

示例：

```json
{
  "run_id": "20260612_1530_isaacgym_dp_pickplace_s0",
  "method": "icrl",
  "backend": "simulator",
  "backend_name": "isaacgym",
  "model_type": "diffusion_policy",
  "env_name": "pick_place",
  "seed": 0,
  "created_at": "2026-06-12T15:30:00",

  "command": "python scripts/train.py --config configs/icrl_pickplace.yaml --seed 0 --run_id 20260612_1530_isaacgym_dp_pickplace_s0",
  "git_commit": "a1b2c3d4",
  "git_dirty": false,
  "git_branch": "main",
  "env": {
    "python": "3.10.13",
    "key_packages": {"torch": "2.3.0", "isaacgym": "1.0rc4"},
    "gpu": "1x A100 80GB",
    "snapshot": "env.txt"
  },
  "data": {
    "name": "expert_pickplace",
    "path": "data/expert/pickplace_v2",
    "version": "v2",
    "hash": "sha256:0f1e2d..."
  },
  "notes": ""
}
```

其中以下四个字段为**必含**，用于复现：

- `command`：本次实际执行的完整命令，必须可直接复制粘贴运行，不能留 `...` 占位；
- `git_commit` / `git_dirty` / `git_branch`：产生该结果的代码版本；若 `git_dirty` 为 `true`，说明有未提交改动，复现不保证一致，应尽量避免；
- `env`：python 版本、关键依赖、GPU；`snapshot` 指向 run 目录下的环境快照文件（见下）；
- `data`：输入数据的路径与版本/hash。

此外，每个 run 目录下应保存一份**环境快照文件**（例如 `env.txt`，由 `pip freeze` 或 `conda env export` 生成），供精确复现使用。

对于 MVP、simulator、real-world 这类差异很大的环境，建议使用：

```json
{
  "backend": "real_world",
  "backend_name": "abb_gofa"
}
```

这样可以支持：

- MVP 快速验证；
- simulator 训练和评测；
- real-world 部署和评测；
- simulator 到 real-world transfer；
- 不同 backend 使用不同模型；
- 同一个模型跨不同 backend 做 eval。

## train 目录

训练结果必须写入：

```text
runs/<method>/<run_id>/train/
```

推荐结构：

```text
train/
  train_config.json
  checkpoints/
  logs/
  artifacts/
```

其中：

- `train_config.json`：本次训练实际使用的完整参数；
- `checkpoints/`：训练得到的模型权重；
- `logs/`：训练日志；
- `artifacts/`：其他训练产物，例如归一化统计、数据缓存、可视化结果等。

如果某些 run 没有训练阶段，例如只做 real-world evaluation，也可以没有 `train/`，但必须在 `run_config.json` 中写清楚。

## eval 目录

评测结果必须写入：

```text
runs/<method>/<run_id>/eval/<eval_id>/
```

同一个 run 可以有多次 eval，例如：

```text
runs/
  icrl/
    20260612_1530_isaacgym_dp_pickplace_s0/
      eval/
        success_isaacgym_pickplace_ckpt100_s0/
        robustness_isaacgym_noise0.1_ckpt100_s1/
        transfer_real_gofa_pickplace_ckpt100_t0/
```

## eval_id 规范

`eval_id` 必须唯一，并且必须包含 eval 类型和关键参数。

推荐格式：

```text
<eval_type>_<eval_backend>_<env>_<checkpoint>_<seed_or_trial>
```

示例：

```text
success_isaacgym_pickplace_ckpt100_s0
robustness_isaacgym_noise0.1_pickplace_ckpt100_s1
transfer_real_gofa_pickplace_ckpt100_t0
```

如果 eval 中包含关键参数，例如噪声强度、扰动类型、任务名称、checkpoint 编号，也应该体现在 `eval_id` 中。

## 每个 eval 目录必须包含的文件

每次评测完成后，`eval/<eval_id>/` 目录下至少需要包含：

```text
eval_config.json
metrics.json
raw_results/
```

### eval_config.json

记录本次 eval 实际使用的完整参数。

示例：

```json
{
  "eval_id": "robustness_isaacgym_noise0.1_pickplace_ckpt100_s1",
  "eval_type": "robustness",
  "backend": "simulator",
  "backend_name": "isaacgym",
  "env_name": "pick_place",
  "checkpoint": "checkpoints/ckpt100.pt",
  "seed": 1,
  "noise_level": 0.1,
  "num_episodes": 100,

  "command": "python scripts/eval.py --run_dir runs/icrl/20260612_1530_isaacgym_dp_pickplace_s0 --eval_type robustness --noise_level 0.1 --num_episodes 100 --seed 1",
  "git_commit": "a1b2c3d4",
  "git_dirty": false,
  "env": {"python": "3.10.13", "snapshot": "env.txt"}
}
```

eval 同样要记录自己的 `command`、`git_commit`、`env`，因为评测可能在与训练不同的代码版本或环境下运行。`checkpoint` 字段记录被评测的输入。

### metrics.json

记录用于 leaderboard 或结果汇总的核心指标。

示例：

```json
{
  "success_rate": 0.82,
  "mean_return": 134.5,
  "mean_cost": 0.12,
  "violation_rate": 0.04,
  "num_episodes": 100
}
```

`metrics.json` 应该保持结构稳定，方便后续自动汇总。

### raw_results/

保存原始评测结果，例如：

```text
raw_results/
  episodes.jsonl
  trajectories/
  logs/
  videos/
```

原始结果用于 debug、复查、画图和进一步分析，不要求完全统一格式，但必须保留足够信息以复现实验结论。

## eval 脚本要求

每个 eval 脚本应该支持以下两种方式之一来定位被评测的 run。

方式一：直接传入完整 run 目录：

```bash
python evaluate_icrl.py \
  --run_dir runs/icrl/20260612_1530_isaacgym_dp_pickplace_s0
```

方式二：传入 `run_id` 与 `runs_root`：

```bash
python evaluate_icrl.py \
  --run_id 20260612_1530_isaacgym_dp_pickplace_s0 \
  --runs_root runs/icrl
```

eval 脚本运行时必须：

1. 根据 `--run_dir` 或 `--run_id + --runs_root` 找到目标 run；
2. 读取该 run 下的 `run_config.json`；
3. 根据本次 eval 参数生成唯一的 `eval_id`；
4. 创建 `runs/<method>/<run_id>/eval/<eval_id>/`；
5. 写入 `eval_config.json`（必须包含本次 eval 的 `command`、`git_commit`、`env`）；
6. 写入 `metrics.json`；
7. 保存原始评测结果文件。

## leaderboard 汇总

后续 leaderboard 或结果汇总脚本可以递归扫描：

```text
runs/*/*/eval/*/metrics.json
```

然后结合：

```text
runs/<method>/<run_id>/run_config.json
runs/<method>/<run_id>/eval/<eval_id>/eval_config.json
runs/<method>/<run_id>/eval/<eval_id>/metrics.json
```

生成统一结果表。

## 适用范围

这套结构适合：

- imitation learning；
- reinforcement learning；
- robot learning；
- VLA / policy learning；
- supervised learning；
- benchmark eval；
- ablation study；
- simulator transfer；
- multi-seed experiment tracking；
- real-world validation。

它主要管理实验运行结果，不替代数据集版本管理、模型发布系统或线上服务日志系统。
