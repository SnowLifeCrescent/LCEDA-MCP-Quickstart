# MCP 工具目录

## 基础工具（始终可用）

| 工具 | 用途 | 参数 |
|------|------|------|
| `schematic_read` | 读取当前原理图页面的完整电路语义快照（器件、引脚网络、DRC） | `{}` |
| `schematic_review` | 读取全工程所有原理图页面的网表文件，适合全局审查、BOM 核查 | `{}` |
| `component_select` | 在 EDA 系统库中搜索候选器件，返回给用户确认 | `keyword`（必填）, `limit`（2-20，默认 20） |
| `component_place` | 引导放置已确认的器件列表 | `components`（必填，uuid + libraryUuid）, `timeoutSeconds`（30-180，默认 60） |

## 透传 API 工具（需开启开关）

需在 mcp-hub 设置中开启 `exposeRawApiTools` 后暴露：

| 工具 | 用途 | 参数 |
|------|------|------|
| `api_index` | 列出所有可用 EDA API 模块名称 | `owner`（可选） |
| `api_search` | 按关键词搜索具体 API 方法及其参数说明 | `query`（必填）, `scope`, `owner`, `limit` |
| `eda_context` | 读取当前 EDA 页面上下文（文档类型、工程信息、选中图元） | `scope`（可选） |
| `api_invoke` | 调用任意 EDA API 并透传结果 | `apiFullName`（必填）, `args`, `timeoutMs` |

## 注意事项

- `schematic_read` 仅覆盖当前活动页面，审查多页电路请用 `schematic_review`
- `schematic_review` 会返回完整网表文本，适合 AI 做全局分析
- 电源符号（VCC/GND 及其变体）禁止通过 `component_select` / `component_place` 搜索或放置，需要用户在 EDA 中手动添加
- `component_select` 调用后需等待用户确认器件（侧边栏交互），取消或跳过不会重试
- 开启透传 API 工具后，AI 调用 `api_invoke` 前必须先通过 `api_search` 确认参数签名
