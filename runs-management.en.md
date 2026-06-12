# Runs Management Guide

The `runs/` directory is used to manage experiment results. It stores training artifacts, evaluation results, metrics files, logs, checkpoints, and other evidence produced by experiment execution.

The `runs/` directory should not explain the experiment design. Experiment design and task descriptions belong in `docs/`.

## Core Principle

Each experiment run should have a unique `run_id`, similar to the run mechanism used by Weights & Biases.

A `run_id` should represent one reproducible experiment artifact, including the method, environment, backend, model, seed, configuration files, training outputs, and later evaluation results.

To avoid overly deep directory structures, do not split simulator, environment, model, and seed into many directory levels. Use a relatively flat directory structure and store complete metadata in config files.

## Recommended Structure

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

Where:

- `<method>` is the experiment method, such as `expert`, `bc`, or `icrl`;
- `<run_id>` identifies one unique training or experiment run;
- `<eval_id>` identifies one unique evaluation run;
- `run_config.json` records complete metadata for the run;
- `train/` stores training configs, checkpoints, logs, and artifacts;
- `eval/` stores one or more evaluation results for the run.

## run_id Convention

The `run_id` must be unique and should include a small amount of human-readable information.

Recommended format:

```text
<date>_<time>_<backend>_<model>_<env>_s<seed>
```

Examples:

```text
20260612_1530_isaacgym_dp_pickplace_s0
20260612_1715_mujoco_mlp_lift_s1
20260613_1015_real_gofa_dp_pickplace_s0
```

The `run_id` is only a quick identifier. It should not be the only source of metadata. Complete metadata must be stored in `run_config.json`.

## run_config.json

Each run directory must contain `run_config.json`.

Example:

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
  "notes": ""
}
```

For very different execution environments such as MVP, simulator, and real-world, use fields like:

```json
{
  "backend": "real_world",
  "backend_name": "abb_gofa"
}
```

This supports:

- MVP validation;
- simulator training and evaluation;
- real-world deployment and evaluation;
- simulator-to-real-world transfer;
- different models for different backends;
- cross-backend evaluation for the same model.

## train Directory

Training results must be written to:

```text
runs/<method>/<run_id>/train/
```

Recommended structure:

```text
train/
  train_config.json
  checkpoints/
  logs/
  artifacts/
```

Where:

- `train_config.json` records the full resolved training parameters;
- `checkpoints/` stores model weights;
- `logs/` stores training logs;
- `artifacts/` stores other training artifacts, such as normalization statistics, cached data, or visualizations.

Some runs may not have a training phase, for example real-world-only evaluation runs. In that case, `train/` may be absent, but this should be documented in `run_config.json`.

## eval Directory

Evaluation results must be written to:

```text
runs/<method>/<run_id>/eval/<eval_id>/
```

One run can have multiple evaluations:

```text
runs/
  icrl/
    20260612_1530_isaacgym_dp_pickplace_s0/
      eval/
        success_isaacgym_pickplace_ckpt100_s0/
        robustness_isaacgym_noise0.1_pickplace_ckpt100_s1/
        transfer_real_gofa_pickplace_ckpt100_t0/
```

## eval_id Convention

The `eval_id` must be unique and should include the evaluation type and key parameters.

Recommended format:

```text
<eval_type>_<eval_backend>_<env>_<checkpoint>_<seed_or_trial>
```

Examples:

```text
success_isaacgym_pickplace_ckpt100_s0
robustness_isaacgym_noise0.1_pickplace_ckpt100_s1
transfer_real_gofa_pickplace_ckpt100_t0
```

If the evaluation includes key parameters such as noise level, perturbation type, task name, or checkpoint number, include them in `eval_id`.

## Required Files in Each eval Directory

Each `eval/<eval_id>/` directory should contain at least:

```text
eval_config.json
metrics.json
raw_results/
```

### eval_config.json

Records the full resolved evaluation parameters.

Example:

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
  "num_episodes": 100
}
```

### metrics.json

Records summary metrics used by leaderboards or result tables.

Example:

```json
{
  "success_rate": 0.82,
  "mean_return": 134.5,
  "mean_cost": 0.12,
  "violation_rate": 0.04,
  "num_episodes": 100
}
```

The structure of `metrics.json` should remain stable so that results can be aggregated automatically.

### raw_results/

Stores raw evaluation results, such as:

```text
raw_results/
  episodes.jsonl
  trajectories/
  logs/
  videos/
```

Raw results are used for debugging, inspection, plotting, and deeper analysis. The exact format does not need to be identical across all projects, but it should preserve enough information to reproduce the experiment conclusion.

## Evaluation Script Requirements

Each evaluation script should support one of the following ways to locate the target run.

Option 1: pass the full run directory:

```bash
python evaluate_icrl.py \
  --run_dir runs/icrl/20260612_1530_isaacgym_dp_pickplace_s0
```

Option 2: pass `run_id` and `runs_root`:

```bash
python evaluate_icrl.py \
  --run_id 20260612_1530_isaacgym_dp_pickplace_s0 \
  --runs_root runs/icrl
```

An evaluation script should:

1. locate the target run using `--run_dir` or `--run_id + --runs_root`;
2. read `run_config.json`;
3. generate a unique `eval_id` from evaluation parameters;
4. create `runs/<method>/<run_id>/eval/<eval_id>/`;
5. write `eval_config.json`;
6. write `metrics.json`;
7. save raw evaluation result files.

## Leaderboard Aggregation

A leaderboard or result aggregation script can recursively scan:

```text
runs/*/*/eval/*/metrics.json
```

Then combine:

```text
runs/<method>/<run_id>/run_config.json
runs/<method>/<run_id>/eval/<eval_id>/eval_config.json
runs/<method>/<run_id>/eval/<eval_id>/metrics.json
```

to generate a unified result table.

## Scope

This structure is suitable for:

- imitation learning;
- reinforcement learning;
- robot learning;
- VLA / policy learning;
- supervised learning;
- benchmark evaluation;
- ablation studies;
- simulator transfer;
- multi-seed experiment tracking;
- real-world validation.

It manages experiment run outputs. It does not replace dataset versioning, model release systems, or online service logging systems.
