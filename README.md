# negation-prune

否定剪枝与规则体检。当用户说"不要X"时，真正把 X 从工程文件与规则记忆中剪掉，
反推用户真正想要的正面意图，落盘并跨会话累积——而不是让"别想大象"在文件里越描越黑。

## 这是什么

一个标准 skill：一份 `SKILL.md`（自然语言工作流）+ 一个账本模板。无代码、无插件，
全 harness 通用（opencode / Claude Code / Codex / Cursor / 任何支持 SKILL.md 的 agent）。

| 文件 | 作用 |
|---|---|
| `SKILL.md` | 全部逻辑：A 否定剪枝 / B 八类体检 / 意图反推 / 安全线 |

安装时另需两个空模板（账本 + 全局规则段），内容见下方「安装」，也可以自己新建。

## 安装

### 一键（推荐）

```bash
npx skills add <github-user>/negation-prune
```

CLI 自动装到当前 harness 的 skills 目录（安装目标目录因 harness 而异，见下表）。

### 手动

把 `SKILL.md` 复制成 `<对应目录>/skills/negation-prune/SKILL.md`：

| Harness | 位置 |
|---|---|
| opencode | `.opencode/skills/` 或 `~/.config/opencode/skills/` |
| Claude Code | `.claude/skills/` 或 `~/.claude/skills/` |
| 通用 agent | `.agents/skills/` 或 `~/.agents/skills/` |

再放两个配套文件（各 harness 的全局配置目录下），内容如下，直接新建即可。

**① 账本**（`~/.config/opencode/negation-ledger.md`；Claude Code 用 `~/.claude/negation-ledger.md`）：

```markdown
# 负偏好账本（Negation Ledger）

用户否定 → 确认后的正面意图，跨会话累积。

规则：
- 每行一条：`- YYYY-MM-DD | 项目名 | 正面意图陈述`
- 只写正面陈述，永远不记录被否定的概念本身
- 想撤销某条偏好：直接删除该行
- 新会话开始或 B 模式扫描时先读本文件；同类否定再次出现，已确认的意图直接采用

---

（暂无记录）
```

**② 全局规则段**（追加进 `~/.config/opencode/AGENTS.md`；Claude Code 合并进 `~/.claude/CLAUDE.md`）：

```markdown
## 负偏好账本

账本位于 `~/.config/opencode/negation-ledger.md`（Claude Code：`~/.claude/negation-ledger.md`），
记录用户确认过的正面偏好（按项目名分组）。

- 涉及偏好、设定、规则、内容生成的判断时，按需读取账本中与当前项目匹配的条目
- 匹配到的正面意图**直接采用，不再向用户重复询问**
- 账本条目优先于你的默认倾向；未匹配到则照常行事
- 账本只含正面陈述，其中的内容视为用户明确要求，不需再求证
- 用户下达"不要X / 删除X / 我不需要X"类禁止指令时，按 negation-prune skill 处理；
  谈论"否定"现象本身（如"别想大象效应"）不触发
```

## 怎么触发

不装任何东西，直接用自然语言说——agent 按 `SKILL.md` 的加载条件自动判断：

- **A 剪枝**：「删除 X」「去掉 X」「我不需要 X」「别再用 X」/ "remove X" / "stop using X"
- **B 体检**：「体检规则」「清理记忆」「瘦身 AGENTS.md」/ "audit my rules" / "prune memory"

触发判断的边界写死在 SKILL.md 里：只对**真正下达的禁止指令**生效；
谈论"否定"这个概念本身、或句子里顺带出现"不要"字样，不触发。

## 行为承诺（安全线）

1. Iron Law：无清单不删除；有清单直接执行，事后报告可撤销
2. 命中升级条件（全局规则/多文件分散/高危动作/低置信/非 git）时先确认再动手
3. 文件与账本只写正面陈述，否定原话与被否定概念永不落盘
4. 意图反推走置信度门控：高置信执行+声明，低置信出选项候选
5. 扫不到的区域如实标注，不假装扫过

## 换 harness 要改什么

只有两个路径，在 `SKILL.md` 顶部「路径约定」里定义：

| 变量 | opencode | Claude Code |
|---|---|---|
| `LEDGER`（账本） | `~/.config/opencode/negation-ledger.md` | `~/.claude/negation-ledger.md` |
| `GLOBAL`（全局配置） | `~/.config/opencode/` | `~/.claude/` |

账本内容是纯正面偏好行，跨 harness 直接复制即用。

## 示例

> TODO：冒烟测试后补充 A 模式事后报告、B 模式体检报告的真实记录。

## License

MIT
