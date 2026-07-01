# research-task-implementation

指导 AI 编码 agent（Claude Code / Codex）把 `docs/<project>/tasks/` 下的一个 task 文件里**一个可验收功能单元**真正实现出来，并强制走「实现 → gh PR + EVIDENCE.md → 独立 agent review → 修 P0/P1 → verifier 全绿 → 人工确认 → 人工 merge」的闭环。

它是 [`research-task-writer`](../research-task-writer/) 的**下游**：writer 写清「做什么、怎么验」，本 skill 负责「怎么做、交证据」。术语与编号（AC / CP / VF / NEG / 覆盖矩阵 / execution_contract / evidence bundle）、docs 规格 ↔ runs 复现单一真相分离，全部沿用 writer。

本目录是 skill 的**单一真相**（git 跟踪）。Claude Code 与 Codex 都通过软链引用这里的文件，不复制。

## 结构

```
research-task-implementation/
├── SKILL.md                      # 主流程（第0步交互闸门 + 6 阶段 + 反模式 + 自检）
├── assets/
│   ├── pr_template.md            # gh PR body 模板
│   ├── evidence_template.md      # EVIDENCE.md 模板
│   ├── implementer_brief.md      # 派实现者 subagent 的 prompt 模板（execution_contract 注入）
│   └── reviewer_brief.md         # 派 reviewer subagent / Codex 交接包模板（P0/P1/P2）
├── references/
│   └── workflow.md               # 闭环细则：gh/git worktree 命令、run_config/manifest 结构、循环终止条件
└── README.md                     # 本文件
```

## 安装

源文件在本目录（单一真相）。Claude Code 与 Codex 各通过一条软链指向这里——改一处，两边生效。

**在 `research-project-guide` 仓库根目录下**执行：

```bash
ln -s "$(pwd)/skills/research-task-implementation" ~/.claude/skills/research-task-implementation
ln -s "$(pwd)/skills/research-task-implementation" ~/.codex/skills/research-task-implementation
```

> skill 在**新会话**启动时被发现；已开的会话需重启才会加载。在 jfan 本机两条软链已建好。

## 前置依赖

- **`gh` CLI + GitHub remote**：本 skill 一律走 `gh PR`；无 remote 时先 `gh repo create` 或加 remote。
- **`research-task-writer` 产出的 task 文件**：本 skill 消费它的 AC / CP / VF / NEG / execution_contract。task 文件缺这些时，先用 writer 补齐。

## 维护

改 skill 直接改本目录文件，两条软链自动生效。本目录在 `research-project-guide` 仓库内，另在 `JialiangFan/skills` 仓库的 `claude/`、`codex/` 各存一份镜像（`diff -rq` 应无差异）。
