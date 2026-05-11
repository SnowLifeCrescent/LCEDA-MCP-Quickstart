---
name: LCEDA-MCP-Quickstart
description: "ONLY use when explicitly invoked via skill tool named 'LCEDA-MCP-Quickstart'. Not auto-triggered."
---

# LCEDA-MCP-Quickstart

通过 MCP 协议远程操控嘉立创 EDA 专业版的快速启动技能。

## 入口流程

执行以下步骤，按结果分叉。

### 步骤 1：首次使用检测

检查本地是否已配置 mcp-hub 运行时：

```
~/.opencode/jlceda-mcp-hub/mcp-hub/out/server/runtime.js
~/.config/systemd/user/jlceda-mcp-hub.service
```

两个文件都存在 → 跳至步骤 2。
任一缺失 → 输出以下提示并加载 `references/prerequisites.md`：

> 疑似初次尝试链接立创 EDA，即将准备配置前置需求。

执行完 prerequisites.md 后回到步骤 1 重新检测。

### 步骤 2：加载技能说明

加载 `references/skill-info.md`，输出技能概述与连接地址。

### 步骤 3：启动服务

```bash
systemctl --user start jlceda-mcp-hub.service
```

等待就绪（轮询 `curl -s http://127.0.0.1:7655/mcp -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'`，直到返回 HTTP 200）。

### 步骤 4：测试 EDA 连接

调用 `schematic_read` 工具：

- **成功返回数据** → "与嘉立创 EDA 连接正常"，继续。
- **返回桥接错误** → 输出：

> 与嘉立创 EDA 的桥接连接测试未通过，可能的原因：
> 1. 嘉立创 EDA 未打开，或不在原理图/PCB 页面 → 请打开 EDA 并进入页面后重试
> 2. 未安装 MCP Bridge 扩展 → 请在 EDA 扩展管理器中搜索 "MCP Bridge" 并安装
> 3. 桥接地址不匹配 → 当前服务端 WebSocket 地址为 `ws://127.0.0.1:8765/bridge/ws`
>    请在 EDA Bridge 设置页核对地址。如果端口不是 8765，请告知端口号，我来修改配置文件。
>
> **端口修改方式**：
> 编辑 `~/.config/systemd/user/jlceda-mcp-hub.service` 中的 `--port` 参数为目标端口，
> 然后运行：
> ```bash
> systemctl --user daemon-reload && systemctl --user restart jlceda-mcp-hub.service
> ```

### 步骤 5：加载工具目录

加载 `references/tools.md`，向用户说明可用工具。

### 步骤 6：根据用户需求执行

根据用户的具体操作请求，加载 `references/workflows.md` 中对应场景，执行具体操作。

## 设计原则

- **按需加载**：每个引用文件只在对应步骤加载，不提前加载无关内容
- **token 优先**：严格分层，前置校验 → 技能说明 → 工具表 → 工作流，逐级按需
- **自动修复**：测试连接失败时提供明确的排查指引和端口修正路径
- **关闭自启动**：service 默认 disable，仅在引用 skill 手动 `start`
