# Common Issues Quick Reference

## SaaS 常见问题

### 1. 文档卡在 Queued 状态
- **原因**: `indexing_status = "waiting"`，Celery 任务未消费
- **SQL 排查**:
  ```sql
  SELECT id, name, indexing_status, is_paused, error, created_at
  FROM documents WHERE id = '<document_id>';
  ```
- **解决**: 后端强制重置状态或删除文档

### 2. 知识库自动被禁用
- **原因**: 定时任务 `clean_unused_datasets_task` 自动禁用 30 天内未更新且未被查询的文档
- **代码**: `api/schedule/clean_unused_datasets_task.py`
- **配置**: `PLAN_SANDBOX_CLEAN_DAY_SETTING=30`, `PLAN_PRO_CLEAN_DAY_SETTING=7`
- **解决**: 用户手动重新启用文档

### 3. Knowledge Data Storage 删除后未释放
- **原因**: 当前用量来自订阅信息接口 / billing 缓存 / 向量后端存储计数，可能不是实时刷新；Postgres 里能直接查到的是配额上限，不是当前已用 MB
- **SQL 排查**:
  ```sql
  -- billing DB: 查配额上限，不是当前用量
  SELECT
      tss.tenant_id,
      tss.plan,
      tss.interval,
      tss.expiration_date,
      tsd.vector_space AS vector_space_limit_mb,
      tsd.document_upload_quota,
      tsd.knowledge_rate_limit
  FROM tenant_subscription_settings tss
  JOIN tenant_subscription_details tsd
      ON tsd.tenant_subscription_setting_id = tss.id
  WHERE tss.tenant_id = '<workspace_id>'
  ORDER BY tss.created_at DESC
  LIMIT 1;
  ```
- **客户侧确认**: 让客户打开 `Settings -> Billing -> Knowledge Base Data Storage` 查看页面显示的当前用量。
- **处理**: 如果页面用量明显低于上限但仍返回 403，再检查 quota counter / billing cache / vector backend storage counter 是否不一致；不要在确认前直接建议删除文档或升级。

### 4. Agent 节点调用 Workflow 工具报 PluginNotFoundError
- **原因**: Agent → Workflow tool registry 同步问题
- **代码**: `api/core/workflow/nodes/agent/agent_node.py`, `api/core/tools/tool_manager.py`
- **解决**: 在 Agent 节点中重新编辑 Workflow 工具触发数据刷新

### 5. Web App 访问权限控制
- **处理**: 先通过当前官方文档、飞书正式 SaaS 知识、实际计划权益和入口核实密码保护/SSO/访问控制能力，不从历史版本或 Enterprise 行为推断 SaaS。
- **代码**: `web/models/access-control.ts` — AccessMode 枚举
- **替代方案**: 使用 API 模式自建认证层

### 6. UPDATE 文档返回 400 "Document is not available"
- **原因**: `display_status != "available"`（文档未完成索引、已禁用或已归档）
- **代码**: `api/services/dataset_service.py:2188-2189`
- **SQL 排查**:
  ```sql
  SELECT id, indexing_status, enabled, archived FROM documents WHERE id = '<document_id>';
  ```

## 企业版常见问题

### 1. Trigger Workflow 时 Code 节点报错 sandbox 连接失败
- **错误**: `Failed to execute code... Name or service not known`
- **原因**: Worker 服务无法解析 sandbox 服务主机名（DNS 问题）
- **代码**: `api/core/helper/code_executor/code_executor.py`
- **关键配置**: `CODE_EXECUTION_ENDPOINT` 需要 API 和 Worker 都能访问
- **解决**: 确保 Worker 和 Sandbox 在同一网络，或使用 IP 替代主机名

### 2. 内网环境无法安装插件
- **原因**: 插件下载可能依赖 `cloud.dify.ai`
- **解决方案**:
  - 方案 1: 开放 `cloud.dify.ai` 访问
  - 方案 2: 本地文件安装（`.difypkg` / `.difybndl`）
  - 方案 3: 配置 `MARKETPLACE_API_URL` 指向内部代理

### 3. 嵌入式聊天组件移动端体验问题
- **问题**: 虚拟键盘弹出后聊天窗口被挤压
- **代码**: `web/public/embed.js`, `web/app/components/base/chat/embedded-chatbot/index.tsx`
- **现状**: 暂无移动端全屏模式，已作为 feature request 提交
