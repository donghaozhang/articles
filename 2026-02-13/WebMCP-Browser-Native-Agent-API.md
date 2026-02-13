# WebMCP：让每个网站成为 AI Agent 的工具接口

> 作者：AlexZ 🦀 (@blackanger) | 2026-02-12
> 原文：https://x.com/blackanger/status/2021886010918105365
> 互动：116 likes, 21 reposts, 206 bookmarks, 52K views

---

## 是什么？

**WebMCP** 是 Google 和 Microsoft 联合推动的 **W3C Web 标准提案**，2026年2月10日在 Chrome 146 中发布早期预览版。

核心：浏览器原生 JavaScript API（`navigator.modelContext`），允许网站将功能以结构化"工具"形式暴露给 AI Agent。

**不是 Anthropic 的 MCP 协议**，不使用 JSON-RPC，是纯客户端 Web 原生标准 — 网页本身就是"服务器"。

---

## 起源

Amazon 工程师 Alex Nahas 的痛点：数千个内部服务打包成一个 MCP 服务器，塞满上下文窗口，且 OAuth 2.1 认证与内部体系不兼容。

核心洞察：**直接在浏览器中运行，复用已有的 SSO、Cookie 和认证机制**。

Google Chrome 团队 + Microsoft Edge 团队同期也在探索类似方向 → 三方通过 W3C 合流 → 2025年8月正式提案 → 2025年9月被接纳为 W3C 社区组交付物。

⚠️ **Apple/Safari 和 Mozilla/Firefox 尚未参与** — 仍以 Chromium 生态主导。

---

## 双轨 API 设计

### 声明式 API（HTML 表单属性）
在现有 `<form>` 上添加 `toolname`、`tooldescription` 属性，浏览器自动从表单字段生成工具参数 Schema。零 JavaScript，内容作者即可参与。

### 命令式 API（JavaScript）
```javascript
navigator.modelContext.registerTool({
  name: "add-to-cart",
  description: "Add a product to the shopping cart",
  inputSchema: {
    type: "object",
    properties: {
      productId: { type: "string" },
      quantity: { type: "number" }
    }
  },
  execute({ productId, quantity }) {
    addToCart(productId, quantity);
    return { content: [{ type: "text", text: "Item added!" }] };
  }
});
```

### 配套特性
- `SubmitEvent.agentInvoked` — 区分 Agent 提交 vs 人类提交
- CSS `:tool-form-active` / `:tool-submit-active` — Agent 操作时不同视觉状态
- Model Context Tool Inspector Chrome 扩展 — 调试工具

---

## WebMCP vs Anthropic MCP

| 维度 | Anthropic MCP | WebMCP |
|------|--------------|--------|
| 协议基础 | JSON-RPC 2.0 | Web 原生 API |
| 架构 | Client-Server（需后端） | 纯客户端（网页即服务器） |
| 传输 | stdio / HTTP / SSE | postMessage / 浏览器运行时 |
| 认证 | OAuth 2.1 | 浏览器原有认证（Cookie） |
| 运行环境 | Python / Node.js 后端 | 浏览器 JavaScript |
| 能力范围 | Tools + Resources + Prompts | 当前仅 Tools |
| 可用性 | 服务器常驻运行 | 用户导航到页面时才可用 |

**互补关系**：MCP 处理后端，WebMCP 处理前端浏览器交互。

---

## vs Browser Use / Computer Use

| 方式 | Token 消耗 | 需要网站配合？ | 确定性 |
|------|-----------|--------------|--------|
| 视觉方式（截图） | ~2000 token/张 | 否 | 概率性 |
| 语义方式（DOM 解析） | 中等 | 否 | 半确定性 |
| **WebMCP** | **20-100 token** | **是** | **确定性** |

WebMCP 可**节省高达 89% 的 token 消耗**，但需要网站主动适配。

---

## 生态现状

1. **W3C WebMCP 标准** — 社区组草案阶段，规范仍骨架化
2. **MCP-B** (947 Stars) — Alex Nahas 的参考实现/Polyfill，npm `@mcp-b/global`
3. **webmcp.dev** (315 Stars) — Jason McGhee (Cursor 联合创始人) 的 WebSocket 桥接方案

---

## 安全隐忧

**"致命三元组"问题**：用户同时打开银行标签页和恶意标签页，浏览器 Agent 可能被操纵泄露数据。

W3C 成员 Tom Jones 直言："没有进行过安全审查…将是隐私噩梦。"

缓解措施：域名级工具隔离、工具哈希验证、用户确认流程、域信任 TTL，但未完全消除风险。

---

## 关键信号

1. Chrome 146 早期预览 → Google 已推进到可测试阶段，预计 **2026 年中后期官方公告**
2. 声明式 API 极低门槛 → 可能催生类似 Schema.org 的大规模普及
3. Apple/Mozilla 缺席 → 最大不确定性，可能沦为 Chromium 独有特性

> **"Agent SEO" 新范式**：网站曾发现搜索引擎也是"用户"，现在需要意识到 AI Agent 同样是"用户"。

---

*Collected: 2026-02-13*
