# 前置需求

从零配置 opencode 通过 MCP 远程操控嘉立创 EDA 专业版。

---

## 环境要求

- Node.js ≥ 20
- git
- 嘉立创 EDA 专业版（已安装）

## 步骤

### 1. 克隆并构建 mcp-hub 运行时

```bash
git clone https://github.com/sengbin/JLCEDA-MCP.git ~/.opencode/jlceda-mcp-hub
cd ~/.opencode/jlceda-mcp-hub/mcp-hub
npm install
npm run build
```

### 2. 创建数据目录

```bash
mkdir -p ~/.opencode/jlceda-mcp-hub/data
```

### 3. 创建 systemd 用户服务

写入 `~/.config/systemd/user/jlceda-mcp-hub.service`：

```ini
[Unit]
Description=JLCEDA MCP Hub - MCP server for JLC EDA integration
Documentation=https://github.com/sengbin/JLCEDA-MCP
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/node /home/fgj/.opencode/jlceda-mcp-hub/mcp-hub/out/server/runtime.js \
    --storage-directory /home/fgj/.opencode/jlceda-mcp-hub/data \
    --session-id standalone \
    --host 127.0.0.1 \
    --port 8765 \
    --status-file /home/fgj/.opencode/jlceda-mcp-hub/data/runtime-status.json \
    --extension-version 1.5.4 \
    --http-port 7655 \
    --enable-system-log true \
    --enable-connection-list true
Restart=on-failure
RestartSec=3
Environment=NODE_ENV=production

[Install]
WantedBy=default.target
```

> `/home/fgj/` 替换为你的实际 home 目录路径。

### 4. 注册服务（不自启）

```bash
systemctl --user daemon-reload
systemctl --user disable jlceda-mcp-hub.service
```

服务默认不自启，仅在需要时手动 `start`。

### 5. 启用 session linger

```bash
loginctl enable-linger
```

确保用户退出登录后服务仍然可以运行。

### 6. 测试 HTTP MCP 端点

```bash
systemctl --user start jlceda-mcp-hub.service
sleep 2
curl -s http://127.0.0.1:7655/mcp -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

返回含 `schematic_read` 等工具列表即成功。

### 7. 配置 opencode.json

在项目 `.opencode/opencode.json` 中添加：

```json
"mcp": {
  "jlceda": {
    "type": "remote",
    "url": "http://127.0.0.1:7655/mcp",
    "enabled": true
  }
}
```

### 8. 嘉立创 EDA 侧安装 Bridge

1. 打开嘉立创 EDA 专业版
2. 扩展管理器 → 搜索 **MCP Bridge** → 安装
3. Bridge 设置页填入桥接地址：`ws://127.0.0.1:8765/bridge/ws`
4. 打开任意工程，确认 Bridge 状态已连接

### 9. 最终验证

在终端执行：

```bash
curl -s http://127.0.0.1:7655/mcp -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"schematic_read","arguments":{}}}'
```

返回电路数据即全部就绪。
