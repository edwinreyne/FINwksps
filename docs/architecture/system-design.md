# 系统设计

## 1. 设计目标与边界

系统是现有财务数据/报表/审批/协作工具之上的决策控制面。原型只验证产品体验和控制语义，使用本地合成数据与模拟连接器；以下生产设计是目标边界，不代表已经建设或验证。

## 2. 概念架构

```text
[Application Layer]
Cockpit | My Work | Decision Case | Performance | Resources | Actions/Value | Admin
                              |
[Controller / Control Plane]
Case workflow | Orchestration | Policy/SoD | Approval & Authorization | SLA
Agent run control | Confidence/quality gates | Stop/circuit breaker | Audit
           |                                      |
[Model Gateway]                         [Skills / MCP / Connectors]
model routing, prompt/version,          read adapters | approved write tools
output schema, safety checks            data lake/DW/SAP/OA/files/notifications
           \                                      /
[Domain & Data Layer]
Case store | evidence/lineage | semantic metrics | object/file store | audit ledger | knowledge index
```

信任边界：浏览器—应用 API、Controller—Model、Controller—Connector、平台—企业系统、业务数据—审计/运维数据。任何模型输出均视为不可信内容，必须经结构校验、策略检查和必要的人审。

## 3. 主要模块

| 模块 | 职责 |
| --- | --- |
| Workspace UI/BFF | 角色化聚合、Case 上下文、权限裁剪；不自行决定授权 |
| Case Service | 生命周期、版本、参与者、类型模板和质量门 |
| Evidence & Lineage | 上传、快照、校验、映射、来源及引用图 |
| Analytics/Scenario | 指标、差异、驱动、情景、影响计算；保留方法版本 |
| Approval & Authorization | 路由、阈值、SoD、代理、条件批准、授权签发/撤销 |
| Action & Value | 任务、里程碑、SLA、例外、价值基线与验证 |
| Agent Controller | 计划/任务编排、依赖、短期上下文、置信度检查、预算、停止与熔断 |
| Model Gateway | 模型抽象、输入输出约束、版本、内容安全和可观测性 |
| Connector Gateway | 标准读写契约、白名单、幂等、限流、凭据隔离及回执 |
| IAM/Policy | 企业身份映射、RBAC + 事项/属性范围、动态策略 |
| Audit/Memory | 追加事件、证据包、经审核知识、保留与法律留置接口 |

## 4. 关键数据流

### 4.1 数据到决策

1. Connector/File Adapter 接收数据，记录来源与模拟/生产标识。
2. 隔离区进行恶意文件检查（生产）、格式/字段/口径校验；失败数据不发布。
3. 生成版本化 Dataset 和 Evidence 快照、质量结果及 lineage。
4. Analytics 或 Agent 读取获准范围，生成结构化 Insight/Option；Controller 校验引用、置信度和限制。
5. 人工复核形成 Case 版本并提交 ApprovalRequest。

### 4.2 授权到执行

1. Policy Engine 根据事项类型、金额、风险、组织范围和 SoD 产生路由。
2. 合格审批人对固定快照作出决定；批准签发细粒度 Authorization。
3. Action/Agent Run 请求执行时再次检查主体、动作、对象、额度、有效期与停止状态。
4. Connector 使用服务身份调用目标；业务用户/模型看不到凭据。
5. 回执、失败与幂等键写入审计；失败达到阈值熔断并等待人工决定。

### 4.3 结果到记忆

结果 Evidence → Verification 对比锁定基线 → 人工确认归因 → 关闭 Case → 审核发布 DecisionMemory。实际与预期差异不能覆盖原推荐。

## 5. 标准化集成边界

### 5.1 读取契约（概念）

请求包含 `source, dataset, schema_version, scope, as_of/period, filters, purpose, case_id, requester`；响应包含 `data/reference, source_version, extracted_at, quality, lineage, classification, mock_flag`。Connector 不把上游字段直接泄漏为稳定领域契约，先映射到规范指标/维度。

### 5.2 执行契约（概念）

请求包含 `authorization_id, action_type, target, parameters, idempotency_key, expected_precondition, initiated_by, case/action_id`；响应包含 `accepted/status, external_reference, executed_at, result_summary, reconciliation_hint`。生产写连接器必须默认关闭，逐工具启用。

### 5.3 原型适配器

- `MockDataLake/MockWarehouse`：读取合成 Actual/Budget/Forecast、业务驱动和主数据。
- `MockFileUpload`：CSV/XLSX 样例及刻意错误版本。
- `MockSAP/MockOA/MockNotification`：仅生成显著标记的预览/回执，不发起网络调用。

## 6. 安全与权限

- 身份：生产对接企业 IdP、MFA 和生命周期；服务/Agent 身份独立且短期凭据化。
- 授权：RBAC 提供动作基线，ABAC/事项参与关系限制业务、法人、基地、成本中心、数据分类、金额与风险；默认拒绝。
- SoD：申请、最终批准、执行及验证的冲突矩阵按事项类型配置；紧急例外需双人批准、限时且强审计（未来政策待定）。
- 敏感数据：传输/存储加密、字段/行级控制、脱敏、水印、受控导出；管理员与业务访问分权。
- Agent：最小输入、工具白名单、参数约束、调用预算、超时、停止开关、沙箱/网络出口控制、提示注入防护和人工质量门。
- 凭据：只由 Connector 运行时从企业密钥服务获取，不进入提示、Case、日志或前端。

## 7. 审计与可观测性

- 业务审计记录 Case/证据/建议/审批/授权/执行/验证/知识事件及理由。
- 技术遥测记录 correlation ID、延迟、错误、策略拒绝、模型/连接器版本和成本，但避免敏感正文。
- 关键事件追加写、时间同步、哈希校验；生产可采用 WORM/签名和独立安全域，具体方案待评估。
- 审计证据包包含时间线、对象版本、来源清单、审批链、授权、工具回执和验证，不默认包含超范围数据。

## 8. 原型与生产差异

| 方面 | 原型 | 未来生产 |
| --- | --- | --- |
| 数据 | 合成小规模数据，持续水印 | 数据湖/DW/SAP 等，分类与质量 SLA |
| 身份 | 虚构角色切换 | 企业 SSO/MFA、组织同步、PAM |
| 集成 | 本地 Mock，无真实网络写入 | 受控连接器、私网、密钥、双向对账 |
| Agent | 预设或受控生成、模拟工具 | 模型网关、评测、监控、白名单 L2 |
| 存储 | 原型存储，可重置 | 高可用、备份、灾备、驻留与保留 |
| 安全 | 控制体验演示 | 威胁建模、渗透、DLP、SOC、合规评审 |
| 性能 | 小数据体验目标 | 容量、并发、RTO/RPO/SLA 经验证 |
| 部署 | 不代表生产拓扑 | 企业内部或受控企业环境，方案待选 |

## 9. 风险与缓解

- **错误建议：**证据引用、置信度门槛、反方观点、人审和离线评测。
- **提示注入/恶意文件：**内容隔离、解析安全、工具与模型权限分离。
- **授权漂移：**短期授权、执行时复查、撤销与策略版本记录。
- **口径不一致：**语义指标目录、版本、数据 Owner 与质量状态。
- **自动化偏见：**展示替代方案和限制，记录接受/拒绝原因，定期回看结果。
- **价值重复计算：**Action 主 Case 与价值归属规则，财务验证。
