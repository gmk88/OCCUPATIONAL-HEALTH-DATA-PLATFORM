# 职业健康安全管理数据中台 API 接口设计

| 字段 | 内容 |
|-----|------|
| 文档版本 | V1.0 |
| 编写日期 | 2026-11-19 |
| 依据 | api-rules.md、产品架构设计.md、数据库设计.md、PRD.md |
| 风格 | RESTful HTTP API（JSON） |
| Base URL | /api/v1 |

## 1. 通用约定（遵循 api-rules）

### 1.1 路径与方法

- 路径使用小写英文、名词表达资源，如 `/enterprises`、`/tasks`，不含糊命名、无必要动词。
- HTTP 方法语义：GET 读取、POST 创建、PATCH 部分更新、PUT 整体替换、DELETE 删除。
- 路径参数标识资源（如 `/tasks/{id}`），查询参数用于筛选/排序/分页，请求体用于提交数据。
- 敏感信息（密码、令牌）不得放入 URL 查询参数，只走请求体或 Header。

### 1.2 参数校验

- 每个接口明确参数名称、类型、是否必填、默认值、格式与校验规则（见各接口说明）。
- 服务端必须校验所有客户端输入，不能只依赖前端。
- 字符串长度、数值范围、枚举值、日期格式、数组大小均设合理限制。

### 1.3 统一响应格式

成功：

```json
{ "code": "0", "message": "success", "data": { } }
```

失败：

```json
{ "code": "E1001", "message": "参数校验失败", "data": null, "errors": [ { "field": "credit_code", "reason": "格式不正确" } ] }
```

- 字段命名与类型同类接口保持一致。
- 响应只含调用方需要的数据，不暴露数据库内部字段或敏感信息。
- 列表接口统一分页结构，页码从 1 起，默认每页 20 条，最大 100 条：

```json
{ "code": "0", "message": "success", "data": { "page": 1, "page_size": 20, "total": 156, "list": [ ] } }
```

### 1.4 HTTP 状态码与业务错误码

- 不得将所有错误返回为 HTTP 200；HTTP 状态码与业务 code 不矛盾。
- 状态码：200 成功、201 创建成功、204 无内容、400 参数错误、401 未认证、403 无权限、404 不存在、409 冲突、422 业务规则校验失败、500 服务端错误。
- 业务错误码稳定可识别，统一命名：E1xxx 参数/认证、E2xxx 权限、E3xxx 业务规则、E9xxx 系统。
- 错误响应提供足够处理信息但不泄露内部实现；不得仅凭 HTTP 200 判断业务成功，须同时校验 code。

| 错误码 | 含义 |
|-------|------|
| E1001 | 参数校验失败 |
| E1002 | 验证码错误 |
| E1003 | 账号或密码错误 |
| E2001 | 无权限访问 |
| E2002 | 越权访问（数据隔离拦截） |
| E3001 | 不符合建档条件（硬拦截） |
| E3002 | 状态流转不合法 |
| E3003 | 采样/检测数据异常（软预警，业务继续） |
| E9001 | 服务端错误 |

### 1.5 认证与数据隔离

- 除登录接口外，所有接口须在 Header 携带 `Authorization: Bearer <JWT>`。
- JWT 载荷含 user_id、terminal_type、role、region_codes（辖区集合）、enterprise_id、institution_id。
- 服务端统一鉴权；MyBatis-Plus 拦截器按 JWT 中的辖区与业务标识自动注入数据隔离条件（见数据库设计第 9 节）。
- 前端路由守卫仅做导航控制，真正鉴权在服务端（宪法 5-10）。

## 2. 接口清单

| 模块 | 接口 | 方法 | 路径 | 说明 |
|-----|------|------|------|------|
| 认证 | 登录 | POST | /auth/login | 三端统一登录，签发 JWT |
| 认证 | 刷新令牌 | POST | /auth/refresh | 刷新 JWT |
| 认证 | 验证码 | GET | /auth/captcha | 获取图形验证码 |
| 组织权限 | 行政区划树 | GET | /regions | 三级区划树 |
| 组织权限 | 用户列表 | GET | /users | 分页 |
| 主数据 | 企业列表 | GET | /enterprises | 分页，按辖区隔离 |
| 主数据 | 企业详情 | GET | /enterprises/{id} | 含隔离校验 |
| 主数据 | 危害因素列表 | GET | /hazard-factors | 分页 |
| 规则库 | 方案版本列表 | GET | /solution-versions | — |
| 规则库 | 表单配置详情 | GET | /form-schemas/{id} | 返回 Schema JSON |
| 规则库 | 保存表单配置 | POST | /form-schemas | 管理端配置 |
| 任务 | 任务列表 | GET | /tasks | 分页，按辖区隔离 |
| 任务 | 创建任务 | POST | /tasks | 管理端 |
| 任务 | 任务拆分 | POST | /tasks/{id}/dispatch | 按区划拆分 |
| 流程 | 流程详情 | GET | /processes/{id} | 当前状态节点 |
| 流程 | 触发流转 | POST | /processes/{id}/transitions | 提交事件，驱动状态机 |
| 流程 | 流转日志 | GET | /processes/{id}/logs | 追溯 |
| 遴选 | 提交核准 | POST | /screenings | 执行硬拦截/软预警校验 |
| 调查 | 提交调查表 | POST | /surveys | 按 Schema 提交 |
| 采样 | 提交采样记录 | POST | /samplings | 点位/时长校验 |
| 检测 | 提交检测数据 | POST | /testings | 超标判定与谱图上传 |
| 帮扶 | 新增帮扶记录 | POST | /supports | — |
| 整改 | 提交整改材料 | POST | /rectifications | 触发复核 |
| 整改 | 复核 | PATCH | /rectifications/{id}/review | 通过/不通过回放 |
| 上报 | 数据上报 | POST | /reports | — |
| 抽查 | 抽查比例配置 | PUT | /quality/ratio-config | 管理端 |
| 抽查 | 生成抽检清单 | POST | /quality/samples | 按比例生成 |
| 抽查 | 登记抽查结果 | PATCH | /quality/checks/{id} | — |
| 档案 | 档案列表 | GET | /enterprises/{id}/files | 按类型/脱敏返回 |
| 档案 | 上传资料 | POST | /files | 返回 OSS 地址 |
| 看板 | 综合指标 | GET | /dashboard/overview | 按辖区聚合 |
| 看板 | 地域进度 | GET | /dashboard/regions | — |
| 看板 | 超标统计 | GET | /dashboard/exceedance | — |
| 看板 | 一键报告 | POST | /dashboard/reports | 异步生成初稿 |

