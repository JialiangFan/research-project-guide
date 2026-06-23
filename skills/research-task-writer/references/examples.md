# 范例：两份写到位的 task 文件

两个完整范例，覆盖轻重两端。照着感受"必填节怎么填、轻量任务怎么缩放、四要素长什么样"。两份都全中文。

- [范例一：重型代码 / 仿真任务](#范例一重型代码--仿真任务)
- [范例二：轻量论文修改任务](#范例二轻量论文修改任务)

---

## 范例一：重型代码 / 仿真任务

> 场景：为机器人仿真项目搭一个新场景。章节填满，含领域专属小节。提炼自一个真实的 MuJoCo 装箱场景设计任务。

````markdown
# 任务 01：MuJoCo 场景设计 — 方盒装箱（运输纸箱 → 零售展示盒）

## 状态

状态：已完成
最近更新：2026-06-23
一句话进展：装箱场景已落地（几何 + 资产 + builder + 导出/预览 + clean/fault eval）；正式 eval matrix 待登记到 runs/。

## 目标

把仿真场景从"桌面散放杂物 → 2×2 托盘"重设为更贴近真实的上架任务：机械臂从运输纸箱（source bin）逐个取出同款方盒，放进右侧零售展示盒（retail display）的前向槽位。场景仍须能 step physics、渲染 RGB-D，并保留原有 contract/scenario schema。

## 背景

原桌面散放场景与真实包装/上架任务差距大。参考图里"源容器 → 目标容器"的结构正好对应装箱任务，故据此重设。依赖既有 MuJoCo 基础设施（生成器、机器人/夹爪/物体加载、相机、RGB-D），服务于下游 contract 治理评测。

## 技术路线

### 方法

场景仍由 runtime generator 生成 MJCF，不手写独立场景文件；在同一 schema 上替换容器、资产池、slot layout 和故障解释。对象用 identical-SKU：N 个实例复用同一 HOPE 方盒 mesh，curated 为 `box_like`，顺带制造实例歧义作为感知压力源。

### 输入

- scenario / contract schema；
- 主对象 = HOPE `obj_000014` 方盒 mesh，`mesh_scale=(0.001, 0.001, 0.001)`，curated category = `box_like`；
- seed、`n_targets`、`n_resident`、fault types；
- table / 机器人 / 夹爪 / 相机 baseline。

### 输出

- 含 source bin、display 与方盒对象的 MJCF 场景；
- `carton_<tier>` / `carton_<fault>` 命名场景；
- standalone `.xml` + `manifest.json` + 预览图；
- contract trace 与执行记录。

## 实现计划

| 模块 / 文件 | 作用 | 状态 |
|---|---|---|
| `src/contractguard/pgtray/scenario.py` | carton 资产、`carton_pack_scenario` builder、难度梯度、bin grid、display slots | 已完成 |
| `src/contractguard/pgtray/mujoco_source.py` | 渲染 source bin / retail display fixtures，抑制 legacy tray | 已完成 |
| `scripts/export_pgtray_scenes.py` | 导出 standalone 场景 + manifest + 预览 | 已完成（复用，未改） |

## 指标

- `scene_compile_success`：命名场景能否被 MuJoCo compile；
- `interpenetration_count`：对象与桌面/bin/display 的初始穿插数量；
- `success_rate`：成功 episode 数 / 总 episode 数；
- `carton_orientation_valid_rate`：upright + front-facing postcondition 通过比例。

## 交付物

> 状态为「已完成」时必填。

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| 场景生成器 | `src/contractguard/pgtray/scenario.py` | 已完成 | 资产/builder/梯度/grid/slots 均接入 |
| fixture 渲染 | `src/contractguard/pgtray/mujoco_source.py` | 已完成 | source bin / display 渲染 |
| standalone 场景 | `configs/scenarios/.../scenes/*.xml` | 已完成 | 6 个 carton 场景 + manifest + 预览 PNG |
| 单测 | `tests/test_pgtray_carton.py` | Partial | 7 个非渲染测试通过；render 测试需 EGL device |
| carton eval 证据 | `/tmp/.../carton_eval_summary.json` | 临时证据 | clean tiers `success_rate=1.0`；尚未登记到正式 `runs/` |

产出命令（生成场景与预览）：

```bash
python scripts/export_pgtray_scenes.py --catalog carton
```

验证命令（编译校验 + 看预览）：

```bash
python -m contractguard.pgtray.validate --scene configs/scenarios/.../scenes/pgtray_v2_carton_medium.xml
open configs/scenarios/.../scenes/previews/pgtray_v2_carton_medium.png
```

## 复现入口

- 发起新 run：见 `../运行命令.md`
- 完整命令 / 代码版本 / 环境 / 数据：见对应 `runs/.../run_config.json`
- 结果登记：见 `../实验结果.md`

## 验收标准

- [x] source bin（4 壁开口顶）+ display（后墙/前沿/槽位）在 MJCF compile 且无穿插
- [x] N 个 identical-SKU 方盒竖立码进 bin，rest 在底面、不沉桌、不互穿
- [x] 难度梯度 `carton_min/light/medium/heavy`（4/8/12/24）可在 pipeline 跑
- [x] clean episode `success_rate=1.0`、`external_execution_error_total=0`
- [x] 故障场景 `carton_missing_detection` 端到端跑通：不成功 + 触发 recovery、无 execution error
- [x] 每个命名场景导出 standalone `.xml` + manifest + 预览图

## 阻塞与边界

- 已有底座：MuJoCo 生成器、机器人/夹爪/物体加载、overhead/third_view/hero 相机、RGB-D、deterministic handoff、难度梯度框架、standalone 导出。
- 本次落地：source bin / display fixtures、identical-SKU carton 资产、`carton_pack_scenario` builder、命名场景与故障、导出 + 预览，全部经 compile-verify + clean/fault eval。
- 仍未做：多 seed/多 shield 的 eval matrix 跑批与 fault attribution 统计、force-level no-crush gate（需替换 deterministic handoff）、真实贴图。

## 备注

- 决策 18（运行时生成器 = 单一真相 + 导出快照 + 难度梯度）继续成立，carton 场景沿用同一框架。
- 偏离：本想用牙膏盒比例的 `obj_000002`，实际改用 HOPE `obj_000014` 方盒，原因是需求要更接近正方形的对象。旧牙膏/primitive box 表述只作（历史记录），不是当前场景真相。
````

---

## 范例二：轻量论文修改任务

> 场景：回应 reviewer 对某指标的质疑，改论文若干处。没有 runs/，指标节省略，技术路线压成"思路 / 依据 / 产出"。注意：即便原始 reviewer 意见和公式是英文，叙述正文仍全中文。

````markdown
# 任务 04：问题定义与攻击指标契约

## 状态

状态：进行中
最近更新：2026-06-23
一句话进展：已定稿改写方案，待落到 AAAI draft 的问题定义、指标小节与 Algorithm 1。

## 目标

修正 reviewer 2010 的最高风险质疑：现稿用 `V_c(s + delta) > V_c(s)` 这类表达，会被读成"攻击直接改物理状态"。要把攻击重新定义为**观测攻击**，并把"攻击有效性"绑定到真实环境代价，消除"改物理态"的歧义。

## 背景

reviewer 2010 指出攻击指标疑似用 `s + delta` 替换物理状态。这是可能导致拒稿的最高风险点：若指标与物理过程不符，结论站不住。该任务服务于 AAAI rebuttal。

## 技术路线

- 思路：把攻击建模为观测攻击 `a_t = pi_E(o_t + delta_t)`，环境转移仍为 `s_{t+1} = f(s_t, a_t)`；有效性用真实环境代价 `sum_t c(s_t, a_t, s_{t+1})` 度量，而非在被扰物理态上的学习代价。全程区分三个对象：受害观测、物理环境状态、学习到的代理状态。
- 依据：reviewer 2010 意见；现稿问题定义与指标小节。
- 产出形态：改写后的问题定义、指标小节、Algorithm 1 文本，加一个符号表。

## 实现计划

要改的章节 / 文件：

| 位置 | 改动 | 状态 |
|---|---|---|
| AAAI draft §问题定义 | 攻击改为观测攻击，转移式不变 | 待办 |
| AAAI draft §攻击指标 | 有效性绑定真实环境代价，学习约束分仅作攻击目标 | 待办 |
| AAAI draft Algorithm 1 + pipeline 文本 | 扰动在代理模型上优化，仅经真实 rollout 评测 | 待办 |
| 符号表 | 新增 `s_t, o_t, delta_t, a_t, psi, c` 定义 | 待办 |

## 交付物

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| 改后问题定义与指标小节 | AAAI draft（Overleaf）| 待办 | 消除"改物理态"歧义 |
| 改后 Algorithm 1 与 pipeline 文本 | AAAI draft | 待办 | 扰动优化与评测解耦 |
| 修正评测器实现 checklist | `docs/aaai_rebuttal/tasks/04_*.md` 本文 | 待办 | 供工程落地核对 |

## 复现入口

无 runs/。如何核对产物：改动落在 AAAI draft 的问题定义、指标、Algorithm 1 三处；逐条对照下面验收标准检查。

## 验收标准

- [ ] 正文无任何方程暗示攻击者直接改物理状态
- [ ] 学习到的约束分被描述为攻击目标，而非最终评测指标
- [ ] 每个上报的攻击结果都绑定真实环境代价、violation rate、return

## 备注

这是最高风险的 AAAI blocker；其余 reviewer 质疑围绕它展开，优先处理。
````
