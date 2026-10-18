# DeepSeek Harness 工作机制

> **作者** DeepSeek v4.1-flash　｜　**时间** 2026-10-18　｜　**版本** DSH 0.2.1-rc.2
>
> **性质：说明性。** 描述 DSH 实际怎么运转，不是规范或约束。
>
> **一句话**：本文说明 DeepSeek Harness（DSH）如何把一个语言模型装配成能读写文件、执行命令、
> 检索网页、委托子任务的编码 Agent——它由什么组成、一次任务如何运转、边界在哪。

---

## 本文是什么

本文回答三个问题：

1. **它由什么装配而成**——宿主、插件、会话如何组合出一个可运行的 Agent；
2. **一次任务如何运转**——从接收需求到交付，循环、上下文、委托各起什么作用；
3. **它有什么边界**——哪些事做不了，哪些行为会随配置变化。

- **读者**：使用、维护、排障 DSH 的人；不需要读过源码。
- **范围**：与任何具体项目无关（项目只是它的使用者）；以官方内置插件为准。
- **可信度**：文中逐条标注来源（实测 / 官方文档 / 工具接口 / 归纳），见「取材与可信度」；
  本机实测事实与复核命令集中在文末附录。

---

## DSH 是什么

DSH 是 DeepSeek 官方的 Agent 宿主，开源（MIT），以 npm 包
`@deepseek-ai/dsh` 分发。它**不是模型**，也**不是 IDE 插件**，而是「把模型接到工具与
工作区之上的运行时」：

- **一个启动器**：`dsh <profile>`，profile 决定这一次装什么；
- **profile 是插件包的有序补丁层栈**：官方基础层（`dsh-base`、`dsh-web-app`）在前，
  插件与用户覆盖在后；
- **多个入口**：Web GUI、`headless`（一次性任务）、`sdk`、`acp`，共用同一套 Agent 循环；
- **本机形态**：profile `web`，preset `standard`，模型 `deepseek-flash`。

---

## 与同类系统的关系

DSH 的直接同类是终端/浏览器里的编码 Agent：**Claude Code、Codex CLI、Gemini CLI、OpenCode**。
五者共享同一骨架——模型在循环里调用工具、结果回注上下文、接近上限时压缩、必要时派生子代理；
因此本文的循环、成本、压缩原理对它们同样成立。

> 注意区分层级：GPT、Grok、Gemini、Claude、DeepSeek 是**模型**，不是同类产品；
> 比较只在 Agent 宿主这一层进行。宿主可以接不同模型（例如 OpenCode 与 DSH 都能换路由），
> 因此「谁更强」要先固定模型再比宿主。

差别在六个维度上：

| 维度 | 同类常见做法 | DSH 的做法 | 差异意味着什么 |
|---|---|---|---|
| 扩展方式 | 内置工具 + MCP 服务器 | 插件 bundle + 可替换接缝（`ctx.shell`、`ctx.web` 等，也支持 MCP） | 换的是能力实现，不只是加工具 |
| 会话装配 | 全局或项目级配置 | 每个会话选一次 agent preset | 同一宿主可跑不同工具面 |
| 上下文压缩 | 接近上限时摘要化，部分产品提供手动命令 | 宿主自动触发，摘要 + 逐字保留尾部，参数可配（见第 4 节） | 触发点不是固定百分比 |
| 委托与编排 | 部分产品提供子代理 | 子代理（spawn / fork）+ workflow 并行 | fork 可复用已有上下文 |
| 持久化 | 会话文件或数据库 | 追加式会话日志，按工作区分组（见第 8 节） | 可回放、可迁移 |
| 权限 | 审批 + 沙箱 | 三档沙箱 + 审批，无应答者时失败关闭（见第 7 节） | 越界会被拒，而不是静默执行 |

> 只比较机制形态，不比较功能多少，也不评价优劣；DSH 一侧可在文末附录复核。
> 通用部分可迁移；DSH 特有部分（profile / preset / 接缝、日志格式、触发公式）不可直接套用。

---

## 术语约定

