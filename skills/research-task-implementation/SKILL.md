---
name: research-task-implementation
description: >-
  执行 research-project 的 task 文件（docs/<project>/tasks/ 下由 research-task-writer 产出的执行契约）里的一个
  可验收功能单元——默认对应一个 Checkpoint（CP-N）纵切片——并强制走「实现 → gh PR + EVIDENCE.md → 独立 agent
  review → 修 P0/P1 → verifier 全绿 → 人工确认 → 人工 merge」的闭环。为每条 AC 绑一个 verifier(正例)、为每个关键
  verifier 加 negative/mutation test(反例)，没有证据不算完成，绝不自动 merge 到 main。触发条件：用户说「执行/实现这个
  task」「实现 task 03 的 CP-2」「把这个功能提个 PR 走 review 闭环」「跑 implementation 闭环」，或
  /research-task-implementation。也在用户明显想「按 task 文件把一个功能真正做出来并交证据 / 提 PR / 找另一个 agent
  review / 让 verifier 全绿」时触发，即使没说「implementation」四个字。

  不要用于：只想写/更新 task 文件（那是 research-task-writer）、只要概念性研究建议、单纯 debug，除非用户明确要按 task
  文件落地实现并走 PR + review + verifier 的交付闭环。
argument-hint: "[task-path | task-id]（可附 CP 编号，如 `task_03 CP-2`；空参=列出 docs/*/tasks/ 下的 task 让用户挑）"
user-invocable: true
---

# 研究项目 task 执行闭环

把一个 task 文件里**一个**可验收功能单元真正实现出来，并用证据证明它完成——而不是让 agent 说一句「done」。

本 skill 是 [research-task-writer](../research-task-writer/SKILL.md) 的**下游**：writer 写清「做什么、怎么验」（AC / CP / VF / NEG / 覆盖矩阵 / execution_contract），本 skill 负责「怎么做、交证据」。术语、编号系统、docs 规格 ↔ runs 复现单一真相分离，全部沿用 writer，不再重复定义。

## 三条不可让步的纪律

1. **一个 PR = 一个可验收功能单元**（默认一个 CP 纵切片）。不把多个功能、重构、bugfix 混进同一个 PR。功能闭环划分，不按代码行数划分。
2. **没有 evidence 的完成不算完成**。每条 AC 绑一个 verifier；每个关键 verifier 至少一个 positive + 一个 negative/mutation test。总验证命令一次全绿 exit 0，证据落 `runs/.../verification/`。
3. **绝不自动 merge，绝不直改 main**。全程在 feature branch / worktree 上做；停在「人工确认 evidence」闸门，由用户自己 merge。实现者不 review 自己的 PR（运动员不当裁判）。

---

## 第 0 步（必做）：交互闸门

动手前必须和用户敲定三件事，缺一不往下走：

1. **实现哪个功能单元** —— 哪个 task、哪个 CP。默认「一个 CP → 一个 PR」；task 没有 CP（轻量任务）则整个 task 作为一个 PR。若用户想一次做多个 CP，提醒这会让 PR 变粗、review 变泛、回滚变难，请其确认或拆分。
2. **reviewer 后端**（阶段 4 用）——二选一：
   - **后端 A · 本会话 Claude 自动闭环**：派一个与实现者**隔离 context** 的 Claude reviewer subagent 出 findings，自动派回修订。最省事，但 reviewer 也是 Claude（同模型自审）。
   - **后端 B · Codex 交接包**：生成一份 review 交接文件，你拿去 Codex（或另一个会话）做真正跨工具的独立 review，把 findings 贴回。符合「Claude 主实现 / Codex reviewer」，但需人来回搬。
3. **项目有 GitHub remote** —— 本 skill 一律走 `gh PR`。`git remote -v` 无 remote 时，先让用户 `gh repo create` 或加 remote，别退化成本地流程。

用户说不清完成口径时，回到 task 文件的 AC；task 文件缺 AC/VF/NEG（不是 writer 规范产出）时，先建议用 `research-task-writer` 补齐，或当场和用户敲定本次 PR 的验收 + verifier，再往下走。

