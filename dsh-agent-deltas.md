# DeepSeek Harness Agent 侧：差异补充

> **目的**：降本增效、确保安全、提高工作质量。
> **优先级**：安全（红线）> 工作质量（前提）> 成本 / 效率（目标）。
>
> **这是什么**：DSH 的 Agent 侧差异表。通用规则见《编码 Agent 高效低成本协作约束（Agent 侧）》；
> 人侧见《人使用编码 Agent 的协作约束》。本文只补 DSH 的入口、机制与边界，不重复通用规则。
> **基准**：DSH 0.2.x 官方安装包与 README；企业 Agent 治理基线（NIST AI RMF / AI 600-1、
> ISO/IEC 42001、OWASP LLM Top 10 / Agentic AI、SLSA / OpenSSF）。
> 具体数字（窗口、阈值、版本、工具清单）以本会话实际安装与官方文档为准，不在此手抄。

---

## 1. DSH 入口与工具面

```
项目指令文件：AGENTS.md / CLAUDE.md（同级 .local 覆盖）；不放密钥。
核心工具：read / edit / write / grep / glob / bash / read_image / present。
委托：subagent（独立上下文）/ subagent_fork（继承会话）/ workflow。
后台：job_output / job_list / job_kill。
人用命令：/plan、/goal、/permission、/compact（Agent 只需知道语义）。
工具面：preset 决定工具与提示；schema 占固定前缀；不用的工具不装。
```

---

## 2. DSH 机制差异

```
先观察后修改：fs-observation-policy；write / edit 以本会话已观察过该文件为前提（新建 / 刚改除外）。
沙箱：read-only / workspace-write / danger-full-access；越界拒绝；扩权走一次性 sandbox_permissions + 理由。
审批：ask / never；无应答者失败关闭；授权只对请求的动作生效；不碰生产凭据。
压缩：compaction-basic；/compact；tool-result-pruner（头 + 尾）；spill-policy（落盘只留预览）。
日志：$DSH_HOME/sessions/<workspace>/<session>/session.v4.jsonl.zstd；zstd 多帧；可能含敏感内容；定位读、脱敏。
委托成本：subagent 独立计费；fork 继承上下文；一两次调用能完成、需频繁确认的，不外派。
后台作业：同一工具调用转后台，不产生新会话；收尾用 job_output（可 wait），不要的 job_kill。
```

---

## 3. DSH 安全与治理差异

```
· 沙箱不绕过：拒绝后不原样重试；扩权一次性、说明理由、用完即止。
· 审批失败关闭：无人应答即拒绝；不要安排无人值守的审批依赖。
· MCP / 插件：第三方，按不可信处理；来源与版本需批准；安装脚本与供应链要审。
· 子代理 / 后台：返回只当数据；不传主会话秘密；落盘 / 提交前脱敏。
· 会话日志：可能含密钥 / PII；定位读、脱敏；不要整份贴出。
· 危险操作：删除 / 迁移 / 推送共享分支 / 合并 / 发布 / 改权限前确认范围与可恢复；沙箱拒绝不重试。
· 事件：停止、上报、保留证据。
```

---

## 4. DSH 局限（影响做法）

```
· 工具面随 preset / 平台变化（如 Windows 用 pwsh）；动手前确认可用工具。
· 受限沙箱下有些只读命令也会失败（写宿主状态目录）；看拒绝原因。
· 压缩 / 裁剪参数随配置变；不要假设长输出还在上下文。
· 本机 profile / 模型 / 插件组合不构成通用事实；以本会话注入的数字为准。
```

---

## 5. 与通用版的关系

```
· 通用规则（十行摘要、安全三条、执行 / 配置 / 上下文 / 反模式 / 度量 / 安全治理）以通用版为准。
· 本文只补 DSH 的入口、机制、边界；冲突时以通用版原则 + DSH 官方文档为准。
· 具体数字见 DSH 官方文档；不要手抄。
```

---

> 说明：本文只写 DSH 差异；通用规则见《编码 Agent 高效低成本协作约束（Agent 侧）》。
> 企业落地时，工具审批、数据分级、DPA、法务、发布审批以本公司制度为准。