| 本文用法 | 含义 | 常见英文 |
|---|---|---|
| **轮（turn）** | 一次用户输入触发的完整处理过程 | turn |
| **步（step）** | 一次模型推理：读入当前上下文，产出文本或工具调用 | step |
| **接缝（seam）** | 插件宿主暴露的可替换服务接口，如 `ctx.shell`、`ctx.web` | seam |
| **工具（tool）** | 面向模型的调用入口，背后接某个接缝 | tool |
| **上下文** | 每一步都要重新发送给模型的完整可见内容，兼作工作记忆 | context |
| **提示缓存** | 服务端复用相同前缀；命中部分按缓存价计费 | prompt cache |
| **自动压缩** | 宿主按占用率触发，把已消费历史替换为摘要；有损，模型侧不可逆 | compaction |
| **无损保留** | 压缩时逐字保留、不改写 | verbatim retention |
| **子代理** | 独立上下文中的执行单元，只回传结论 | subagent |
| **运行时事实** | 宿主每一步注入的环境信息：占用率、策略、路径、时间 | runtime facts |

---

## 取材与可信度

| 来源 | 含义 | 本文中的例子 |
|---|---|---|
| **实测** | 在本机实际执行、实际读取 | 版本、窗口、压缩触发点、工具清单、会话日志格式 |
| **官方文档** | DSH 自带 README 与 `dsh --help` | 启动方式、子命令、profile 的定义、入口模式 |
| **工具接口** | 各工具的 schema 与说明 | 第 3 节的工具能力与用法 |
| **归纳** | 为便于理解做的抽象，不是产品定义 | 1.3 节的六层、第 0 节的概要 |

凡「归纳」处均已标注；实测与配置不一致时据实并列，不取其一。

---

## 0. 概要

```
装配     profile（bundle 补丁层栈）→ preset（每会话再选一次）→ 工具面
插件     接缝（ctx.*）+ 实现（可替换）+ 工具（面向模型）
循环     推理 → 工具调用（并发安全者并行）→ 结果回注 → 推理 → …
成本     每步重发可见上下文；稳定前缀可命中缓存；长结果会被裁剪
上下文   宿主自动压缩（公式固定，参数可配）；有损，模型侧不可逆
约束     沙箱（可写范围）· 审批（越界需授权，失败关闭）· 门禁
旁路     子代理（独立上下文）· 后台作业（长耗时）· 目标轮次（自动续跑）
```

---

## 1. 系统构成

### 1.1 宿主：DSH

```bash
dsh [--profile] <name> [app-args…]        # 启动一个 profile
dsh plugin --profile <name> <pnpm-args…>  # 为 profile 安装/卸载插件
dsh --dump-config / --dump-config-schema / --dump-default-config
dsh --patch <path>                        # 追加一层补丁覆盖（可重复）
```

- **profile**：若干 bundle 按顺序叠加，最后叠加用户的 `cordis.patch.yml`；补丁层可热加载。
- **agent preset**：创建会话时再选一次插件组合（官方内置 `standard`、`ptc`、`minimal`、`cordis`），
  决定这次会话用哪些工具、哪套提示、如何压缩；会话头记录 `agentPreset`。
- **入口**：`web`（浏览器）、`headless`（一次性任务）、`sdk`、`acp`（自动化协议）、`tui`（若安装）。
- **本机**：`web` profile + `standard` preset，官方两层之上另装了 6 个社区插件（见附录 A）。

### 1.2 插件 = 接缝 + 实现 + 工具

这是理解 DSH 的关键，也是官方内置插件的共同结构：

```
接缝（abstract）      ctx.shell · ctx.web · ctx.subagents · ctx.approval · ctx.jobs · ctx.fs
  ↑ 实现（可替换）     bash-sandbox / bash-local · web-search-deepseek / web-fetch-http · spawn / fork
  ↑ 工具（面向模型）   tool-bash · tool-web · tool-subagent · tool-jobs · …
```

