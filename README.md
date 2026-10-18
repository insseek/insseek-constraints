# 通用约束文档库（insseek-constraints-docs）

> 与具体项目无关的约束、模板与机制说明。项目以 git submodule 引入本库，不复制。
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

本库只当一个只读依赖：新项目加 submodule 即可，不复制、不生成文件。

```bash
git submodule add https://github.com/insseek/insseek-constraints-docs.git \
  constraints/insseek-constraints-docs
git -C constraints/insseek-constraints-docs checkout v2.0   # 锁版本
git add .gitmodules constraints/insseek-constraints-docs
```

- **锁版本**：submodule 指针就是版本；升级时 `git -C … checkout v2.x` 并提交指针。
- **让 Agent 找到**：项目 `AGENTS.md` 里加一行路径，如「遵守 `constraints/insseek-constraints-docs/agent-collaboration-constraints.md`」，只放路径引用，不抄正文。
- **项目自定义**：需要时在项目内写 `constraints/project-custom.md`，按[项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md)六章结构；不写进 submodule。
- **只读**：不在 submodule 内改文件；改动提交到本库，项目再更新指针。

**加载成本**：项目入口只放路径；Agent 打开后先读「十行摘要 + 安全三条」或人侧「最低限度」，全文按需查阅。

## 维护

```
· 本库是唯一来源；项目用 submodule 指针引用，不产生副本。
· 作者原文三份不改写。
· 不写机器、账户、插件、版本、阈值等环境事实；必要时引用机制文档。
· 命名：宿主相关加 dsh-，跨宿主不加。
· 改文档时同步本表。
```
