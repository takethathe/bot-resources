# OpenClaw Canvas 深度调研报告

**调研时间**: 2026-03-02  
**项目**: [openclaw/openclaw](https://github.com/openclaw/openclaw)  
**调研焦点**: Canvas 功能模块

---

## 📌 概述

**Canvas** 是 OpenClaw 项目中的一个核心可视化组件，它是一个**由 AI 智能体控制的轻量级可视化工作区**，用于渲染 HTML/CSS/JS、A2UI 和小型交互式界面。

---

## 🎯 Canvas 是什么

### 核心定义

Canvas 是一个嵌入在 OpenClaw 客户端应用（macOS、iOS、Android）中的**可视化面板**，通过 WebView 技术实现：

| 平台 | 实现技术 |
|------|----------|
| **macOS** | `WKWebView` + 自定义 URL 方案 (`openclaw-canvas://`) |
| **iOS** | `WKWebView` + WebKit Message Handlers |
| **Android** | `WebView` + JavaScript Interface |
| **Desktop/Web** | 标准 HTML5 Canvas + WebSocket |

### 架构位置

```
┌─────────────────────────────────────────────────────────────┐
│                      OpenClaw 架构                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐ │
│  │   Agent     │───▶│   Gateway   │───▶│   Canvas Host   │ │
│  │  (智能体)    │    │  (WebSocket)│    │  (HTTP Server)  │ │
│  └─────────────┘    └─────────────┘    └────────┬────────┘ │
│                                                  │          │
│                                                  ▼          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Client Applications                     │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │   │
│  │  │  macOS   │  │   iOS    │  │     Android      │  │   │
│  │  │ WKWebView│  │ WKWebView│  │      WebView     │  │   │
│  │  │  Canvas  │  │  Canvas  │  │      Canvas      │  │   │
│  │  └──────────┘  └──────────┘  └──────────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 💡 Canvas 的用途

### 1. **智能体 UI 渲染**

Canvas 最主要的用途是渲染 **A2UI (Agent-to-User Interface)**，这是 OpenClaw 的声明式 UI 系统：

- 智能体通过 WebSocket 发送 A2UI 指令
- Canvas 实时渲染组件（文本、按钮、列表等）
- 支持用户交互并回传给智能体

### 2. **交互式工作区**

- 显示 HTML/CSS/JS 内容
- 运行小型 Web 应用
- 展示数据可视化图表
- 提供表单和输入界面

### 3. **智能体控制界面**

智能体可以通过 API 完全控制 Canvas：

```bash
# 显示 Canvas 面板
openclaw nodes canvas present --node <id>

# 导航到指定 URL
openclaw nodes canvas navigate --node <id> --url "/"

# 执行 JavaScript
openclaw nodes canvas eval --node <id> --js "document.title"

# 捕获截图
openclaw nodes canvas snapshot --node <id>

# 推送 A2UI 界面
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

### 4. **实时预览**

- 文件变更自动重新加载（live reload）
- 支持 WebSocket 实时通信
- 调试状态显示

---

## 📚 使用场景

### 场景 1: AI 助手界面展示

```
用户：帮我创建一个待办事项列表
智能体：→ 通过 Canvas 渲染交互式 Todo 列表
        → 用户可以直接在 Canvas 中勾选完成
        → 智能体同步更新状态
```

### 场景 2: 代码预览和编辑

```
用户：查看这个 HTML 文件的效果
智能体：→ 在 Canvas 中渲染 HTML 预览
        → 用户可以看到实时效果
        → 支持即时修改和重新渲染
```

### 场景 3: 数据可视化

```
用户：分析这个数据集
智能体：→ 在 Canvas 中生成图表
        → 交互式数据探索界面
        → 支持缩放、筛选等操作
```

### 场景 4: 多模态交互

```
用户：（语音）显示今天的天气
智能体：→ Canvas 显示天气卡片
        → 包含温度、降水概率等可视化信息
        → 支持语音和视觉双向交互
```

---

## 🔧 底层实现方案

### 1. **自定义 URL 方案 (Custom URL Scheme)**

OpenClaw 实现了专有的 URL 方案来服务 Canvas 内容：

**URL 格式**: `openclaw-canvas://<session>/<path>`

**示例**:
- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`

**macOS 实现** (`CanvasSchemeHandler.swift`):

```swift
final class CanvasSchemeHandler: NSObject, WKURLSchemeHandler {
    private let root: URL
    
    func webView(_ webView: WKWebView, start urlSchemeTask: WKURLSchemeTask) {
        // 解析 URL
        guard let url = urlSchemeTask.request.url else { return }
        guard let session = url.host, !session.isEmpty else { return }
        
        // 安全解析文件路径
        let sessionRoot = self.root.appendingPathComponent(session)
        let resolved = self.resolveFileURL(sessionRoot: sessionRoot, requestPath: url.path)
        
        // 目录遍历保护
        guard standardizedFile.path.hasPrefix(standardizedRoot.path) else {
            return self.html("Forbidden")
        }
        
        // 返回文件内容
        let data = try Data(contentsOf: standardizedFile)
        let mime = CanvasScheme.mimeType(forExtension: standardizedFile.pathExtension)
        urlSchemeTask.didReceive(data)
    }
}
```

**安全特性**:
- ✅ 阻止目录遍历攻击
- ✅ 会话隔离（每个 session 独立目录）
- ✅ MIME 类型验证
- ✅ 路径标准化检查

---

### 2. **Canvas 服务器 (Canvas Host Server)**

Gateway 内置 HTTP 服务器用于服务 Canvas 内容：

**核心文件**: `src/canvas-host/server.ts`

```typescript
export async function createCanvasHostHandler(opts: CanvasHostHandlerOpts) {
    const rootDir = resolveUserPath(opts.rootDir ?? resolveDefaultCanvasRoot())
    const rootReal = await prepareCanvasRoot(rootDir)
    
    // WebSocket 服务器用于 live reload
    const wss = liveReload ? new WebSocketServer({ noServer: true }) : null
    
    // 文件监听器
    const watcher = chokidar.watch(rootReal, {
        ignoreInitial: true,
        awaitWriteFinish: { stabilityThreshold: 75, pollInterval: 10 }
    })
    watcher.on('all', () => scheduleReload()) // 文件变更广播 reload
    
    // HTTP 请求处理
    const handleHttpRequest = async (req, res) => {
        const opened = await resolveFileWithinRoot(rootReal, urlPath)
        if (!opened) return send404()
        
        const mime = await detectMime({ filePath: realPath })
        if (mime === 'text/html') {
            // 注入 live reload 脚本
            res.end(injectCanvasLiveReload(html))
        } else {
            res.end(data)
        }
    }
}
```

**特性**:
- 📁 文件服务（HTML、CSS、JS、图片等）
- 🔄 实时重载（WebSocket 广播）
- 🔒 路径安全验证
- 📊 MIME 类型自动检测

---

### 3. **A2UI 渲染引擎**

**A2UI (Agent-to-User Interface)** 是 OpenClaw 的声明式 UI 系统：

**协议版本**: v0.8 (当前支持)

**消息类型**:
| 消息 | 用途 |
|------|------|
| `beginRendering` | 开始渲染 surface |
| `surfaceUpdate` | 更新组件树 |
| `dataModelUpdate` | 更新数据模型 |
| `deleteSurface` | 删除 surface |

**组件系统**:

```json
{
  "surfaceUpdate": {
    "surfaceId": "main",
    "components": [
      {
        "id": "root",
        "component": {
          "Column": {
            "children": { "explicitList": ["title", "content"] }
          }
        }
      },
      {
        "id": "title",
        "component": {
          "Text": {
            "text": { "literalString": "Canvas (A2UI v0.8)" },
            "usageHint": "h1"
          }
        }
      }
    ]
  },
  "beginRendering": { "surfaceId": "main", "root": "root" }
}
```

**渲染流程**:

```
1. Gateway 发送 A2UI JSONL 消息
   ↓
2. Canvas 解析消息
   ↓
3. 构建组件树 (Component Tree)
   ↓
4. 渲染到 HTML5 Canvas
   ↓
5. 用户交互 → 触发 Action → 回传 Gateway
```

---

### 4. **跨平台通信桥接**

**iOS (WebKit Message Handlers)**:

```javascript
// 在 WKWebView 中
window.webkit.messageHandlers.openclawCanvasA2UIAction.postMessage(payload)
```

**Swift 端接收**:

```swift
func userContentController(_ controller: WKUserContentController,
                           didReceive message: WKScriptMessage) {
    if message.name == "openclawCanvasA2UIAction" {
        // 处理智能体动作
        self.handleCanvasAction(message.body)
    }
}
```

**Android (JavaScript Interface)**:

```javascript
// 在 WebView 中
window.openclawCanvasA2UIAction.postMessage(payload)
```

**Java/Kotlin 端接收**:

```kotlin
@JavascriptInterface
fun postMessage(payload: String) {
    // 处理智能体动作
    handleCanvasAction(payload)
}
```

**统一桥接层** (`a2ui.ts`):

```typescript
function postToNode(payload) {
    const handlerNames = ["openclawCanvasA2UIAction"]
    for (const name of handlerNames) {
        // iOS
        const iosHandler = globalThis.webkit?.messageHandlers?.[name]
        if (iosHandler && typeof iosHandler.postMessage === "function") {
            iosHandler.postMessage(JSON.stringify(payload))
            return true
        }
        // Android
        const androidHandler = globalThis[name]
        if (androidHandler && typeof androidHandler.postMessage === "function") {
            androidHandler.postMessage(JSON.stringify(payload))
            return true
        }
    }
    return false
}
```

---

### 5. **Canvas 能力令牌 (Capability Token)**

**安全机制**: Canvas 访问需要能力令牌

**令牌生成** (`canvas-capability.ts`):

```typescript
export function mintCanvasCapabilityToken(): string {
    return randomBytes(18).toString("base64url") // 144-bit 随机令牌
}

export const CANVAS_CAPABILITY_TTL_MS = 10 * 60_000 // 10 分钟过期
```

**URL 格式**:

```
http://<gateway-host>:18789/__openclaw__/cap/<capability>/__openclaw__/canvas/?oc_cap=<capability>
```

**验证流程**:

```
1. 客户端请求 Canvas URL
   ↓
2. 提取 capability token
   ↓
3. 验证 token 有效性（未过期、未使用）
   ↓
4. 验证通过后允许访问
   ↓
5. Token 使用后立即失效
```

---

### 6. **Canvas 工具 (Agent Tool)**

智能体通过 `canvas` 工具与 Canvas 交互：

**工具定义** (`canvas-tool.ts`):

```typescript
const CANVAS_ACTIONS = [
    "present",      // 显示面板
    "hide",         // 隐藏面板
    "navigate",     // 导航到 URL
    "eval",         // 执行 JavaScript
    "snapshot",     // 捕获截图
    "a2ui_push",    // 推送 A2UI 界面
    "a2ui_reset",   // 重置 A2UI
] as const

export function createCanvasTool() {
    return {
        name: "canvas",
        description: "Control node canvases (present/hide/navigate/eval/snapshot/A2UI)",
        parameters: CanvasToolSchema,
        execute: async (_toolCallId, args) => {
            const action = args.action
            switch (action) {
                case "present":
                    await invoke("canvas.present", { url, placement })
                    break
                case "navigate":
                    await invoke("canvas.navigate", { url })
                    break
                case "eval":
                    const result = await invoke("canvas.eval", { javaScript })
                    return result.payload.result
                case "snapshot":
                    const image = await invoke("canvas.snapshot", { format, maxWidth })
                    return imageResult(image.base64)
                case "a2ui_push":
                    await invoke("canvas.a2ui.pushJSONL", { jsonl })
                    break
            }
        }
    }
}
```

---

## 📁 文件结构

```
openclaw/
├── src/
│   ├── canvas-host/
│   │   ├── server.ts              # Canvas HTTP 服务器
│   │   ├── a2ui.ts                # A2UI 服务和注入
│   │   ├── file-resolver.ts       # 文件解析和安全验证
│   │   └── a2ui/
│   │       └── index.html         # A2UI 宿主页面
│   │
│   ├── gateway/
│   │   └── canvas-capability.ts   # Canvas 能力令牌管理
│   │
│   ├── agents/
│   │   └── tools/
│   │       └── canvas-tool.ts     # 智能体 Canvas 工具
│   │
│   └── infra/
│       └── canvas-host-url.ts     # Canvas 主机 URL 解析
│
├── apps/macos/
│   └── Sources/OpenClaw/
│       ├── CanvasManager.swift          # Canvas 管理器
│       ├── CanvasWindowController.swift # Canvas 窗口控制器
│       ├── CanvasSchemeHandler.swift    # 自定义 URL 方案处理
│       └── CanvasFileWatcher.swift      # 文件变更监听
│
└── docs/
    └── platforms/mac/canvas.md    # Canvas 文档
```

---

## 🔐 安全特性

| 安全机制 | 描述 |
|----------|------|
| **自定义 URL 方案** | 隔离 Canvas 内容，防止外部访问 |
| **能力令牌** | 10 分钟过期的随机令牌，防止未授权访问 |
| **路径验证** | 标准化路径检查，防止目录遍历 |
| **会话隔离** | 每个 session 独立目录，数据隔离 |
| **MIME 验证** | 自动检测文件类型，防止 XSS |
| **HTTPS 支持** | 支持加密传输（通过代理） |

---

## 📊 技术栈总结

| 层级 | 技术 |
|------|------|
| **客户端渲染** | WKWebView (macOS/iOS), WebView (Android), HTML5 Canvas |
| **服务器** | Node.js HTTP Server + WebSocket |
| **通信协议** | WebSocket (Gateway ↔ Client), Custom URL Scheme |
| **UI 系统** | A2UI v0.8 (声明式组件系统) |
| **文件监听** | chokidar (跨平台文件监听) |
| **安全** | Capability Token, Path Validation, MIME Detection |

---

## 🎓 总结

**OpenClaw Canvas** 是一个设计精良的跨平台可视化组件，核心特点：

1. **智能体控制** - AI 智能体可以完全控制 Canvas 的显示、内容和交互
2. **跨平台一致** - macOS/iOS/Android 使用统一的 API 和协议
3. **安全可靠** - 多层安全机制保护用户数据
4. **实时交互** - WebSocket 实现低延迟双向通信
5. **声明式 UI** - A2UI 提供简洁的界面描述语言
6. **灵活扩展** - 支持自定义 HTML/CSS/JS 和 A2UI 组件

Canvas 是 OpenClaw 实现**多模态人机交互**的核心基础设施，使 AI 智能体能够以可视化方式与用户进行自然交互。

---

*调研报告生成于 2026-03-02*
