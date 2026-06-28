# 分节写作指引：每一节怎么写到位

下笔拿不准某节怎么写时读这里。每节给"职责 / 好的样子 / 常见反例"。范例多提炼自两类真实任务：一个重型仿真任务（MuJoCo 装箱场景）和一个重型评测任务（safety evaluator）。

## 目录

- [总原则](#总原则)
- [状态](#状态)
- [目标](#目标)
- [背景](#背景)
- [技术路线](#技术路线)
- [数据契约与不变量](#数据契约与不变量)
- [实现计划](#实现计划)
- [指标与统计验证](#指标与统计验证)
- [验收标准](#验收标准)
- [Verifier 设计](#verifier-设计)
- [Verifier 反例测试](#verifier-反例测试)
- [AC / CP / Verifier 覆盖矩阵](#ac--cp--verifier-覆盖矩阵)
- [Checkpoint 切片](#checkpoint-切片)
- [交付物](#交付物)
- [复现入口](#复现入口)
- [阻塞与边界](#阻塞与边界)
- [执行约束](#执行约束)
- [完成证据包](#完成证据包)
- [备注](#备注)

---

## 总原则

- **每一节都要有信息量。** 只剩套话的节就精简、合并或删掉。宁可短而实，不要长而空。
- **能表格就表格。** 交付物、代码位置、覆盖矩阵这类多字段信息，表格比长段落好读、好维护。
- **写给三个月后的人和执行 agent。** 他不记得上下文，也没读过代码。术语第一次出现给一句解释，路径写全，编号（AC/CP/VF/NEG）保持一致。
- **真实优先。** 路径、命令、指标只写真实存在或确定的；不确定就 `TBD：待确认` 并说明缺什么。
- **先验后做。** 重型任务先写 AC、verifier、反例测试，再写实现计划——不要先写实现再临时补验收。

---

## 状态

**职责**：让人 3 秒看出这个任务现在能不能用、新不新。

**好的样子**：

```
状态：已完成
最近更新：2026-06-23
一句话进展：evaluator 已落地（schema+metric+invariant+smoke）；VF-1~VF-4 与 NEG-1~NEG-3 全绿，证据见 runs/2026-06-23_task03/verification/。
```

一句话进展点出"做完了什么 + 还差什么 + 证据在哪"，比单一个"已完成"信息量大得多。

**反例**：只写 `状态：进行中`，没有日期、没有进展句、没有证据入口——读者无法判断是昨天还是三个月前停在哪。

---

## 目标

**职责**：用一两句说清"做什么、达成什么、边界在哪"。

**好的样子**：先说要做的事，再说不变约束。例：
> 把场景从"桌面散放杂物 → 2×2 托盘"重设为"从运输纸箱逐个取同款方盒 → 放进零售展示盒"的上架任务；场景仍须能 step physics、渲染 RGB-D，并保留原有 contract/scenario schema。

同时给了**正向目标**和**不变约束**，边界很清楚。

**反例**："优化场景，让它更好。"——什么叫更好？没有可判断的边界。

---

## 背景

**职责**：解释为什么需要这个任务、它在项目里的位置（上游依赖、下游服务）。

**好的样子**：点明动机和依赖关系，必要时附参考图/直觉来源。轻量任务一两句即可。

**反例**：把整个项目的来龙去脉复述一遍。背景只服务"理解这个子任务为什么存在"，不是项目综述。

---

## 技术路线

**职责**：方法 + 输入 + 输出 + 关键取舍。技术读者最关心的一节。

**好的样子**：
- `方法`：不只说"用 MuJoCo"，还说**怎么用**和**为什么**——"场景仍由 runtime generator 生成 MJCF，不手写独立场景文件；在同一 schema 上替换容器、资产池、slot layout"。带上关键设计取舍和被否的替代方案。
- `输入`：列具体——schema 名、mesh 编号、缩放系数、seed、baseline 资产。
- `输出`：列具体产物形态——生成的 MJCF、命名场景、standalone xml + manifest + 预览图、trace。

**反例**：方法只写一个框架名（"用 MuJoCo / 用 PyTorch"），没有"怎么用、为什么这么选"。框架名不是技术路线。

---

## 数据契约与不变量

**职责**：固定输入/输出 schema、关键定义、不变量，防止实现时被脑补或悄悄改语义。代码/实验/数据/评测任务必填。

**好的样子**：
- 输入/输出契约逐字段写清：必填字段、类型、单位、ID 规则、split 规则、缺失/非法样本处理、是否允许 skip、dropped report。
- 关键定义写成**可判定公式**，不是模糊描述：`unsafe_success = success == true and has_safety_violation == true`。
- 关键 metric 配**手算 golden**（G-1）：列最小输入 + 逐步手算 + 期望数值 + tolerance。
- 不变量分通用 + 领域专用。ML/eval 任务必写 split 互斥、no-leakage、不用 test 调参、不静默移除失败 episode；robotics/sim 任务必写 pose/unit/frame、limit 不被绕过、不读 privileged state。

**反例**：
- 只写"输入 episodes，输出 metrics"，不写字段、不写缺失处理——实现会自由发挥。
- metric 只有文字定义没有手算例子——算错了也看不出来。
- 不变量只写 count 守恒，漏掉 split 泄漏——评测结论可能整体作废。

---

## 实现计划

**职责**：代码实现在哪里、改/增什么、对应哪条 AC。让读者能直接跳到对应文件。

**好的样子**：表格列**真实文件路径 + 作用 + 对应 AC + 状态**：

| 模块 / 文件 | 作用 | 对应 AC | 状态 |
|---|---|---|---|
| `src/contractguard/pgtray/scenario.py` | carton 资产、场景 builder、难度梯度 | AC-1 | 已完成 |
| `scripts/run_full_verifier.py` | 总验证入口，跑全部 VF/NEG | AC-1~AC-4 | 待办 |

**反例**：只写"实现场景生成逻辑"，不给文件路径，也不绑 AC。路径写错或编造比不写更糟——会把人引向不存在的文件。

---

## 指标与统计验证

**职责**：列指标定义保证跨 run 可比，并按"确定性 / 统计型"两类分别写验证方式。实验/评测任务必填。

**好的样子**：
- 指标表给出 ID、公式、分母分子、对应 AC：`M-1: success_rate = success_count / total_valid_episode_count`，并写明分母含 timeout/failure、不静默移除。
- 确定性指标（metric function/schema/golden）：精确比对 + tolerance。
- 统计型指标（RL/sampling/LLM eval）：写 seed 列表、每 seed episode 数、CI/bootstrap、tolerance、pass/fail 判据、flaky 与重跑规则。例："5 seeds × 100 episodes，mean ± 95% CI；method 比 baseline 高 3% 且 CI 不跨 0 视为 regression"。

**反例**：
- 只写指标名不写定义（"看 success_rate 和 accuracy"）——不同人算法不同，不可比。
- 对随机实验要求"固定 seed 后 hash 完全一致"——要么误杀 flaky，要么形同虚设。纯写作类没有量化指标，整节省略。

---

## 验收标准

**职责**：可勾选、可客观判断的完成条件，每条用 AC-ID 编号并绑 verifier / 反例 / 证据。

**好的样子**：每条都能被明确判定真/假，且挂上 VF/NEG/Evidence：

```
- [ ] AC-3：rule_breakdown.csv 含每条 rule 的 violation count，且总和等于 per-episode report 的 violation event 总数。
  - 验证：VF-3
  - 反例：NEG-3
  - 证据：完成后指向 runs/.../verification/
```

**反例**："场景做得不错""效果满意""代码工作正常"——无法客观判定，永远扯皮。验收标准是闸门，必须可证伪、且有 verifier 兜底。

---

## Verifier 设计

**职责**：把"怎么验"写成可执行、分层的命令，每个 VF 覆盖明确的 AC。代码/实验/数据/评测任务必填。

**好的样子**：
- 表格列 VF-ID、覆盖 AC、类型、命令、期望结果、日志路径。类型至少覆盖 schema/unit/golden/invariant/smoke 中若干层，别只有"脚本 exit 0"。
- 提供**一条总验证命令**（如 `python scripts/run_full_verifier.py --task 03`），失败 exit 非 0，输出每个 AC 覆盖状态，日志落 `runs/.../verification/`。
- 每条 VF 注明：检查对象、期望输出、失败后看哪个日志、哪些失败阻塞、是否允许更新 golden（及条件）。

**反例**：只写"用 pytest 验证"，不分层、不绑 AC、没有总入口——无法判断覆盖是否完整，也无法一键复验。

---

## Verifier 反例测试

**职责**：证明 verifier 自己不是摆设——对坏输入会失败。每个关键 verifier 必须有反例测试（硬要求）。

**好的样子**：
- 表格列 NEG-ID、对应 VF、覆盖 AC、注入什么错误、期望 verifier 行为、命令。例：NEG-1 删除 required field → schema verifier 必须 exit 非 0。
- 反例测试本身在测试框架里"通过"——它断言 verifier 对坏输入返回非零 / 抛异常：

```python
def test_schema_verifier_rejects_missing_required_field(tmp_path):
    bad = tmp_path / "metrics_bad.json"
    bad.write_text('{"success_rate": 0.5}')
    r = subprocess.run(["python", "scripts/validate_metrics_schema.py", str(bad)],
                       capture_output=True, text=True)
    assert r.returncode != 0
    assert "missing" in r.stderr.lower() or "required" in r.stderr.lower()
```

**反例**：
- `def test_negative(): assert True`——什么都没证明。
- 只写"人工确认 verifier 能失败"，没有可执行断言。
- 把 corrupted output 丢进正常 artifact 目录冒充结果。

---

## AC / CP / Verifier 覆盖矩阵

**职责**：把 AC、checkpoint、verifier、反例、证据的对应关系摊在一张表上，堵住覆盖缺口。必填。

**好的样子**：表格每行一条 AC，列出 Checkpoint / Verifier / Negative Test / Evidence / 状态。一眼能看出"AC-5 没有 verifier"或"VF-2 没有反例"。只能人工审查的 AC 用 `MANUAL-N` 并注明审查对象、标准、证据位置。

**反例**：AC 写一处、verifier 写另一处、checkpoint 又写一处，互不引用——结果 AC 有 10 条 verifier 只覆盖 6 条，没人发现。

---

## Checkpoint 切片

**职责**：把任务拆成 2–5 个有序纵切片防漂移，每片引用 AC-ID + 报到点。

**好的样子**：表格列 CP-ID、覆盖 AC、目标、产物、报到点、Verifier、状态。每片是纵切片——"一个小输入 → 一段核心实现 → 一个输出 → 一个 verifier → 一个 evidence"。例：CP-1 完成 schema+parser，报到点提交 VF-1 日志。

**反例**：横向分工（"先写全部代码 → 再写全部测试 → 最后写文档"）——一口气闷头跑，跑偏了也没有中途可验证的产物。

---

## 交付物

**职责**：交付物表（含真实路径、对应 AC）+ 能跑的产出命令 + 对应 VF 的验证命令。"完成"长什么样在这里定死。

**好的样子**：表格列 `交付物 | 路径 | 对应 AC | 状态 | 产出命令 | 验证命令 | 说明`，诚实标注半成品（`已完成 / Partial / 临时证据`）比一律写"已完成"可信得多。已完成时路径必须指向 `runs/` 真实 artifact。命令分产出（生成）和验证（核对，对应 VF-ID）两类。

**反例**：
- 交付物只写"代码和结果"，无路径、无状态、无 AC。
- 把每次 run 的精确命令行、git commit、env 抄进来——这些属于 `run_config.json`，抄进 task 文件必然过期失同步。

---

## 复现入口

**职责**：把读者导向真正的复现真相，而不是在 task 文件里重抄一遍。

**好的样子**：
> - 发起新 run：见 `../运行命令.md`
> - 完整命令 / 代码版本 / 环境 / 数据：见 `runs/.../run_config.json`
> - 验证证据：见 `runs/.../verification/manifest.json`
> - 结果登记：见 `../实验结果.md`

**反例**：在 task 文件里贴一大段"复现步骤"，包含写死的 git commit、conda 环境、数据 hash。这些一换 run 就错；它们的家在 `run_config.json`。

**写作类变体**：没有 runs/ 时改成"核对入口"——修改文件、diff 命令、reviewer comment 对应位置、人工审查记录。

---

## 阻塞与边界

**职责**：把"已有的 / 这次做的 / 还没做的"分开，并显式划出 Non-goals 栅栏与越界处置。

**好的样子**：
> - 已有底座：MuJoCo 生成器、机器人/夹爪/物体加载、RGB-D、导出。
> - 本次落地：source bin / display fixtures、carton 资产、命名场景与故障、导出+预览。
> - 仍未做：多 seed eval matrix、force-level no-crush gate、真实贴图。
> - Non-goals：不改 deterministic handoff；不引入新依赖；不顺手重构导出脚本。
> - 越界处置：发现栅栏外但有价值的事 → 记到 `## 开放问题`，不动手做。

读者一眼知道能用什么、不能指望什么、越界怎么办。

**反例**：只写"还有些问题没解决"，没有 Non-goals——agent 容易自由发挥、过度工程、把不该改的改了。

---

## 执行约束

**职责**：写明禁止行为，并规定派活时必须注入 execution contract。LLM/agent 执行任务必填。

**好的样子**：
- 派活注入清单：task 文件 + execution_contract + 当前 CP/AC/VF/NEG + 禁止行为 + evidence 格式 + 报告格式。
- 禁止行为具体到可判定：不改测试/golden 来过验收、不硬编码答案、不 silent skip、不混 unrelated refactor、不 mock 冒充真实结果、未运行不得声称通过、只跑部分 verifier 不得声称全绿、无证据不得标已完成。
- 派活别只说"完成 task 03"，要说"完成 CP-2（覆盖 AC-3/AC-4），遵守 execution_contract，跑 VF-3/VF-4/NEG-3，存 evidence bundle"。

**反例**：把约束只埋在 task.md 中段，派活时一句"帮我做完 task 03"——agent 根本没读到约束，照样假完成。

---

## 完成证据包

**职责**：任务标"已完成"时给出 evidence bundle 摘要，指向 runs/ 原始日志。

**好的样子**：摘要写总验证命令 + exit code + manifest/stdout/stderr/artifact 路径 + git commit/dirty + 每条 AC 的覆盖通过情况；完整日志放 `runs/.../verification/`。只有满足硬条件（全部 AC 有 verifier、关键 verifier 有反例、干净环境一次全绿、证据齐全）才能标"已完成"。

**反例**："状态：已完成。测试已通过。"——没有命令、没有 exit code、没有日志路径，无法复查，等于无证据谎报完成。

---

## 备注

**职责**：关联决策、设计取舍、偏离计划之处。维护任务记忆。

**好的样子**：
- 记关联决策："决策 18（运行时生成器=单一真相）继续成立"。
- 记偏离："本想用牙膏盒 mesh，实际改用 HOPE `obj_000014` 方盒，原因是需求要更接近正方形"。
- 过时内容用「（历史记录）」标注保留，不直接删——保住决策轨迹，别让后人重踩。

**反例**：把所有变更直接覆盖删除，文件只剩"最终态"。后人不知道为什么是这样、之前试过什么、为什么放弃。