| 能力 | 接缝 | 官方实现 | 面向模型的工具 |
|---|---|---|---|
| Agent 循环 | —（`dsh-agent-loop` 直接驱动） | `dsh-agent-loop` | — |
| 终端 | `ctx.shell` | `dsh-bash-sandbox` / `dsh-bash-local` | `bash` |
| 后台作业 | `ctx.jobs` | `dsh-jobs-local` | `job_output`、`job_list`、`job_kill` |
| 文件 | `ctx.fs` | `dsh-fs-sandbox` / `dsh-fs-local` | `read`、`write`、`edit`、`glob`、`grep`、`read_image` |
| 网页 | `ctx.web` | `dsh-web-search-deepseek`、`dsh-web-fetch-http` | `web_search`、`web_fetch` |
| 委托 | `ctx.subagents` | `spawn` / `fork`（in-process provider） | `subagent`、`subagent_fork`、`workflow` |
| 审批 | `ctx.approval` | `dsh-user-approval` | —（阻塞在工具调用上） |
| 沙箱 | `ctx.sandbox` + 策略解析 | `dsh-sandbox-local`、`dsh-sandbox-policy` | —（策略写入上下文） |

**换实现只改装配，不改工具**：例如终端可以从沙箱执行器换成宿主执行器，`bash` 工具的用法不变。

**四个内置插件，同一套结构**：

- **Agent 循环**（`dsh-agent-loop`）：不是接缝，而是驱动器——它按「一步」调度模型请求与工具执行，
  并发安全者并行（上限可配），无工具调用时结束。循环本身也是一个可替换的插件。
- **终端**（`ctx.shell`）：接缝只定义「执行命令」；`dsh-bash-sandbox`（沙箱内）与
  `dsh-bash-local`（宿主子进程）是两种实现；`dsh-tool-bash` 把它暴露成 `bash`。
  配置点是超时、后台作业等；**换执行器不改工具接口**。
- **网页搜索**（`ctx.web`）：接缝维护 provider 注册表，搜索与抓取各选一家
  （DeepSeek 搜索、HTTP 抓取）；`dsh-tool-web` 只负责调用。外部依赖因此可整体替换。
- **子智能体**（`ctx.subagents`）：接缝维护具名 provider——`spawn`（独立上下文）与
  `fork`（继承会话）；`subagent` / `subagent_fork` 是两个挂载点，控制工具负责后续对话。
  同一能力按不同策略实现，并按会话选择。

这四个例子共同说明：**新增能力 = 一个接缝 + 至少一个实现 + 一个工具插件**；
**替换能力 = 换实现**；**调整行为 = 改配置**。

### 1.3 六个逻辑层（归纳）

> 本节分层是为便于理解做的归纳；产品自身的结构是「profile 补丁层栈」＋「preset 每会话组合」。

| 层 | 职责 | 本机实现 |
|---|---|---|
| **模型层** | 判断、分解、写作、取舍 | `deepseek-flash`（`deepseek-account`，`reasoningEffort: high`） |
| **工具层** | 与外界交互 | 核心工具见 3.2；本机另有社区插件追加（见 3.3） |
| **上下文层** | 计量、裁剪、压缩 | `compaction-basic`、`tool-result-pruner`（preset 内）、`spill-policy`（宿主级） |
| **委托层** | 把独立任务移出主上下文 | `subagent` / `subagent_fork` / `workflow` |
| **约束层** | 权限边界、审批、门禁 | 沙箱 · 审批 · 文件观察策略 |
| **持久层** | 会话、目标、工作区归属 | 会话日志 · `goal` · workspace |

**术语澄清**：DSH 中的 ACP 指 **Agent Client Protocol**（JSON-RPC 自动化服务端），
与上下文管理无关；本机也没有 `acp_status` / `compress` / `decompress` / `search_context`
这类工具——压缩由宿主侧自动完成（见第 4 节）。

### 1.4 单次调用的成本模型

```
每步请求 = 系统提示 + 工具 schema + 全部可见历史（含工具结果）+ 新增
每步计价 = 缓存命中的前缀（低价）+ 未命中的前缀（全价）+ 输出
```