---

## 阶段 1 · 解析 task 与执行契约

- 定位 `docs/<project>/tasks/<task>.md`（空参时 `rg --files docs/*/tasks/` 或 glob 列出让用户挑）。
- 从中提取本次 PR 需要的规格（**不抄进别处，只在内存里用来派活**）：
  - 目标、Non-goals、越界处置
  - 目标 CP 覆盖的 AC 列表，及每条 AC 对应的 VF / NEG
  - 「执行约束 / execution_contract」节的禁止行为清单、evidence 格式、报告格式
  - 覆盖矩阵当前状态（哪些 AC 已通过、哪些 TBD）、数据契约与不变量、指标定义与统计验证规则
- 判定层级：轻量（grep + 人工审查即可）还是重型（有数据契约 / verifier / 反例）。缩放后续动作，别给论文修改任务硬套完整 verifier 机制。
- 校验真实性：task 里引用的路径 / 脚本是否真实存在。缺失就在派活 brief 里标 `TBD`，不编造。

---

## 阶段 2 · 建隔离工作区 + 派实现者

1. **隔离工作区**：从最新 main 切 `feat/<task-id>-<cp-id>-<slug>`，用 `git worktree add` 建独立工作目录（借 agent-review 的 worktree 思路，便于并行 / 回滚，绝不在 main 上直接改）。命令细节见 `references/workflow.md`。
2. **派实现者**：用 `Agent()` 派一个 implementer subagent，注入 `assets/implementer_brief.md` 填好的 brief——**task 文件相关规格 + execution_contract + 目标 CP/AC/VF/NEG + evidence 格式 + PR 模板 + 禁止行为**。实现者职责：
   - 只实现该 CP 的纵切片：小输入 → 核心实现 → 输出（不横向铺开、不混无关改动）。
   - 为每条 AC 落一个 verifier（VF-N）；为每个关键 verifier 写 negative/mutation test（NEG-N，注入错误 → 断言 verifier 失败，**禁止 `assert True`**）。
   - 跑总验证命令，把 stdout/stderr/manifest/artifact 落到 `runs/<run-id>/verification/`，run 元信息落 `run_config.json`。
   - 重型实验/仿真若产生可观测行为（rollout 视频、渲染帧、指标曲线），把真实可视化路径一并登记。
   - 写 `EVIDENCE.md`（用 `assets/evidence_template.md`）。
- 实现者是**唯一**改代码的角色。同一 PR 的主导权只给它一个，不让 reviewer 直接动手改实现。

---

## 阶段 3 · 提 PR + EVIDENCE.md

- `gh pr create`，body 用 `assets/pr_template.md` 填好：Goal / Scope / Non-goals / Acceptance Criteria 勾选 / Verification（命令 + exit code + artifacts）/ Agent Review（待填）/ Merge Checklist。
- `EVIDENCE.md` 放在 PR 分支里（并在 PR body 链接），或落 `runs/.../verification/` 后在 PR 引用。内容：总验证命令 + exit code、每条 AC 的 VF+NEG 覆盖、artifact 与可视化路径、git commit / dirty 状态。
- PR 标题即功能单元名（如 `feat(task03): CP-2 metric + invariant 校验`）。一个 PR 只讲一件事。

---

## 阶段 4 · 独立 agent review

reviewer 必须**隔离 context**：只喂它 task 文件（相关 AC）+ PR diff + EVIDENCE.md，**不给实现者的自辩**。按第 0 步选的后端走，模板都在 `assets/reviewer_brief.md`：

- **后端 A（Claude 自动）**：`Agent()` 派隔离 reviewer subagent，复用 `agent-review` skill 的证据优先 / 只读纪律（用 `git -C <worktree> diff/log`、`gh pr diff` 取证，不改任何文件）。
- **后端 B（Codex 交接）**：把 `reviewer_brief.md` 填好（PR 链接 + diff 摘要 + task AC + EVIDENCE + review 清单 + 输出格式）写成文件，交给用户拿去 Codex，等 findings 贴回。

