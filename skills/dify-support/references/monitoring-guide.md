# 监控排查指南（SigNoz / Cloudflare）

## 目录

- 平台级故障与影响范围
- SigNoz MCP 快查
- Cloudflare 排查场景

## 平台级故障与影响范围（仅 SaaS）

当同一时间段出现多个客户报告、5xx/超时或服务不可用时，先证明客户样本，再通过 OpenConnector `dify-signoz` 按 tenant、deployment、service、operation 和执行池聚合，并对相同维度的邻近基线窗口做对比。只在边缘路径确有证据时追加 OpenConnector `cloudflare_mcp/default` 查询；不要使用本地 CLI、Grafana 页面或其他监控 UI 作为调查来源。

如 OpenConnector 连接或已采集遥测未暴露事故、Kubernetes、队列或容量所需事实，记录准确的 owner-only gap，并交给 SRE/owner 核验，不以缺失数据推断平台状态。

## SigNoz MCP 快查（仅 SaaS）

**唯一入口**：OpenConnector `custom_mcp` provider 的 `dify-signoz` 连接。先用 `custom_mcp.list_tools` 读取当前工具面；旧任务可能缓存旧 schema，必须以实时发现为准。连接、认证或动作不可用时记录准确缺口，不直连远端 MCP，不用本地 `sz`、旧 `signoz-mcp-server` 或 Web UI 兜底。

**结构化能力**：

| 工具 | 用途 |
|------|------|
| `signoz_list_services` | 发现实际 service / deployment 名称 |
| `signoz_get_field_keys` / `signoz_get_field_values` | 发现字段和值，避免猜 tenant、environment 或 K8s 标签 |
| `signoz_search_logs` / `signoz_aggregate_logs` | 日志明细、计数、分组和时间序列 |
| `signoz_search_traces` / `signoz_aggregate_traces` | 查错误/慢 span，计算数量和 P50/P95/P99 |
| `signoz_get_trace_details` | 按 trace ID 还原完整 span 链路 |
| `signoz_list_metrics` / `signoz_query_metrics` | 发现并查询 Pod/deployment 指标 |
| `sz_run` | 结构化工具无法表达的高级只读 ClickHouse 查询 |

**常用表**：

| 表 | 用途 |
|----|------|
| `signoz_logs.distributed_logs_v2` | 日志正文、级别、trace_id/span_id、attributes/resource |
| `signoz_traces.distributed_signoz_index_v3` | span 明细、服务、操作名、耗时、HTTP/DB/Redis 维度 |
| `signoz_traces.distributed_signoz_error_index_v2` | trace error、exception message、stacktrace |
| `signoz_traces.distributed_dependency_graph_minutes_v2` | 服务依赖调用量、错误量、错误率 |
| `signoz_metrics.distributed_samples_v4` / `distributed_time_series_v4` | HTTP duration/count、plugin invocation、DB latency、system/K8s/OTel 指标 |

**排查习惯**：
- 平台级判断先证明客户样本，再按 tenant/deployment/operation 聚合，并用相同维度对比邻近基线窗口；同一时间出现只能证明相关，不能证明同一根因。
- 结构化工具优先走“服务/字段发现 → 聚合判范围 → 明细/Trace → 指标验证”，不要一上来写大 SQL。
- 查"最近有没有日志"时，同时查 logs / traces / metrics 的最新时间；可能出现 logs ingestion 停了但 traces/metrics 仍持续写入。
- 过滤后返回空时，不要直接下结论；先发现字段和值，核对服务/deployment、时间转换和信号新鲜度，再决定是否用 `sz_run` 查表。
- 大查询必须带时间范围和 `LIMIT`。日志时间是 ns，需要 `fromUnixTimestamp64Nano(timestamp)`；trace `timestamp` 是 `DateTime64`，可直接 `timestamp >= now() - INTERVAL ...`。
- 需要按 tenant/app/workflow/conversation 查 trace 时，先查 `signoz_traces.distributed_tag_attributes_v2` 找实际 tag key，常见有 `tenant_id`、`dify.tenant_id`、`dify.app_id`、`dify.workflow_id`、`messaging.conversation_id`。

**SQL 模板**：

```sql
-- Logs 最新写入时间
SELECT count() AS logs, max(fromUnixTimestamp64Nano(timestamp)) AS last_log
FROM signoz_logs.distributed_logs_v2
WHERE timestamp >= toUnixTimestamp64Nano(now64(9) - INTERVAL 1 HOUR);

-- Trace 服务耗时
SELECT serviceName, count() AS spans,
       quantile(0.95)(duration_nano)/1000000 AS p95_ms,
       quantile(0.99)(duration_nano)/1000000 AS p99_ms
FROM signoz_traces.distributed_signoz_index_v3
WHERE timestamp >= now() - INTERVAL 1 HOUR
GROUP BY serviceName ORDER BY spans DESC LIMIT 20;

-- 服务依赖错误率
SELECT src, dest, sum(total_count) AS calls, sum(error_count) AS errors,
       round(errors / calls, 4) AS error_rate
FROM signoz_traces.distributed_dependency_graph_minutes_v2
WHERE timestamp >= now() - INTERVAL 1 HOUR
GROUP BY src, dest HAVING calls > 0 ORDER BY errors DESC LIMIT 20;
```

## Cloudflare 排查场景（仅 SaaS）

只使用 OpenConnector `cloudflare_mcp/default` 的只读路线；不使用 Wrangler、Cloudflare Dashboard、直连 Cloudflare MCP 或其他别名兜底。仅在证据链涉及 edge/CDN/WAF/DNS/TLS、HTTP analytics 或 Workers observability 时进入该路线，并与 SigNoz 的源站证据关联。具体输入、查询边界和空结果解释见 [Cloudflare route](cloudflare-mcp.md)。
