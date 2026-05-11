[![中文](https://img.shields.io/badge/lang-zh-red.svg)](README.md)
[![ENG](https://img.shields.io/badge/lang-en-blue.svg)](README.en.md)

# LCEDA-MCP-Quickstart

这是一个 Skill ，一个让你的 AI 助手通过 MCP 协议**远程读取、审查和操控嘉立创 EDA 专业版原理图**的 Claude Code / opencode 技能包。

能够在终端里直接让 AI 帮你检查电路、选型器件、浏览工程。

本 **Skill** 完全基于 [MCP-Bridge](https://github.com/sengbin/JLCEDA-MCP) ，仅把原作者的功能打包成了技能方便调用，并让 agent 能够对原作者开发的 MCP 协议快速完成配置开始使用。

   - 请务必支持原作者，并为原作者点上 **STAR !!!**


## 工作原理

```
AI 助手 (Claude Code / opencode)
    ↕ HTTP/MCP (端口 7655)
mcp-hub 运行时 (本地 Node.js 进程)
    ↕ WebSocket 桥接 (端口 8765)
mcp-bridge 扩展 (嘉立创 EDA 专业版)
    ↕ EDA 内部 API
嘉立创 EDA 当前工程
```

本技能将引导 agent 自动完成全套配置，对 [MCP-Bridge](https://github.com/sengbin/JLCEDA-MCP) 实现即插即用。

## 文件结构

```
LCEDA-MCP-Quickstart/
├── SKILL.md                    # 入口：首次检测 → 服务生命周期 → 连接测试
├── README.md                   # 本文件（英文版）
├── README.zh.md                # 本文件（中文版）
└── references/
    ├── prerequisites.md        # 从零配置流程（新环境首次使用）
    ├── skill-info.md           # 技能概述、链路图、地址记录
    ├── tools.md                # 全部 8 个 MCP 工具及参数
    └── workflows.md            # 常用场景编排（读图、审图、选型、放置）
```

## 可用功能

| 工具 | 用途 |
|------|------|
| `schematic_read` | 读取当前原理图页面的完整电路语义快照 |
| `schematic_review` | 读取全工程所有原理图页面的网表文件 |
| `component_select` | 在 EDA 库中搜索器件，侧边栏确认 |
| `component_place` | 引导逐步放置器件到原理图中 |
| *(另有 4 个可选透传 API 工具)* | 高级/自定义 EDA API 操作 |

## 环境要求

- Node.js ≥ 20
- git
- 嘉立创 EDA 专业版
- 支持 skill 机制的 Claude Code / opencode 等

## 新用户快速开始

在新环境中首次调用本技能时，它会自动检测到未配置并引导你完成：

1. **克隆和构建** mcp-hub 运行时（基于 [JLCEDA-MCP](https://github.com/sengbin/JLCEDA-MCP) 项目）
2. **注册** systemd 用户服务（设计上**不自启** — 需要时才启动）
3. **配置** `opencode.json` 添加 MCP 端点
4. **引导**你在嘉立创 EDA 中安装 MCP Bridge 扩展
5. **验证**连接是否端到端正常

配置完成后 AI 即可直接连接到你的 EDA。

## 服务管理

```bash
systemctl --user start jlceda-mcp-hub      # 按需启动
systemctl --user stop jlceda-mcp-hub       # 用完关闭
systemctl --user status jlceda-mcp-hub     # 查看状态
journalctl --user -u jlceda-mcp-hub -f     # 实时日志
```

## 致谢

- **mcp-hub** 运行时来自 [sengbin](https://github.com/sengbin) 的 [JLCEDA-MCP](https://github.com/sengbin/JLCEDA-MCP) 项目

## 许可证

Apache 2.0