## 3. 核心接口详细定义

### 3.1 POST /auth/login

- 请求体：`username`（必填，字符串，1-64）、`password`（必填，字符串，1-128）、`captcha`（必填，4 位）、`captcha_key`（必填）。
- 响应 200：`data` 含 `token`、`expires_in`、`user_info`（terminal_type、region_codes、role）。
- 错误：400 参数错误（E1001）、验证码错误（E1002）、账号密码错误（E1003）。

### 3.2 GET /enterprises

- 查询参数：`page`（默认 1）、`page_size`（默认 20，最大 100）、`keyword`（可选）、`industry_code`（可选）、`status`（可选）。
- 响应 200：统一分页结构；服务端按 JWT 辖区集合自动过滤 `region_code`。
- 错误：401 未认证、403 无权限（E2001）。

### 3.3 POST /screenings

- 请求体：`process_id`（必填）、`credit_code`（必填，18 位）、`industry_code`（必填）、`scale_type`（必填，枚举 1-4）、`materials`（数组，最多 20 项）。
- 处理：按规则库校验——硬拦截返回 422 + E3001 + 不符合项清单；软预警返回 200 + code=0 + warnings 数组，业务继续。
- 响应：成功 201；硬拦截 422；状态不合法 409 + E3002。

### 3.4 POST /samplings

- 请求体：`process_id`、`point_code`、`factor_code`、`sample_duration`（数值，>0）、`required_duration`（来自规则库）。
- 处理：时长/点位不达标时软预警，返回 `is_abnormal=true` + 预警信息，记录留痕并进入复核队列，不阻断提交（宪法 5.3 / PRD 5.3）。
- 响应：201 创建成功；参数错误 400；数据异常软预警 200 + warnings。

### 3.5 POST /processes/{id}/transitions

- 请求体：`event`（必填，枚举：approve/submit_survey/submit_sampling/submit_testing/submit_rectify/review_pass/review_reject/report/quality_finish/archive）。
- 处理：状态机校验当前状态是否允许该事件；`review_reject` 时状态回退到 `supporting`（帮扶中），其余按状态字典前进。
- 响应：200 含 `from_state`、`event`、`to_state`；非法流转 409 + E3002。

### 3.6 PATCH /rectifications/{id}/review

- 请求体：`result`（必填，枚举 pass/reject）、`comment`（可选）。
- 处理：pass 推进状态；reject 回退到帮扶节点，企业可重新提交（submit_count 递增，保留历史版本）。
- 响应：200；无权限 403；状态不合法 409。

### 3.7 GET /enterprises/{id}/files

- 查询参数：`file_type`（可选，枚举见数据库设计 8.5）。
- 处理：按 JWT 校验企业可见性；`sensitivity=2`（体检资料）脱敏返回、分级授权。
- 响应：200 文件索引列表（含 oss_key、version、retain_until）；越权 403 + E2002。

### 3.8 POST /quality/samples

- 请求体：`source_level`（必填，枚举 province/city/county）、`ratio`（可选，覆盖配置）。
- 处理：按各级比例（省 5%/市 8%/县 12%，可配置）从已上报数据随机抽取，生成清单，结果可追溯。
- 响应：201 含生成的抽检清单；无权限 403。

### 3.9 GET /dashboard/overview

- 查询参数：`region_code`（可选，默认取 JWT 辖区）。
- 处理：按辖区聚合指标（企业数、任务进度、超标数、帮扶完成率、体检完成率、抽查覆盖率），走 Redis 预聚合。
- 响应：200 指标对象；越权 403。

### 3.10 POST /dashboard/reports

- 请求体：`report_type`（必填，enum summary/risk）、`region_code`、`period`。
- 处理：异步生成帮扶总结或风险评估初稿，返回任务 ID 供轮询。
- 响应：202 含 `task_id`；生成完成后经查询接口获取结果。

## 4. 与数据库 / 模块的一致性约束

- API 资源路径与数据库表、产品架构模块一一对应（如 `/processes` 对应 `biz_process`、`/screenings` 对应 `biz_screening`）。
- 状态机事件枚举与数据库设计第 8.2 节状态节点、产品架构 4.1 节流转定义保持一致。
- 隔离字段（`region_code`、`enterprise_id`、`institution_id`）由服务端拦截器注入，接口调用方不得通过请求参数覆盖（宪法 2-4、api-rules 6）。
- 设计变更时须同步更新本文档、数据库设计、产品架构设计与 PRD（宪法 2-5）。

## 5. 非功能约定

- 性能：常规查询 ≤ 500ms；看板走预聚合；报表导出与一键报告异步。
- 安全：服务端强制鉴权与数据隔离；防 SQL 注入/XSS/CSRF；错误响应不泄露内部细节；全量关键操作留痕。
- 幂等：提交类接口支持幂等键（请求头 `Idempotency-Key`），防止重复提交导致数据异常（宪法 6-25）。

---

*文档结束*