- **固定前缀很大**：本机 37 个工具（核心 + 社区）的描述与 schema 序列化后约 3.4 万字符；
- **前缀稳定则命中缓存**：本机一条 `usage` 实测为新输入 5,945 + 缓存命中 4,224 + 输出 156；
- **工具结果进入上下文**，但超长部分会被宿主裁剪（见 4.4）。

结论：成本 ≈ 新增输入 + 缓存前缀 + 输出；两个杠杆仍然是**上下文长度 × 步数**。
缓存只给固定前缀打折，不会让「多走一步」变便宜。

---

## 2. 工作循环

### 2.1 一步做什么

```
组装请求（系统提示 + 工具 schema + 可见历史）
  → 调模型
  → 写 assistant 消息（文本 + 工具调用）
  → 执行工具调用
  → 结果作为 tool/result 追加
  → 无工具调用？结束；否则进入下一步
```

会话日志按此顺序记录 `turn/start`、`step/start`、`assistant/message`、`tool/call`、
`tool/result`、`step/end`。一次真实任务（本机某会话）：**46 步 / 74 次工具调用 / 8 条用户消息**
——工具调用远多于用户消息，这就是成本集中在「步数 × 结果」的原因。

界面上看到的 `Think`、`Ran commands`、`Analysis completed` 等，是这些记录的展示形态，
对照见 3.5。一次任务在界面上大致呈现为：

```
Think → Analysis completed → Read files → Ran commands → Edited files → … → 最终回复
（每步的思考块）    （步末摘要）      （该步的工具调用按类别汇总）
```

### 2.2 并行与互斥

同一 assistant 消息里的多个工具调用按并发安全调度：

- 工具声明 `isConcurrencySafe(args)`；未声明或抛错 → 视为不安全；
- 安全调用进入滚动池并行执行，上限由 `maxParallelToolCalls` 决定（默认 10）；
- 遇到不安全调用即形成屏障：之前的并发池等它跑完，之后的调用再重新分类；
- 中断时未开始的调用记为 skipped，而不是静默丢失。

### 2.3 每一步注入的运行时事实

宿主把环境事实**重新注入**可见上下文（不改写系统提示，因此不破坏缓存前缀）：

```
当前时间（采样于准备该步时）与时区
文件策略（read-only / workspace-write / danger-full-access）
审批策略（ask / never）
实现目录、Web GUI 地址
占用率与压缩点（由插件注入；是否出现取决于装了什么）
```

因此「上下文是否紧张」以宿主注入的数字为准，而不是模型自己的感觉。

### 2.4 结束与续跑

| 情形 | 行为 |
|---|---|
| 模型不再发起工具调用 | 本轮结束，控制权交回用户 |
| 活跃目标（goal）未完成 | 宿主注入 `<goal_round>`，同一会话自动续跑 |
| 处于计划模式 | 只能调研与提交方案，`exit_plan_mode` 交人审 |
| 后台作业完成 | 宿主在会话内通知，不要求轮询 |

### 2.5 自动纠偏

| 机制 | 触发 | 效果 |
|---|---|---|
| 重复调用提醒 | 同一工具、同样参数连续 3 / 5 / 8 次 | 注入提醒，要求换参数或换路径 |
| 工具超时 | 超过工具声明的时限 | 长任务转为后台作业，不静默丢弃 |
| 沙箱拒绝 | 越界写入 | 返回拒绝原因，可申请一次性扩权 |
| 审批挂起 | 操作需要授权 | 阻塞等待；无应答者时失败关闭 |

### 2.6 典型任务的阶段（观察，不是规范）

```
只读勘察（测量规模）→ 计划（多步任务先给步骤）→ 执行（最小改动）
→ 验证（读产物，不靠「看起来对」）→ 持久化（写清改了什么、为什么、验证了什么）→ 复盘
```

这是从实际任务中归纳的常见路径，不是机制要求。

---

## 3. 工具面

### 3.1 工具从哪来

工具由工具插件注册到注册表（`ctx.tools`），再以某种**呈现方式**交给模型：

