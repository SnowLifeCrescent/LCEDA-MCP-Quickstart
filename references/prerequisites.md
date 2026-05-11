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

### 3. 【关键修正】为 component_select 添加无交互模式

上游仓库中 `component_select` 依赖 VS Code/Cursor 侧边栏交互确认，**在 opencode CLI 等纯终端环境下会永久挂起直到 15 分钟超时**。需要注入一个 `interactive` 参数，当为 `false` 时直接返回搜索结果。

依次检查以下两个文件，对比记录的原始代码段，若匹配则替换为修正后代码。

#### 3a. 修正工具定义

文件：`~/.opencode/jlceda-mcp-hub/mcp-hub/src/data/mcp-tool-definitions.json`

查找 `component_select` 段中 `"limit"` 对象结尾后的原始代码：

<details>
<summary>原始代码（点击展开）</summary>

```json
        "limit": {
          "type": "integer",
          "minimum": 2,
          "maximum": 20,
          "default": 20,
          "description": "返回候选数量上限，范围 2-20，默认 20。"
        }
      },
      "required": [
        "keyword"
      ]
    }
```

</details>

若文件中此段与该原始代码**完全匹配**，则替换为：

<details>
<summary>修正后代码（点击展开）</summary>

```json
        "limit": {
          "type": "integer",
          "minimum": 2,
          "maximum": 20,
          "default": 20,
          "description": "返回候选数量上限，范围 2-20，默认 20。"
        },
        "interactive": {
          "type": "boolean",
          "default": true,
          "description": "是否启用交互选型面板。false 时跳过侧边栏人工确认，直接返回搜索结果由 AI 自行筛选。无图形界面的环境（如 opencode CLI）应设为 false。"
        }
      },
      "required": [
        "keyword"
      ]
    }
```

</details>

替换完成后保存。

#### 3b. 修正分发器逻辑

文件：`~/.opencode/jlceda-mcp-hub/mcp-hub/src/server/mcp/tool-dispatcher.ts`

查找 `handleComponentSelect` 方法中解析完 `initialPayload` 后的原始代码：

<details>
<summary>原始代码（点击展开）</summary>

```typescript
		const initialPayload = this.parseComponentSelectBridgePayload(initialResult);
		if (!initialPayload) {
			return initialResult;
		}

		const requestId = createInteractionRequestId('component_select');
```

</details>

若文件中此段与该原始代码**完全匹配**，则替换为：

<details>
<summary>修正后代码（点击展开）</summary>

```typescript
		const initialPayload = this.parseComponentSelectBridgePayload(initialResult);
		if (!initialPayload) {
			return initialResult;
		}

		// 非交互模式：跳过侧边栏等待，直接返回搜索结果由 AI 自行筛选。
		if (argumentsObject.interactive === false) {
			return {
				ok: true,
				interactive: false,
				candidates: initialPayload.candidates,
				pageSize: initialPayload.pageSize,
				currentPage: initialPayload.currentPage,
				totalPages: Math.ceil(initialPayload.candidates.length / initialPayload.pageSize),
				message: `非交互搜索完成，共 ${String(initialPayload.candidates.length)} 个候选器件。请 AI 根据名称、封装、描述等字段自行筛选最合适的器件，并将最终确认的 uuid 和 libraryUuid 传递给 component_place。`,
			};
		}

		const requestId = createInteractionRequestId('component_select');
```

</details>

替换完成后保存。

#### 3c. 重新构建

```bash
cd ~/.opencode/jlceda-mcp-hub/mcp-hub
node config/esbuild.prod.mjs
```

### 4. 创建 systemd 用户服务

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

### 5. 注册服务（不自启）

```bash
systemctl --user daemon-reload
systemctl --user disable jlceda-mcp-hub.service
```

服务默认不自启，仅在需要时手动 `start`。

### 6. 启用 session linger

```bash
loginctl enable-linger
```

确保用户退出登录后服务仍然可以运行。

### 7. 测试 HTTP MCP 端点

```bash
systemctl --user start jlceda-mcp-hub.service
sleep 2
curl -s http://127.0.0.1:7655/mcp -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

返回含 `schematic_read` 等工具列表即成功。

### 8. 配置 opencode.json

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

### 9. 嘉立创 EDA 侧安装 Bridge

1. 打开嘉立创 EDA 专业版
2. 扩展管理器 → 搜索 **MCP Bridge** → 安装
3. Bridge 设置页填入桥接地址：`ws://127.0.0.1:8765/bridge/ws`
4. 打开任意工程，确认 Bridge 状态已连接

### 10. 最终验证

在终端执行：

```bash
curl -s http://127.0.0.1:7655/mcp -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"schematic_read","arguments":{}}}'
```

返回电路数据即全部就绪。
