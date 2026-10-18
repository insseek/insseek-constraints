# 通用约束文档库（insseek-constraints-docs）

> 与具体项目无关的约束与模板。项目以 git submodule 引入本库，不复制。
>
> **目的**：降本增效、确保安全、提高工作质量；冲突时 安全 > 质量 > 成本。
> **边界**：不依赖具体项目、机器、账户配置、preset 或插件；机制数字不手抄。
> `dsh-` 前缀是宿主差异；其余文档跨宿主通用。

## 文档

| 文档 | 性质 | 用途 |
|---|---|---|
| [Agent 入口](AGENTS.md) | 入口（约束性） | 登记全部文档与加载顺序；项目 `AGENTS.md` 指向它 |
| [代码规范](coding-constraints.md) / [写作与审核原则](writer-constraints.md) / [道德规范](life-good-and-antifragility-constraints.md) | 约束性（作者原文） | 代码、文字、道德基准；不改写 |
| [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md) | 约束性（人） | 人怎么用 Agent：三条红线、任务书、验收、安全 / 合规 |
| [编码 Agent 高效低成本协作约束（Agent 侧）](agent-collaboration-constraints.md) | 约束性（Agent 侧） | Agent 怎么执行：十行摘要、验证、上下文、安全治理 |
| [DSH Agent 侧差异补充](dsh-agent-deltas.md) | 约束性（DSH 差异） | DSH 入口、机制、安全边界；通用规则见上 |
| [项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md) | 模板 | 项目自定义内容骨架；六章，结构可校验 |

## 怎么读

- **人**：[人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md)（最低限度三条；任务书 / 评审模板）
- **Agent**：从 [Agent 入口](AGENTS.md) 开始，按其顺序加载全部文档；先读「十行摘要 + 安全三条」
- **项目**：按[项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md)填《自定义内容》；机制细节以 DSH 官方文档为准

## 在项目里使用

本库只当一个只读依赖：新项目加 submodule 即可，不复制、不生成文件。

```bash
git submodule add https://github.com/insseek/insseek-constraints-docs.git \
  constraints/insseek-constraints-docs
git -C constraints/insseek-constraints-docs checkout v2.1   # 锁版本
git add .gitmodules constraints/insseek-constraints-docs
```

- **锁版本**：submodule 指针就是版本；升级时 `git -C … checkout v2.x` 并提交指针。
- **让 Agent 找到**：项目 `AGENTS.md` 只放一行路径：

  ```
  约束：constraints/insseek-constraints-docs/AGENTS.md
  ```

  该入口登记本库全部文档并规定加载顺序，Agent 必须逐项遵守；新增文档只改本库入口，项目不用改。
- **非 DSH 宿主**：同样是 submodule + 项目 `AGENTS.md` 指向入口；入口第 6 项标注仅 DSH。
- **人侧**：项目 README / CONTRIBUTING / 团队 wiki 指向同一 submodule 里的 [人使用编码 Agent 的协作约束](human-agent-collaboration-constraints.md)，先读 §0 三条红线。
- **项目自定义**：需要时在项目内写 `constraints/project-custom.md`，按[项目自定义内容模板](agent-collaboration-constraints-project-custom-template.md)六章结构；不写进 submodule。
- **只读**：不在 submodule 内改文件；改动提交到本库，项目再更新指针。

**加载成本**：项目入口只放路径；Agent 从入口按序加载，先读「十行摘要 + 安全三条」或人侧「最低限度」，其余按需。

## 维护

```
· 本库是唯一来源；项目用 submodule 指针引用，不产生副本。
· 作者原文三份不改写。
· 不写机器、账户、插件、版本、阈值等环境事实；必要时引用 DSH 官方文档。
· 命名：宿主相关加 dsh-，跨宿主不加。
· 新增、改名、删除文档时同步本表与 [Agent 入口](AGENTS.md) 的登记和顺序。
```
