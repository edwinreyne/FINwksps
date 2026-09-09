# 信息架构

## 1. 组织原则

信息架构采用“**工作 → 事项 → 业务域 → 资产与治理**”四层，而非十个彼此孤立的菜单。Decision Case 是跨页面上下文容器；用户从 Cockpit、My Work、绩效、资源或 Agent 待批进入后，均落到同一 Case 详情并保留返回路径。

## 2. 全局导航

| 一级入口 | 目的 | 主要用户 |
| --- | --- | --- |
| 首页 | 角色化态势和优先级 | 所有人 |
| My Work | 我负责、参与、待复核/审批/执行/验证的队列 | 所有人 |
| Decision Cases | 全生命周期检索、组合视图和创建入口 | 财务团队、领导 |
| Performance & Forecast | 指标、复盘、预测周期及相关 Cases | FP&A、CFO |
| Resource Allocation | OPEX/HC/Capex 组合、申请及使用监督 | BP、经理、CFO |
| Actions & Value | 跨 Case 行动、例外和价值实现 | Owner、经理、CFO |
| Data & Evidence | 上传、数据集、映射、质量和血缘 | 分析人员、数据管理员 |
| Agent & Approvals | Agent Runs、待批准动作、授权和停止 | 审批人、经理、运维 |
| Knowledge | 已审核复盘、相似 Case 和决策记忆 | 财务组织 |
| Administration | 角色、范围、阈值、代理、连接器配置和审计 | 管理员、审计 |

移动到 Case 后，全局导航之外还显示面包屑、Case ID/状态/模拟标识、关注与分享入口。

## 3. Decision Case 详情结构

1. **Overview：**事项摘要、触发、Owner、参与者、期限、影响、当前质量门与下一步。
2. **Evidence：**来源、快照、质量、血缘、引用关系。
3. **Analysis：**事实、Insight、假设、驱动、置信度与人工复核。
4. **Options：**基准及替代方案、影响矩阵、敏感性、推荐和反方观点。
5. **Approval：**路由、职责分离、阈值、意见、条件和授权包。
6. **Execution：**Action、依赖、Agent Run、回执、阻塞与例外。
7. **Verification：**预期/实际价值、归因、验证证据和复盘。
8. **Timeline：**跨对象审计轨迹和版本差异。

右侧持久上下文栏显示参与者、关键日期、标签、关联 Cases/Actions 和下一步；高风险操作始终回到 Approval 上下文。

## 4. 页面连接关系

```text
CFO Cockpit ─┬─> Decision Case ─> Approval ─> Action ─> Verification ─> Knowledge
My Work ─────┤         ↑              │          │
Performance ─┤         │              └─> Agent Run / simulated receipt
Resources ───┘         │
Data & Evidence ───────┴─> Insight / Option
Actions & Value ─────────> Case and portfolio value views
Administration ──────────> policies, audit and access context (not business ownership)
```

仪表板卡片是“入口”而不是信息终点；任何 KPI 异常可新建/关联 Case，任何审批必须能查看 Case 的批准快照，任何价值结果必须回链授权与证据。

## 5. 角色默认首页

| 角色 | 默认首页 | 首屏重点 |
| --- | --- | --- |
| CFO / 财务领导 | Executive Cockpit | 待决策、风险热图、预测变化、资源组合、关键例外、价值实现 |
| FP&A / 绩效 | My Work（预测与复盘视图） | 当前周期、待校验数据、待复核洞察、预测发布、准确性验证 |
| Finance BP | My Work（业务范围） | 我的申请、补件、获批条件、行动、使用/价值偏差 |
| 财务经理 | My Work（团队视图） | 待分派、待复核、待审批、容量、SLA、升级事项 |
| 分析人员 | My Work（制作队列） | 上传校验、分析草稿、证据复核、模型/规则维护任务 |
| 管理员 | Administration | 配置变更、访问异常、连接器/Agent 策略状态 |
| 审计人员 | Administration / Audit | 只读检索、异常访问、证据包导出 |

用户可收藏视图，但默认首页不改变其权限。

## 5.1 第一版双工作面

第一版不为每个财务岗位分别制作完整首页，而是深入制作两个可按职责裁剪的工作面：

- **CFO Decision Cockpit：**围绕“需要决定什么、为什么、有哪些方案、决定后是否兑现”组织，突出重大 Case、风险、方案、审批、执行例外和价值评价。
- **Finance Workbench：**面向 FP&A、Finance BP、财务经理和分析人员，围绕“今天需要完成什么工作”组织，突出待办、工作流步骤、所需输入、交付物、截止时间、阻塞、上下游和执行方式。

角色差异不是换肤：CFO 主要消费经准备的决策上下文并承担审批；财务执行人员主要准备证据、复核分析、推进任务和验证结果。二者引用同一个 Decision Case 和同一组对象版本。

## 6. 查找、通知与状态

- 全局搜索按 Case ID/名称、业务、期间、类型、Owner、状态、风险和标签过滤；结果遵循权限。
- 通知分为需行动、风险升级、信息三档；支持从通知直达具体 Case 步骤。
- 统一状态词用于 Case、Approval、Action、Agent Run，各对象仍保留独立状态，避免用一个“进行中”掩盖真实情况。
- 每页固定显示数据截至时间、版本/币种/单位；模拟环境显示持续水印。
