# MWStudio 技术准备清单

这份清单根据 `MWS_PRD_v03.docx` 整理，用于开发前的工程、CloudBase 和安全准备。当前不实现业务功能。

## 1. 必须先确认的决策

- 确认 CloudBase 环境 ID、地域、套餐和是否使用已有共享环境。
- 确认 GitHub 组织或个人账号，以及远程仓库是否公开。
- 确认正式域名、测试域名和 CloudBase 静态托管域名策略。
- 确认账号导入格式，以及用户名密码由谁创建、重置和停用。
- 确认 Workbuddy 发布 API 的调用方、Key 生命周期和初始 scope。

## 2. CloudBase 资源准备顺序

1. 完成 CloudBase CLI/MCP 登录，并固定显式 `envId`。
2. 启用用户名密码认证；先不开放匿名登录。
3. 建立带 `MWS_` 前缀的文档集合和索引：
   `MWS_users`、`MWS_classes`、`MWS_class_members`、`MWS_tasks`、
   `MWS_task_versions`、`MWS_assignments`、`MWS_assignment_targets`、
   `MWS_task_records`、`MWS_record_events`、`MWS_api_keys`。
4. 规划云存储路径：`MWS/tasks/{taskId}/{versionId}/index.html`。
5. 预留函数边界：`MWS_taskRuntime`、`MWS_workbuddyApi`、`MWS_aiProxy`。
6. 设置最小权限、CORS、备份策略和日志保留策略。

## 3. 安全与边界

- Task iframe 不持有 CloudBase Auth token，只通过 Host Bridge / `postMessage` 使用白名单能力。
- Workbuddy Key 只展示一次，服务端只保存 hash；支持 Rotate、Revoke、Last Used 和 scope。
- 所有 Workbuddy 请求校验 Content-Type、payload 大小、scope、速率和审计字段。
- `MWS_aiProxy` 只允许服务白名单，禁止任意 URL fetch，防止 SSRF。
- 任何 Secret 只存在云函数运行环境，不进入 HTML、浏览器 bundle 或 `.env` 提交内容。
- 学生只能访问自己的 Assignment/TaskRecord；教师与管理员权限分层，Preview 数据用 `isTest=true` 隔离。

## 4. 数据与可靠性准备

- `MWS_task_versions` 不可变，保存 `contentHash`；Assignment 必须引用明确版本。
- `MWS_task_records` 以 `assignmentId + studentId` 做唯一约束。
- 自动保存采用 300–800ms debounce、patch 合并、`idempotencyId`、`revisionNo` 和 sessionId。
- 浏览器侧准备 IndexedDB pending queue；`pagehide` / `visibilitychange` 尝试 flush。
- 规划索引：assignment/status、student/lastActiveAt、task/createdAt、class/user、record/serverAt。
- 核心集合定期备份；任务 HTML 使用版本路径保留历史。

## 5. GitHub 与工程准备

- 当前目录已初始化为独立 Git 仓库；远程 GitHub 仓库需在账号授权后创建并绑定。
- 分支建议：`main` 作为受保护稳定分支，开发使用短生命周期 feature 分支。
- 首批 CI 只做安装、类型检查、lint、build 和敏感信息扫描；暂不部署。
- 后续应用骨架再建立 React/Vite、CloudBase 初始化模块、类型定义和测试目录。

## 6. 推荐的第一条端到端 Contract Test

先用一个最小单文件 HTML 验证：`MWS.ready()` → `getState()` → `saveState()` → `track()` → `complete()`；验证断网重连、关闭页面恢复、幂等重试、版本不可变和教师侧读取。通过后再做教师首页和班级 Matrix。

