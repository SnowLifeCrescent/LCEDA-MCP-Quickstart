# 常用工作流

## 场景 1：读取当前原理图

**工具**：`schematic_read`

1. 调用 `schematic_read`
2. 解析返回的 JSON：器件列表（位号、符号名、引脚网络）、网络连接关系、DRC 状态
3. 按需向用户输出电路功能描述、关键信号路径、电源网络分析

---

## 场景 2：全局审查原理图

**工具**：`schematic_review`

1. 调用 `schematic_review`
2. 分析完整的网表文本，覆盖所有原理图页面
3. 输出六类分析项（以 Markdown 表格呈现）：
   - ① 电路功能概述
   - ② 器件清单与选型合理性
   - ③ 电源方案分析
   - ④ 信号与连线检查
   - ⑤ 保护与可靠性分析
   - ⑥ 整体可用性评估

---

## 场景 3：器件选型 + 放置

**工具**：`component_select` → `component_place`

1. 根据用户需求确定需要搜索的器件关键词
2. 调用 `component_select(keyword)`，等待用户确认具体型号
3. 用户确认后记录返回的 `uuid` 和 `libraryUuid`
4. 所有器件确认完毕后，调用 `component_place(components=[...])`
5. 按顺序逐一下达放置指令，用户在 EDA 中完成手动放置
6. 电源/地符号（VCC/GND）需提示用户手动添加

---

## 场景 4：使用透传 API 执行定制操作

**工具**：`eda_context` → `api_index` → `api_search` → `api_invoke`

1. （可选）调用 `eda_context` 了解当前环境
2. （可选）调用 `api_index` 浏览 API 命名空间
3. **必做** 调用 `api_search(query)` 确认目标 API 的参数签名
4. 根据签名构造参数，调用 `api_invoke(apiFullName, args)`
5. 不得跳过 `api_search` 凭空猜测参数
