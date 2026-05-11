[![中文](https://img.shields.io/badge/lang-zh-red.svg)](README.md)
[![ENG](https://img.shields.io/badge/lang-en-blue.svg)](README.en.md)

# LCEDA-MCP-Quickstart

This is a "Skill"—specifically a Claude Code / opencode skill package—that enables your AI assistant to **remotely read, review, and manipulate JLC EDA Pro schematics** via the MCP protocol.

Ask your AI to check circuits, select components, and navigate your EDA project, all from the terminal.


This **Skill** is built entirely upon [MCP-Bridge](https://github.com/sengbin/JLCEDA-MCP). It simply packages the original author's functionality into a convenient "skill" format for easy invocation, while also enabling the agent to quickly configure and begin utilizing the MCP protocol developed by the original author.

   - Please be sure to support the original author—and don't forget to give him/her a **STAR !!!**


## How It Works

```
AI Assistant (Claude Code / opencode)
    ↕ HTTP/MCP (port 7655)
mcp-hub runtime (local Node.js process)
    ↕ WebSocket bridge (port 8765)
mcp-bridge extension (LCEDA Professional)
    ↕ EDA internal API
嘉立创 EDA active project
```

The skill automates the full setup and provides a **zero-friction onboarding** for new environments.

## Skill Structure

```
LCEDA-MCP-Quickstart/
├── SKILL.md                    # Entry point: first-use detection, service lifecycle, connection testing
├── README.md                   # This file
└── references/
    ├── prerequisites.md        # First-time setup from scratch (for new environments)
    ├── skill-info.md           # Overview, connection diagram, addresses
    ├── tools.md                # All 8 MCP tools with parameters
    └── workflows.md            # Common usage scenarios (read, review, select, place)
```

## What You Can Do

| Tool | Purpose |
|------|---------|
| `schematic_read` | Read full circuit snapshot of the current schematic page |
| `schematic_review` | Review all schematic pages via netlist files |
| `component_select` | Search components in EDA library, confirm via sidebar |
| `component_place` | Guide component placement in EDA step by step |
| *(4 more optional passthrough API tools)* | Advanced/custom EDA API operations |

## Requirements

- Node.js ≥ 20
- git
- 嘉立创 EDA Professional Edition
- Claude Code or opencode and so on which with skill support

## Quick Start for New Users

When you invoke this skill in a fresh environment, it will detect that nothing is configured and automatically walk you through:

1. **Clone & build** the mcp-hub runtime from the [JLCEDA-MCP](https://github.com/sengbin/JLCEDA-MCP) project
2. **Register** a systemd user service (auto-start **disabled** by design — start only when needed)
3. **Configure** `opencode.json` with the MCP endpoint
4. **Guide** you through installing the MCP Bridge extension in 嘉立创 EDA
5. **Verify** the connection end-to-end

After setup, your AI will immediately connect to your EDA.

## Service Management

```bash
systemctl --user start jlceda-mcp-hub      # Start when needed
systemctl --user stop jlceda-mcp-hub       # Stop when done
systemctl --user status jlceda-mcp-hub     # Check status
journalctl --user -u jlceda-mcp-hub -f     # Live logs
```

## Credits

- **mcp-hub** runtime by [sengbin](https://github.com/sengbin) — the [JLCEDA-MCP](https://github.com/sengbin/JLCEDA-MCP) project

## License

Apache 2.0
