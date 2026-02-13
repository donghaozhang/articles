# MCP Apps：在聊天中渲染交互式 HTML 界面

> Source: Model Context Protocol Official Docs
> Original: https://modelcontextprotocol.io/docs/extensions/apps
> Repo: https://github.com/modelcontextprotocol/ext-apps

---

## 是什么？

**MCP Apps** 让 MCP 服务器返回**交互式 HTML 界面**（数据可视化、表单、仪表板），直接渲染在聊天对话中。

不只是文本回复 — 用户可以直接与数据交互。

---

## 为什么不直接做 Web App？

MCP Apps 的核心优势：

- **上下文保留** — UI 就在对话中，不用切标签页
- **双向数据流** — App 可调用 MCP 服务器的任何工具，Host 可推送数据给 App（无需自建 API/认证/状态管理）
- **集成 Host 能力** — App 可委托 Host 调用用户已连接的工具（如"安排会议"），不用每个 App 自己维护集成
- **安全沙箱** — 运行在 Host 控制的 sandboxed iframe 中，不能访问父页面/Cookie

---

## 工作原理

1. **UI 预加载** — 工具描述中包含 `_meta.ui.resourceUri` 指向 `ui://` 资源，Host 可在工具调用前预加载
2. **资源获取** — Host 从服务器获取 HTML 页面（可内联 JS/CSS，也可加载外部资源）
3. **沙箱渲染** — HTML 在 sandboxed iframe 中渲染，可通过 `_meta.ui` 请求额外权限（麦克风、摄像头等）
4. **双向通信** — App 和 Host 通过 JSON-RPC 协议（`ui/` 前缀方法）通信，App 可调用工具、发送消息、更新上下文

---

## 适用场景

| 场景 | 示例 |
|------|------|
| **复杂数据探索** | 交互式地图，点击区域下钻，悬停查看详情 |
| **多选项配置** | 部署配置表单，一次展示所有选项+验证 |
| **富媒体查看** | PDF 查看器、3D 模型、图片预览（缩放/旋转） |
| **实时监控** | 持续更新的指标仪表板 |
| **多步工作流** | 审批报销、代码 Review、Issue 分类 |

---

## 快速开始

### 用 AI Coding Agent + Skill

```bash
# Claude Code
/plugin marketplace add modelcontextprotocol/ext-apps
/plugin install mcp-apps@modelcontextprotocol-ext-apps

# Vercel Skills CLI
npx skills add modelcontextprotocol/ext-apps
```

支持的 Agent：Claude Code、VS Code/GitHub Copilot、Gemini CLI、Cline、Goose、Codex

### 服务端核心代码

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { registerAppTool, registerAppResource } from "@modelcontextprotocol/ext-apps/server";

const resourceUri = "ui://get-time/mcp-app.html";

// 注册带 UI 的工具
registerAppTool(server, "get-time", {
  title: "Get Time",
  description: "Returns the current server time.",
  inputSchema: {},
  _meta: { ui: { resourceUri } },
}, async () => ({
  content: [{ type: "text", text: new Date().toISOString() }],
}));

// 注册 HTML 资源
registerAppResource(server, resourceUri, resourceUri, 
  { mimeType: RESOURCE_MIME_TYPE },
  async () => ({
    contents: [{ uri: resourceUri, mimeType: RESOURCE_MIME_TYPE, text: html }],
  })
);
```

### 客户端核心代码

```typescript
import { App } from "@modelcontextprotocol/ext-apps";

const app = new App({ name: "Get Time App", version: "1.0.0" });
app.connect();

// 接收 Host 推送的工具结果
app.ontoolresult = (result) => {
  const time = result.content?.find(c => c.type === "text")?.text;
  serverTimeEl.textContent = time ?? "[ERROR]";
};

// 主动调用服务端工具
getTimeBtn.addEventListener("click", async () => {
  const result = await app.callServerTool({ name: "get-time", arguments: {} });
});
```

---

## 测试方式

- **Claude (Web/Desktop)** — 支持 MCP Apps，本地开发用 `npx cloudflared tunnel` 暴露服务
- **basic-host** — ext-apps 仓库自带的测试 Host

---

## 关键设计决策

- `ui://` scheme 标识 MCP App 资源
- 沙箱 iframe 确保安全隔离
- JSON-RPC 双向通信（`ui/` 前缀）
- 工具描述中声明 UI → LLM 决定何时调用 → Host 渲染
- 支持 CSP 配置外部资源加载

---

## 意义

MCP Apps 把 MCP 从"返回文本"扩展到"返回交互式应用"。这意味着 MCP 服务器不只是数据提供者，而是可以提供完整的 UI 体验 — 在对话上下文中。

配合 WebMCP（浏览器端）和 Anthropic MCP（后端），形成了完整的 AI Agent 交互生态。

---

*Collected: 2026-02-13*