| 呈现方式 | 形态 |
|---|---|
| `native` | 每个工具是一条独立工具调用（默认） |
| `ptc` | 工具以程序化方式呈现，供代码式调用 |

同一套工具可以两种呈现，说明「工具清单」是装配结果，不是产品常量。

### 3.2 核心工具（31 个）

| 能力 | 工具 |
|---|---|
| 命令执行 | `bash`、`working_directory` |
| 后台作业 | `job_output`、`job_list`、`job_kill` |
| 文件 | `read`、`read_image`、`write`、`edit`、`glob`、`grep`、`present` |
| 网络 | `web_search`、`web_fetch` |
| 委托 | `subagent`、`subagent_fork`、`workflow`、`send_message`、`list_agents`、`interrupt_agent` |
| 交互与规划 | `ask_user_question`、`exit_plan_mode`、`todo_write` |
| 提醒 | `schedule_create`、`schedule_update`、`schedule_list`、`schedule_delete` |
| 目标 | `create_goal`、`get_goal`、`update_goal` |
| 技能 | `skill` |

**这是能力目录，不是固定清单**：数量随 preset 与平台变化（`minimal` 不含委托与规划；
Windows 以 `pwsh` 替代 `bash`）。本机实际数量见附录 A。

### 3.3 本机追加的社区工具（6 个）

`plugin_vet`、`mv_session`、`mover_doctor`、`mover_list_sessions`、`mover_move_session`、
`mover_repair_sessions`——来自社区插件，**不属于 DSH 核心机制**，卸载即消失。

### 3.4 三条契约

1. **描述即契约**：沙箱、审批、超时语义写在各工具说明里。
2. **先读后改**：`write` / `edit` 以「本会话内已观察过该文件」为前提（`fs-observation-policy`）。
3. **结果进上下文**：每次调用结果都会进入后续每一步，只是可能被裁剪。

### 3.5 界面词汇：轨迹里看到的是什么

Web GUI 显示的是**展示层词汇**：它按工具类别归并、用便于阅读的名字；
机制层用的是工具名与记录类型。两者是同一事件的两套名字。正文以机制词为主，因为
界面词随语言与版本变化（中文界面会把 `bash` 显示成「运行命令」），机制词则稳定：

| 界面词汇 | 机制含义 | 说明 |
|---|---|---|
| Think | 该步的思考块（reasoning） | 折叠显示；只是展示，不改变工具面 |
| Analysis completed | 思考阶段结束（进入工具调用） | 该步的思考摘要 |
| Ran commands | `bash` | 命令执行 |
| Read files / Read images | `read` / `read_image` | 文件读取 |
| Wrote files / Edited files | `write` / `edit` | 新建 / 替换 |
| Searched code | `grep` / `glob` | 代码检索 |
| Searched the web / Visited web pages | `web_search` / `web_fetch` | 外部内容按数据处理 |
| Create subagent | `subagent` / `subagent_fork` | 派生独立上下文的子代理 |
| Run workflow | `workflow` | 脚本化并行分发 |
| Coordinated subagents | 子代理相关的一批调用 | 按类别汇总 |
| Updated the plan | 计划模式、任务清单（`todo_write`） | |
| Asked questions | `ask_user_question` | 需要用户确认 |
| Called tools | 其它工具 | 兜底归类 |
| Background jobs（含计数） | `job_output` / `job_list` / `job_kill` 与后台作业列表 | 长任务不阻塞当前步 |
| Plan | 计划模式开关 | `/plan` |
| Cache hit N% | 提示缓存命中率 | 会话统计 |
| N turns N steps | 轮数 / 步数 | 会话统计 |
| Work details：Standard / Detailed / Verbose / Compact | 展示密度设置 | 只改显示，不改机制 |

> 读会话日志时不要找 `Bash`：日志记的是 `tool/call` 与工具名（`bash`）。
> 界面按类别归并，所以一次 `read` 加一次 `grep` 可能显示成「Read files, Searched code」。

---

