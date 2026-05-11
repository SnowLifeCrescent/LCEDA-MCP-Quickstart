# 技能概述

让 AI 助手通过 MCP 协议远程读取、审查、编辑嘉立创 EDA 专业版中的原理图。

## 连接链路

```
AI 助手 (opencode)
    ↕ HTTP/MCP 协议 (端口 7655)
mcp-hub 运行时 (本地 Node.js 进程)
    ↕ WebSocket 桥接 (端口 8765)
mcp-bridge 扩展 (嘉立创 EDA 专业版)
    ↕ EDA 内部 API
嘉立创 EDA 当前工程
```

## 地址记录

| 用途 | 地址 |
|------|------|
| HTTP MCP 端点 | `http://127.0.0.1:7655/mcp` |
| WebSocket 桥接 | `ws://127.0.0.1:8765/bridge/ws` |

## 服务管理

```bash
systemctl --user start jlceda-mcp-hub     # 启动
systemctl --user stop jlceda-mcp-hub      # 停止
systemctl --user status jlceda-mcp-hub    # 查看状态
journalctl --user -u jlceda-mcp-hub -f    # 实时日志
```

## 使用条件

- 嘉立创 EDA 专业版已打开，且位于原理图或 PCB 页面
- mcp-bridge 扩展已安装且桥接地址已正确配置
- mcp-hub 服务正在运行（通过本 skill 的入口流程自动管理）
