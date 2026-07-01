# 闭环细则

SKILL.md 讲流程，这里放可直接抄的命令与结构，以及边界处置。

## 前置校验

```bash
git remote -v                 # 必须有 GitHub remote；无则先 gh repo create / git remote add
gh auth status                # gh 已登录
git -C <repo> status --porcelain   # main 是否干净
```

## 隔离工作区（git worktree）

```bash
# 在主仓库里，从最新 main 切分支并建独立工作目录
git -C <repo> fetch origin
git -C <repo> worktree add -b feat/<task-id>-cp<N>-<slug> \
    ../<repo>-wt-<task-id>-cp<N> origin/main
```

- 实现者、后续修订都在这个 worktree 目录里操作，主工作区不受影响。
- 收尾（用户 merge 后）清理：
  ```bash
  git -C <repo> worktree remove ../<repo>-wt-<task-id>-cp<N>
  git -C <repo> branch -d feat/<task-id>-cp<N>-<slug>   # 已合并才用 -d
  ```

## 提 PR

```bash
git -C <worktree> add -A && git -C <worktree> commit -m "<feat(task03): CP-2 …>"
git -C <worktree> push -u origin feat/<task-id>-cp<N>-<slug>
gh pr create --repo <owner/repo> --base main \
    --head feat/<task-id>-cp<N>-<slug> \
    --title "<feat(task03): CP-2 metric + invariant 校验>" \
    --body-file <填好的 pr_template.md>
```

## 取证（reviewer 只读，不进主对话大文件）

```bash
gh pr diff <PR#>                        # 看改了什么
git -C <worktree> log --oneline main..HEAD
git -C <worktree> show --stat HEAD
gh pr comment <PR#> --body "<finding>"  # findings 落 inline comment（可选）
```

## run_config.json（放 runs/<run-id>/，不抄进 task 文件）

```json
{
  "run_id": "<date>_<task>_cp<N>",
  "cp_id": "CP-<N>",
  "ac_ids": ["AC-<N>", "AC-<M>"],
  "git_commit": "<hash>",
  "git_branch": "feat/<task-id>-cp<N>-<slug>",
  "environment": { "python": "3.11", "<pkg>": "<ver>" },
  "command": "<总验证命令>",
  "input_data": { "<path>": "sha256:<hash>" },
  "output": { "<artifact>": "runs/<run-id>/artifacts/<...>" },
  "exit_code": 0
}
```

## verification/manifest.json（每条 VF/NEG 的审计追踪）

```json
{
  "run_id": "<...>",
  "cp_id": "CP-<N>",
  "git_commit": "<hash>",
  "git_dirty": false,
  "verifiers": {
    "VF-<N>": { "ac": "AC-<N>", "type": "golden", "command": "<...>", "exit_code": 0, "log_path": "verification/VF-<N>.log" }
  },
  "negative_tests": {
    "NEG-<N>": { "vf": "VF-<N>", "command": "<...>", "exit_code": 0, "log_path": "verification/NEG-<N>.log" }
  },
  "artifacts": { "<name>": "artifacts/<...>" },
  "visualizations": { "rollout": { "path": "visualizations/<...>.mp4", "ac": "AC-<N>", "seed": 42 } },
  "summary": { "ac_coverage": { "AC-<N>": "PASS", "AC-<M>": "PASS" } }
}
```

> 三层证明：run_config 的**总验证命令 + exit code**（复现）→ `VF-*.log` / `NEG-*.log`（定位）→ `manifest.json`（审计）。

## review → 修 → verify 循环的终止条件

- **收敛**：没有 P0/P1 且总验证一次全绿 exit 0 → 进阶段 6 人工确认闸门。
- **不收敛**：同一分歧反复拉锯超过 2~3 轮（reviewer 坚持 P0、实现者坚持已满足）→ 停下，把分歧点、双方证据、你的判断汇报给用户裁决，不无限循环。
- **越界**：实现中发现超出 Non-goals 的事 → 记入 task 的「开放问题」，不在本 PR 顺手做。

## 常见边界

| 情况 | 处置 |
|------|------|
| task 文件缺 AC/VF/NEG | 先建议用 `research-task-writer` 补齐，或当场和用户敲定本次 PR 的验收 + verifier |
| 项目无 GitHub remote | 停下提示 `gh repo create` / 加 remote（本 skill 一律 gh PR） |
| 随机实验（RL / 采样 / LLM eval） | 不要求 hash 一致；按 task 的 seed 列表 / 样本量 / CI / tolerance / pass-fail 判据验证 |
| VF 通过但 NEG 失败 | verifier 本身可能是摆设——回到 task writer 讨论 verifier 设计，别放行 |
| 一个 CP 太大 | 提醒用户拆成多个 CP / 多个 PR，保持一个 PR 一个功能闭环 |