## 4. 上下文管理

### 4.1 窗口与预留

| 符号 | 含义 | 本机取值 |
|---|---|---|
| `W` | provider 上报的上下文窗口 | `1,048,576` |
| `R` | 单次请求预留的输出上限（`maxTokens`） | `256,000` |
| `H` | 压缩余量（`headroomTokens`） | `65,536` |
| `λ` | `thresholdRatio` | `0.8`（默认） |

### 4.2 触发公式（机制固定，参数可配）

```
触发点 = min(⌊W · λ⌋, W − R − H)
保留尾部 = ⌊(W − R) · retainRatio⌋，或直接指定 retainTokens
```

**起决定作用的是容量预算，不是比例**：`0.8 × W` 远大于 `W − R − H`，所以只调 `λ`
（在 0.693 以上）不会改变触发点；改变 `W` 或 `maxTokens` 才会。

可配置项：`thresholdRatio`、`headroomTokens`、`retainRatio` / `retainTokens`、`maxTokens`、
`auto`、`modelPolicies`（按 provider/model 精确覆盖）、`summarizationProvider` / `summarizationModel`。
**这些是配置，不是机制常量**：换配置就换数字，公式不变；第 4.1 节的取值与附录 B 的算例只描述本机当前配置。

### 4.3 触发后发生什么

```
① 宿主按实测占用率判断，达到触发点 → 自动压缩（模型不参与决策）
② 压缩引擎发起一次摘要请求，把已消费历史写成摘要
③ 可见历史 = 摘要 + 逐字保留的尾部
④ 原始记录仍在会话日志里；模型侧不可见
```

压缩本身是一次模型请求，也要花钱；摘要是有损的，不是原文。

### 4.4 三个裁剪器（在压缩之前，更频繁）

| 机制 | 阈值（可配） | 行为 |
|---|---|---|
| `spill-policy` | 单条结果 > 12,500 tokens | 全文落盘，上下文只留预览 + 位置提示 |
| `tool-result-pruner` | 单条结果 > 8,192 字符 | 压成「头部 4,096 + 尾部 1,024 字符」 |
| `image-offload` | 图片超预算 | 最旧图片换占位符后重试 |

推论：需要细节时不能假设「刚才那条输出的全文还在上下文里」。

### 4.5 恢复与手动操作

- 模型侧没有恢复被压缩原文的工具；需要原文只能读会话日志；
- 用户可以执行 `/compact` 提前压缩，但不能指定内容、不能还原。

### 4.6 常见误解：把百分比当固定阈值

触发点不是固定百分比，而是由公式算出。同一份配置下：

```
窗口 1,000,000 → 1,000,000 − 256,000 − 65,536 = 678,464（67.8%）
窗口 1,048,576 → 1,048,576 − 256,000 − 65,536 = 727,040（69.3%）
```

百分比变化只是因为窗口变了，不是阈值变了。此外，配置目录里遗留的其它比例值未必是
实际触发点——以宿主注入的占用提示与公式为准。

---

## 5. 任务委托

### 5.1 三种形态

| 形态 | 上下文 | 适用 |
|---|---|---|
| `subagent` | **独立**：任务需自带全部背景 | 自包含的调研、扫描、迁移、多角度验证 |
| `subagent_fork` | **继承**当前会话 | 评审、续做、基于已有讨论的分析 |
| `workflow` | 独立子代理池 | 大规模并行：阶段、进度、schema 校验、pipeline / parallel |

### 5.2 子会话是真实会话

子代理有自己的会话、自己的步数、自己的上下文；会话头 `delegationDepth ≥ 1`，
与主会话同级持久化。主上下文只收到**最终结论**，收不到中间步骤。

### 5.3 成本结构

```
主上下文付出：委派指令 + 返回结果
主上下文省下：子代理的全部中间步骤
子代理侧：自己的上下文与步数，按同样规则计费
```

因此任务描述要**写全**（独立上下文看不到主会话），返回要**收敛**（结论 + 证据位置）。

### 5.4 适用判断

