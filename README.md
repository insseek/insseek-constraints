# 通用约束文档库（insseek-constraints）

里探个人自定义的 Agent 约束，给编码 Agent 和人共用的规则库：项目以 git submodule 引入，Agent 按入口加载，人按人侧文档执行。

> 与具体项目无关的约束与模板。
>
> **目的**：降本增效、确保安全、提高工作质量；冲突时 安全 > 质量 > 成本。
> **边界**：不绑定任何项目、机器、账户、配置或插件；不照抄会随版本变化的数字。
> 文件名以 `dsh-` 开头的只适用于 DSH，其余适用于任何编码 Agent。

## 文档

| 文档 | 性质 | 用途 |
|---|---|---|
| [Agent 入口](AGENTS.md) | 入口（约束性） | 登记全部文档与加载顺序；项目 `AGENTS.md` 指向它 |
| [代码规范](coding-constraints.md) / [写作与审核原则](writer-constraints.md) / [道德规范](life-good-and-antifragility-constraints.md) | 约束性（作者原文） | 代码、文字、道德基准；他人不改写 |
| [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md) | 约束性（人） | 人怎么用 Agent：三条红线、任务书、验收、安全 / 合规 |
| [编码 Agent 高效低成本协作约束（Agent 侧）](agent-collaboration-constraints.md) | 约束性（Agent 侧） | Agent 怎么执行：十行摘要、验证、上下文、安全治理 |
| [DSH Agent 侧差异补充](dsh-agent-deltas.md) | 约束性（DSH 差异） | DSH 入口、机制、安全边界；通用规则见上 |
| [开发流程规范](development-workflow-constraints.md) | 约束性（开发流程） | 加功能、重构、改旧功能的做法与验收 |
| [项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md) | 模板 | 项目自定义内容骨架；六章结构固定 |

## 怎么读

- **人**：[人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md)（最低限度三条；任务书 / 评审模板）
- **Agent**：从 [Agent 入口](AGENTS.md) 开始，按加载规则读；先读「十行摘要 + 安全三条」
- **项目**：按[项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md)填《自定义内容》；DSH 的具体机制以官方文档为准

## 在项目里使用

不指定版本（固定到添加时 `main` 的提交）：

```bash
git submodule add https://github.com/insseek/insseek-constraints.git
git add .gitmodules insseek-constraints
```

指定版本（推荐，日后能确知用的是哪一版）：

```bash
git submodule add https://github.com/insseek/insseek-constraints.git
git -C insseek-constraints checkout v3.2
git add .gitmodules insseek-constraints
```

两种方式都会把项目锁在具体提交上，之后母库更新不会自动跟。

项目根目录 `AGENTS.md` 写一行（没有就新建）：

```
约束：insseek-constraints/AGENTS.md
```

Agent 按入口顺序加载；人读 [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md)。升级只改它指向的提交：`git -C insseek-constraints checkout v1.x` 后提交。不复制、不改 submodule 内文件。

## 维护

```
· 本库是唯一来源；项目用 submodule 指向本库，不产生副本。
· 作者原文三份：他人不改写正文，作者本人增补除外。
· 不写机器、账户、插件、版本、阈值等环境事实；必要时引用 DSH 官方文档。
· 命名：只适用于 DSH 的加 `dsh-`，通用的不加。
· 新增、改名、删除文档时同步本表与 [Agent 入口](AGENTS.md) 的登记和顺序。
· 本库由人维护；Agent 只读遵守，修改建议由人决定。
· 许可证：MIT。
```
