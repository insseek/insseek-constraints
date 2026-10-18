# Agent 入口

本库所有文档对 Agent 均有约束力。只读本文件不够；执行任务前按下表加载。

冲突时优先级：**安全 > 工作质量 > 成本 / 效率**。

| 顺序 | 文档 | Agent 怎么用 |
|---|---|---|
| 1 | [编码 Agent 高效低成本协作约束（Agent 侧）](agent-collaboration-constraints.md) | 通用执行约束；先读「十行摘要 + 安全三条」 |
| 2 | [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md) | 其中规定 Agent 行为的条款必须执行；规定人职责的条款不代人做决定 |
| 3 | [代码规范](coding-constraints.md) | 写改代码时执行 |
| 4 | [写作与审核原则](writer-constraints.md) | 写文字、注释、提交信息、评审意见时执行 |
| 5 | [道德规范](life-good-and-antifragility-constraints.md) | 底线；任何情况下不破 |
| 6 | [DSH Agent 侧差异补充](dsh-agent-deltas.md) | 仅 DSH 环境；与通用规则冲突时以通用为准 |
| 7 | [项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md) | 项目有 custom 文件时，按六章结构执行，与通用同优先级 |
| 8 | [DSH 工作机制](dsh-working-mechanism.md) | 机制说明，照此操作；本身不作规范 |

加载规则：

- 1–5 每次任务都要加载；DSH 环境加 6；7 仅在项目存在时加。
- 8 仅在涉及 DSH 机制细节时查。
- 项目 `AGENTS.md` 只需指向本文件，不抄内容。