- 适合：任务可自证、边界清楚、中间过程对主上下文无价值；
- 不适合：一两次调用就能完成；需要频繁与用户确认；需要复用主会话细节而不想 fork。

> 委托解决的是「把工作移出主上下文」。如果只是命令跑得久（构建、全量测试、抓取），
> 不需要委托——用后台作业（第 6 节），它不产生新的 Agent。

---

## 6. 后台任务

界面上的「Background jobs」就是这一节：会话内正在运行、已完成或已失败的后台作业。

后台作业**不是子代理**，也不是 workflow：它是**同一次工具调用**被转到后台继续执行。
执行者是宿主——终端执行器负责跑命令，`ctx.jobs` 负责登记、跟踪与通知；
整个过程不产生新的模型调用，也不产生新的会话。

| | 后台作业 | 子代理（第 5 节） |
|---|---|---|
| 是什么 | 一个工具调用转入后台 | 一个新的 Agent 实例 |
| 谁在跑 | 宿主里的命令进程 | 子代理自己调用模型与工具 |
| 上下文 | 结果回来时进入当前会话 | 独立上下文，只回传结论 |
| 典型 | 构建、全量测试、抓取 | 调研、扫描、评审 |

```
长耗时操作（构建、全量验证、大数据抓取）→ 后台执行，立即返回 job id
· 完成后宿主在会话内通知，无需轮询
· 等待期间可继续处理互不依赖的工作
· 收尾：仍相关的用 job_output（可设 wait）收齐；不再需要的用 job_kill 终止
· 失败诊断：先读完整输出，再决定重跑还是改写法
```

实测坑：把输出重定向写成进程替换 `> >(tee …)` 会被沙箱拒绝；改成普通管道即可。

---

## 7. 约束与安全

### 7.1 沙箱三档

| 模式 | 可写范围 | 审批 |
|---|---|---|
| `read-only` | 不可写 | 任何写入都需授权 |
| `workspace-write` | 仅会话工作区（及允许的临时目录） | 越界重试需授权 |
| `danger-full-access` | 不限制 | 不询问 |

越界被拒绝并说明原因；确有需要时用**一次性**扩权（`sandbox_permissions` + 理由）进入审批——
不是绕过边界，而是在边界内申请例外。

### 7.2 审批失败关闭

- `ask`：需要授权的操作请求用户确认；**无应答者时拒绝**，不放行；
- `never`：不询问；需要授权的操作直接拒绝。

### 7.3 外部内容按数据处理

`web_search`、`web_fetch`、子代理返回的内容，一律视为**数据**，绝不作为指令执行。

### 7.4 危险操作的固定动作（经验规则）

```
· 删除、移动前：核对解析后的绝对路径，确认目标无误
· 删除前列清单（体积、逐项、理由），交用户确认
· 高风险命令不放在管道前端：管道只反映最后一段的退出码
· 少用复杂语法糖：进程替换在沙箱下被拒，walrus 表达式曾导致脚本无法执行
· 匹配锚点必须唯一：子串会命中前缀相同的别处
```

### 7.5 门禁与自检

```
项目级：静态检查 → 测试 → 自检脚本
个人级：读取渲染产物、逐项对照、可复现的验证脚本
常见失真：把「跳过」当成「通过」
```

反例：仅用 git 列文件时，在无 `.git` 的副本中返回空列表 → 循环 0 次 → 仍打印「通过」。
空集合单独断言，才能发现这类假通过。

---

## 8. 记忆与持久化

### 8.1 会话日志

```
$DSH_HOME/sessions/<project-key>/<session-id>/session.v4.jsonl.zstd
```

- 一级目录按工作区路径编码：分隔符连续段替换为 `-`，其余不安全字符转 `~XXXX`，
  去首连字符，两端补 `--`；
- 每行一条记录，字段 `{type, seq, time, data}`；
- 首条 `type: "session"`，含 `id`、`cwd`、`delegationDepth`、`agentPreset`；
- 文件是 **zstd 多帧**，逐帧解压才能读全（附录 C）。

