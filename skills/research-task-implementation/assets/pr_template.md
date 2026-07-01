<!-- gh PR body 模板。一个 PR 只讲一个可验收功能单元（通常一个 CP）。删掉不适用的行，别留占位符。 -->

## Goal

<这个 PR 交付哪个功能单元、解决 task 里的什么问题。一两句话。>

## Scope

- Task：`docs/<project>/tasks/<task>.md`
- Checkpoint：CP-<N>
- 覆盖 AC：AC-<N>, AC-<M>

## Non-goals

<这个 PR 明确不做什么：不改 X、不重构 Y、不评估 Z。越界的事记入 task 的「开放问题」，不在本 PR 顺手做。>

## Acceptance Criteria

- [ ] AC-<N>：<可客观判定的条件> — 验证 VF-<N> / 反例 NEG-<N>
- [ ] AC-<M>：<…> — 验证 VF-<M> / 反例 NEG-<M>

## Verification

Commands run（干净环境，一次全绿）：

```bash
<总验证命令，如 python scripts/run_full_verifier.py --task 03>
```

Exit code：

```text
0
```

Artifacts / Evidence：

```text
runs/<run-id>/verification/manifest.json
runs/<run-id>/verification/stdout.log
runs/<run-id>/artifacts/<...>
EVIDENCE.md
（重型实验/仿真若有可视化：runs/<run-id>/visualizations/<rollout.mp4 | curve.png>）
```

## Agent Review

Reviewer：

- [ ] Claude（本会话隔离 subagent）
- [ ] Codex（跨工具交接）

Findings：

- P0（阻塞合并）：<无 / …>
- P1（须修）：<无 / …>
- P2（可延后）：<无 / …，已登记到 task 开放问题>

## Merge Checklist

- [ ] 所有 AC 满足，逐条绑了 VF
- [ ] 关键 verifier 有 negative/mutation test（非 `assert True`）
- [ ] Evidence 齐全（命令 + exit 0 + artifact + git 状态）
- [ ] 无 unrelated 改动（refactor / 无关 bugfix）
- [ ] 独立 agent review 已过，P0/P1 已修
- [ ] 人工已确认 evidence
