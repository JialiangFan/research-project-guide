# research-task-writer

指导 AI 编码 agent（Codex / Claude Code）为研究项目写出清晰、可维护、可阅读的**全中文** task 文件，统一放在 `docs/<project>/tasks/` 下。

本目录是 skill 的**单一真相**（git 跟踪）。Claude Code 与 Codex 都引用这里的文件，不复制。

## 结构

```
research-task-writer/
├── SKILL.md                 # 主指南（四原则 / 工作流程 / 章节清单 / 四要素落点 / 自检清单）
├── assets/
│   └── task_template.md     # 可直接复制去填的全中文模板
├── references/
│   ├── section_guide.md     # 每节"好 vs 差"对照、反例
│   └── examples.md          # 两个完整范例（重型代码任务 + 轻量论文修改任务）
└── README.md                # 本文件（接入说明）
```

## 安装

源文件在本目录（仓库内，单一真相）。Claude Code 与 Codex 各通过一条软链指向这里——改一处，两边生效。Codex 用的是与 Claude Code 相同的 `SKILL.md` 机制（`~/.codex/skills/<name>/SKILL.md`），故对称安装。

**在仓库根目录下**执行下面的命令（用 `$(pwd)` 展开成绝对路径，不依赖具体克隆位置）：

```bash
# Claude Code（全局发现，任意项目都能触发）
ln -s "$(pwd)/skills/research-task-writer" ~/.claude/skills/research-task-writer

# Codex（同样的 SKILL.md 机制）
ln -s "$(pwd)/skills/research-task-writer" ~/.codex/skills/research-task-writer
```

> skill 在**新会话**启动时被发现；已开的会话需重启才会加载。在 jfan 本机两条软链已建好。

验证：在任一项目里让 Claude 或 Codex 写/更新 task 文件，确认触发 `research-task-writer` 并产出全中文 task 文件。

### 备选：没有原生 skills 机制的 Codex

若某台 Codex 不支持 `~/.codex/skills/`，改为在 `~/.codex/AGENTS.md` 里加一段引用：

```markdown
## 写研究项目 task 文件
创建/更新 docs/<project>/tasks/ 下的 task 文件时，先读并遵循：
<本仓库克隆路径>/skills/research-task-writer/SKILL.md（产出全中文，含标题）。
```

`SKILL.md` 是纯 markdown、不依赖任何 Claude Code 专有工具名，两边可逐字使用。

## 维护

改 skill 直接改本目录文件，两条软链自动生效。本目录在 git 仓库内（`research-project-guide`），可按需提交做版本管理。