无论哪个后端，reviewer 必须回答这几条（用户原话 + verifier 纪律）：

- 这个 PR 是否真的满足 task 文件里对应 AC？
- 有没有 missing test？有没有没覆盖的 edge case？
- 有没有破坏旧行为（回归）？
- EVIDENCE.md 里的命令是否**足够**证明完成（不是「跑了」而是「证明对」）？
- 每条 AC 是否都绑了一个 VF，且关键 VF 有 negative/mutation test（不是 `assert True`）？

输出分级 findings：**P0（阻塞合并）/ P1（须修）/ P2（可延后）**。findings 可选用 `gh pr comment` 落成 inline comment。

---

## 阶段 5 · 修 P0/P1 + verifier 全绿

- 把 findings 派回**实现者** subagent（同一 worktree 起新一轮）修 P0/P1；P2 记 backlog（写进 PR 或 task 的开放问题，不静默丢弃）。
- 重跑总验证命令，要求**一次全绿 exit 0**；negative test 在测试框架里通过（断言 verifier 对坏输入失败）。
- 同步更新 `EVIDENCE.md` 与 task 文件的覆盖矩阵（AC 状态、证据路径）。
- 循环 review → 修 → verify，直到**没有 P0/P1 且总验证全绿**。若 review 与实现反复拉锯超过 2~3 轮仍不收敛，停下汇报分歧点，交用户裁决，别无限循环。

---

## 阶段 6 · 人工确认闸门（不自动 merge）

停在这里，打印一份可让用户 3 秒决策的清单：

- PR 链接、分支名
- 每条 AC 的覆盖状态：`AC-N 通过（VF-N + NEG-N）` / 或仍 TBD
- 总验证命令 + exit code
- EVIDENCE 摘要 + artifact / 可视化产物真实路径
- Merge Checklist（全部 AC 满足 / 测试含 negative test / 无 unrelated 改动 / evidence 齐 / 独立 review 已过）

**明确交还控制权**：等用户人工确认 evidence 后自行 merge。本 skill 到此为止，不执行 merge。

可选（用户 merge 后再做）：把 task 文件状态回填「已完成」、覆盖矩阵状态与证据路径写实、清理 worktree（`git worktree remove`）。

---

## 反模式（绝对不做）

- ❌ 一个 PR 混多个功能 / 无关重构 / 顺手 bugfix。
- ❌ 实现者 review 自己的 PR，或 reviewer 直接动手改实现——运动员兼裁判。
- ❌ 无 EVIDENCE、verifier 未全绿、只跑了部分 verifier 就标「已完成」。
- ❌ 改 golden / 改测试来「过」验收、硬编码答案、silent skip 缺字段或失败样本、用 mock 冒充真实结果、没真跑就声称通过。
- ❌ negative test 写成 `assert True` 之类的空反例。
- ❌ 自动 merge 到 main、跳过人工确认、直接在 main 上改。
- ❌ 把整个 session transcript / 大文件读进主对话（取证用 `git -C`、`gh pr diff`、`jq` 精抽，别撑爆 context）。
- ❌ 派活只说「完成 task 03」——要说「完成 CP-2（覆盖 AC-3/AC-4），遵守 execution_contract，跑 VF-3/VF-4/NEG-3，交 evidence」。

---

## 交付前自检

- [ ] 一个 PR 只对应一个功能闭环（一个 CP），没有 unrelated 改动
- [ ] 每条 AC 绑了一个 VF，关键 VF 至少一个 negative/mutation test（非 `assert True`）
- [ ] 总验证命令一次全绿 exit 0，证据在 `runs/.../verification/`
- [ ] EVIDENCE.md 完整：命令 + exit code + artifact + 可视化（若有）+ git 状态
- [ ] 独立 reviewer（隔离 context 或 Codex）出过 findings，P0/P1 已修，P2 已登记
- [ ] task 文件覆盖矩阵 / 状态已同步
- [ ] 停在人工确认闸门，未自动 merge
