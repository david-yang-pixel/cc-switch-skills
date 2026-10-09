# MCP 问题排查 Checklist

MCP 是较新的功能，问题多集中在协议和连接层面：

1. **确认版本**：MCP 功能在 v3.5.0+ 引入，低版本不支持
2. **协议检查**：确认 MCP URL 是否以 `/sse` 或 `/mcp` 结尾（协议 fallback 会导致响应时间波动）
3. **超时排查**：MCP 调用链较长，涉及 Ingress → API → Plugin Daemon → MCP Server，每层都可能超时
4. **[企业版] Ingress 超时**：默认 Ingress 超时可能太短，需调整 annotation
5. **数据库存储**：MCP 相关数据存储在 `plugin_daemon` 库，查 `tool_installations` 表

## 常见 MCP 错误模式

| 错误 | 常见原因 |
|------|----------|
| 7s vs 30s 响应时间波动 | SSE/Streamable HTTP 协议 fallback |
| MCP 连接超时 | Ingress 超时配置过短、网络不通 |
| MCP 工具不可用 | MCP Server 未启动、URL 配置错误 |
| 数据写入失败 | plugin_daemon 数据库连接问题 |