### 8.2 工作区归属

会话归属于某个工作区（`sessions/` 下的一级目录），同一工作区的会话聚合在一起，便于查找与归档。
工作区的迁移、修复工具不属于核心；本机是否安装，见附录 A。

### 8.3 目标（goal）

`create_goal` 建立跨轮次目标；`update_goal` 支持暂停、恢复、编辑、完成、受阻。
活跃目标使宿主在轮次结束时自动续跑，状态随会话持久化。

### 8.4 记忆即上下文

Agent 没有独立于上下文的记忆：被压缩的细节不会自然想起，会话日志是唯一完整档案。

---

## 9. 已知局限

```
· 无法观察屏幕、浏览器 DOM、内存与进程状态——只能读文件、执行命令、抓取网页
· 记忆等于上下文：被压缩的细节需主动从文件或日志恢复
· 会自信地出错：按字段名改错区块、凭记忆写校验串、用不存在的命令
· 上下文计量来自宿主或插件，不是模型自省
· 压缩不可逆，且摘要由模型重写
· 工具结果随时可能被裁剪，长输出的全文未必还在
· 沙箱内的只读命令也可能失败：dsh --dump-config 等会重写 $DSH_HOME 下的文件，
  在 workspace-write 下报 EPERM（实测）
```

---

## 10. 附录：本机实测与复核

### A. 本机形态

| 项 | 值 |
|---|---|
| 版本 | `0.2.1-rc.2` |
| profile | `web`（官方 `dsh-base`、`dsh-web-app` + 6 个社区插件） |
| preset | `standard` |
| 模型 | `deepseek-account` / `deepseek-flash` / `reasoningEffort: high` |
| 窗口 / 输出预留 | `1,048,576` / `256,000` |
| 工具账目 | 核心 31 + 社区 6 = 37 |
| 工作区 | `/Users/inseeker/Documents/demo-projects` |

社区插件及其贡献：`dsh-context-truth`（占用率播报）、`dsh-plugin-vetting`（`plugin_vet`）、
`dsh-workspace-mover`（`mover_*`）、`dsh-mv-session`（`mv_session`）、
`dsh-session-delete`（会话删除）、`dsh-usage-dashboard`（用量面板）。

### B. 参数算例

```
触发点 = min(⌊1,048,576 × 0.8⌋, 1,048,576 − 256,000 − 65,536)
       = min(838,860, 727,040) = 727,040   （69.3% 窗口）
保留尾部 = ⌊(1,048,576 − 256,000) × 0.16⌋ = 126,812
```

这是算例，不是机制常量：改 `maxTokens`、`headroomTokens`、`thresholdRatio` 或换模型，
数字随之变化，公式不变。

### C. 复核命令

| 事实 | 复核方式 |
|---|---|
| 版本 | `dsh --version` |
| profile 与插件清单 | `cat ~/.dsh/profiles/web/package.json` |
| 核心/插件行 | `…/dsh-base/cordis.patch.yml`、`…/dsh-web-app/cordis.patch.yml`、`presets/*.patch.yml` |
| 窗口与输出预留 | 会话日志 `request/context`、`request/header` |
| 工具清单 | 会话日志 `request/header` 的 `tools` |
| 压缩参数 | preset 中 `compaction-basic` 的 `config` |

逐帧解压会话日志（Node 的 `zstdDecompressSync` 只解第一帧）：

```js
// node read-session.mjs <session.v4.jsonl.zstd>
import fs from 'node:fs'
import zlib from 'node:zlib'

const b = fs.readFileSync(process.argv[2])
const M = Buffer.from([0x28, 0xb5, 0x2f, 0xfd]) // zstd 帧魔数
const out = []
let i = b.indexOf(M)
while (i >= 0) {
  const j = b.indexOf(M, i + 4)
  out.push(zlib.zstdDecompressSync(b.subarray(i, j < 0 ? b.length : j)).toString())
  i = j
}
console.log(out.join(''))
```

---

> 说明：本文是某台机器的实测记录；数字随安装与配置变化，不作规范。
