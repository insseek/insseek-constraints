# Agent 入口

本库所有文档对 Agent 均有约束力；需要读哪些、读到什么程度，见下表。

冲突时优先级：**安全 > 工作质量 > 成本 / 效率**。

默认硬规则（适用任何使用本库的项目）：

- 提交之外的对外动作——`git push`、合并、打发布 tag、部署、上线——都要人明确确认后再做。
- 本库自身同样适用：改动只提交到本地，等人说「推送」再推。

| 顺序 | 文档 | Agent 怎么用 |
|---|---|---|
| 1 | [编码 Agent 高效低成本协作约束（Agent 侧）](agent-collaboration-constraints.md) | 必读：先读「十行摘要 + 安全三条」，其余按需 |
| 2 | [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md) | 人侧文档；只执行其中约束 Agent 的条款，不必通读 |
| 3 | [代码规范](coding-constraints.md) | 写改代码时读 |
| 4 | [写作与审核原则](writer-constraints.md) | 写给人读的文字（文档、注释、提交信息、评审意见）时读 |
| 5 | [道德规范](life-good-and-antifragility-constraints.md) | 人侧底线；Agent 不必读，行为受第 1 条安全条款约束 |
| 6 | [DSH Agent 侧差异补充](dsh-agent-deltas.md) | 仅 DSH 环境；与通用规则冲突时以通用为准 |
| 7 | [开发流程规范](development-workflow-constraints.md) | 加功能、重构、改旧功能时读 |
| 8 | [项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md) | 项目有 custom 文件时，按六章结构执行，与通用同优先级 |

加载规则：

- 任务开始：读第 1 条的「十行摘要 + 安全三条」，其余按需。
- 涉及代码：加第 3、7 条；涉及给人读的文字：加第 4 条。
- DSH 环境：加第 6 条；项目有自定义内容：加第 8 条。
- 第 2、5 条是给人看的，Agent 不要求通读，但不得违反其中约束自己的条款。
- 非 DSH 宿主（OpenCode、Claude Code 等）跳过第 6 条。
- 项目 `AGENTS.md` 只需指向本文件，不抄内容。
- 本库由人维护：Agent 只读遵守，不自行修改；有修改建议，说明理由交人决定。
