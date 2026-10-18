# 通用约束文档库（insseek-constraints-docs）

> 与具体项目无关的约束、模板与机制说明。开新项目时整目录复制到项目根使用。
>
> **目的**：降本增效、确保安全、提高工作质量；冲突时 安全 > 质量 > 成本。
> **边界**：不依赖具体项目、机器、账户配置、preset 或插件；机制数字不手抄。
> `dsh-` 前缀是宿主差异；其余文档跨宿主通用。

## 文档

| 文档 | 性质 | 用途 |
|---|---|---|
| [代码规范](coding-constraints.md) / [写作与审核原则](writer-constraints.md) / [道德规范](life-good-and-antifragility-constraints.md) | 约束性（作者原文） | 代码、文字、道德基准；不改写 |
| [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md) | 约束性（人） | 人怎么用 Agent：三条红线、任务书、验收、安全 / 合规 |
| [编码 Agent 高效低成本协作约束（Agent 侧）](agent-collaboration-constraints.md) | 约束性（Agent 侧） | Agent 怎么执行：十行摘要、验证、上下文、安全治理 |
| [DSH Agent 侧差异补充](dsh-agent-deltas.md) | 约束性（DSH 差异） | DSH 入口、机制、安全边界；通用规则见上 |
| [项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md) | 模板 | 项目自定义内容骨架；六章，结构可校验 |
| [DSH 工作机制](dsh-working-mechanism.md) | 说明性 | DSH 机制；某台机器的实测，不作规范 |

## 怎么读

- **人**：[人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md)（最低限度三条；任务书 / 评审模板）
- **Agent**：[编码 Agent 高效低成本协作约束（Agent 侧）](agent-collaboration-constraints.md) 的「十行摘要 + 安全三条」；DSH 再读 [DSH Agent 侧差异补充](dsh-agent-deltas.md)
- **项目**：按[项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md)填《自定义内容》；机制问题查 [DSH 工作机制](dsh-working-mechanism.md)

## 在项目里使用

1. 复制整个目录到 `<项目>/insseek-constraints-docs/`。
2. 按[项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md)填 `agent-collaboration-constraints-project-custom.md`，六章结构不改。
3. 自检只校验章节结构；不校验正文，不要求与母版逐字一致。
4. 在项目文档索引里登记。

**加载成本**：项目入口只放「十行摘要 + 安全三条」或人侧「最低限度」；全文按需查阅。

## 维护

```
· 本库是唯一来源；改这里，不自动同步项目副本。
· 作者原文三份不改写。
· 不写机器、账户、插件、版本、阈值等环境事实；必要时引用机制文档。
· 命名：宿主相关加 dsh-，跨宿主不加。
· 改文档时同步本表。
```
