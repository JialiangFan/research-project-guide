<!-- EVIDENCE.md 模板。放在 PR 分支根或 runs/<run-id>/ 下并在 PR body 链接。只写摘要 + 真实路径，完整日志留 runs/.../verification/。 -->

# EVIDENCE · <task-id> / CP-<N>

- Task：`docs/<project>/tasks/<task>.md`
- Checkpoint：CP-<N>（覆盖 AC-<N>, AC-<M>）
- 分支：`feat/<task-id>-cp<N>-<slug>`
- git commit：`<hash>`　dirty：`false`
- run-id：`runs/<run-id>/`

## 总验证

```bash
<总验证命令>
```

exit code：`0`

## 每条 AC 的覆盖

| AC | 判定条件（摘要） | Verifier | Negative test | 结果 | 证据 |
|----|------------------|----------|---------------|------|------|
| AC-<N> | <…> | VF-<N>（<类型>） | NEG-<N> | 通过 | `verification/VF-<N>.log` / `NEG-<N>.log` |
| AC-<M> | <…> | VF-<M> | NEG-<M> | 通过 | `verification/VF-<M>.log` / `NEG-<M>.log` |

> Negative test 断言 verifier 对坏输入失败，测试本身 exit 0；禁止 `assert True`。

## Artifact / 日志路径

```text
verification/manifest.json     # 元数据（git / 每条 VF·NEG 的 exit code / input·output）
verification/stdout.log
verification/stderr.log
artifacts/<...>                # 本次产出的真实文件
run_config.json                # 完整命令 / 环境 / 数据 hash（在 runs/，不抄进本文件）
```

## 可视化产物（重型实验/仿真默认项；轻量任务删除本节）

| 产物 | 路径 | 对应 AC | 说明 |
|------|------|---------|------|
| rollout 视频 | `visualizations/<rollout.mp4>` | AC-<N> | seed=<…>, frames=<…> |
| 指标曲线 | `visualizations/<curve.png>` | AC-<M> | <指标> |
