# OpenClaw Diffs 插件启用与测试指南

**日期：** 2026-03-02  
**OpenClaw 版本：** 2026.3.1  
**作者：** 大白 🐻‍❄️

---

## 📦 一、前置检查

### 1.1 确认 OpenClaw 版本

```bash
openclaw --version
```

**输出：**
```
2026.3.1
```

> ✅ Diffs 插件在 **v2026.3.1** 中首次引入

### 1.2 查看官方文档

```bash
cat /opt/homebrew/lib/node_modules/openclaw/docs/tools/diffs.md
```

**关键信息：**
- Diffs 是**可选插件工具**
- 支持 `before`/`after` 文本对比或统一补丁（unified patch）
- 输出：Gateway 查看器 URL 和/或 PNG 图片

---

## 🔧 二、安装依赖

### 2.1 安装 @pierre/diffs 包

```bash
npm install -g @pierre/diffs
```

**输出：**
```
added 51 packages in 5s
26 packages are looking for funding
```

> ⚠️ **重要：** 如果不安装此依赖，插件会加载失败：
> ```
> [plugins] diffs failed to load: Cannot find module '@pierre/diffs'
> ```

---

## ⚙️ 三、配置 OpenClaw

### 3.1 编辑配置文件

编辑 `~/.openclaw/openclaw.json`，在 `plugins` 部分添加：

```json5
{
  "plugins": {
    "allow": [
      "telegram",
      "diffs"  // ← 新增
    ],
    "entries": {
      "diffs": {  // ← 新增
        "enabled": true,
        "config": {
          "defaults": {
            "layout": "unified",
            "theme": "dark",
            "showLineNumbers": true,
            "mode": "both"
          }
        }
      }
    }
  }
}
```

### 3.2 完整配置示例

```json
{
  "plugins": {
    "allow": [
      "telegram",
      "diffs"
    ],
    "entries": {
      "telegram": {
        "enabled": true
      },
      "diffs": {
        "enabled": true,
        "config": {
          "defaults": {
            "layout": "unified",
            "theme": "dark",
            "showLineNumbers": true,
            "mode": "both"
          }
        }
      }
    }
  }
}
```

---

## 🔄 四、重启 Gateway

```bash
openclaw gateway restart
```

**验证状态：**

```bash
openclaw gateway status
```

**输出示例：**
```
Service: LaunchAgent (loaded)
Gateway: bind=loopback (127.0.0.1), port=18789
Runtime: running (pid 36932, state active)
RPC probe: ok
```

---

## ✅ 五、验证插件加载

```bash
openclaw plugins list
```

**成功输出：**
```
Plugins (3/39 loaded)

┌──────────────┬──────────┬──────────┬─────────────────────────────────────────┬──────────┐
│ Name         │ ID       │ Status   │ Source                                  │ Version  │
├──────────────┼──────────┼──────────┼─────────────────────────────────────────┼──────────┤
│ Diffs        │ diffs    │ loaded   │ stock:diffs/index.ts                    │ 2026.3.1 │
│              │          │          │ Read-only diff viewer and PNG renderer  │          │
│              │          │          │ for agents.                             │          │
└──────────────┴──────────┴──────────┴─────────────────────────────────────────┴──────────┘
```

> ❌ **失败情况：**
> ```
> [plugins] diffs failed to load from .../extensions/diffs/index.ts
> Error: Cannot find module '@pierre/diffs'
> ```
> **解决：** 执行 `npm install -g @pierre/diffs` 然后重启 gateway

---

## 🧪 六、测试 Diffs 工具

### 6.1 测试输入数据

创建测试文件 `/tmp/test-diff-input.json`：

```json
{
  "before": "# Hello World\n\nThis is the original text.\n\n- Item 1\n- Item 2",
  "after": "# Hello World\n\nThis is the modified text.\n\n- Item 1\n- Item 2\n- Item 3 (new)",
  "path": "test.md",
  "mode": "both",
  "layout": "unified",
  "theme": "dark"
}
```

### 6.2 通过 Agent 调用

在会话中让 Agent 使用 diffs 工具：

```
请帮我对比以下两段文本的差异：

原文：
# Hello World

This is the original text.

- Item 1
- Item 2

修改后：
# Hello World

This is the modified text.

- Item 1
- Item 2
- Item 3 (new)
```

### 6.3 预期输出

Agent 调用 diffs 工具后返回：

```json
{
  "details": {
    "viewerUrl": "http://127.0.0.1:18789/plugins/diffs/view/xxx-xxx-xxx",
    "viewerPath": "/plugins/diffs/view/xxx-xxx-xxx",
    "imagePath": "/tmp/openclaw-diffs-xxx.png"
  }
}
```

### 6.4 使用输出

**查看交互式 diff：**
```bash
openclaw canvas present --url "http://127.0.0.1:18789/plugins/diffs/view/xxx-xxx-xxx"
```

**发送 PNG 图片：**
```bash
openclaw message send --channel telegram --target "xxx" --path "/tmp/openclaw-diffs-xxx.png"
```

---

## 📝 七、工具参数说明

### 7.1 输入模式

| 模式 | 说明 | 示例 |
|------|------|------|
| **before/after** | 直接提供修改前后文本 | `{"before": "...", "after": "..."}` |
| **patch** | 提供统一补丁格式 | `{"patch": "diff --git..."}` |

### 7.2 输出模式

| Mode | 返回值 | 用途 |
|------|--------|------|
| `view` | `viewerUrl`, `viewerPath` | Canvas 交互查看 |
| `image` | `imagePath` | PNG 图片导出 |
| `both` | 全部 | 同时需要 URL 和图片 |

### 7.3 可选参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `layout` | `unified` | `unified` 或 `split` |
| `theme` | `dark` | `light` 或 `dark` |
| `showLineNumbers` | `true` | 显示行号 |
| `expandUnchanged` | `false` | 展开未修改部分 |
| `path` | - | 文件名（用于显示） |
| `title` | - | 自定义标题 |
| `ttlSeconds` | 1800 | 查看器有效期（秒） |
| `baseUrl` | - | 自定义 Gateway 基础 URL |

---

## 🎨 八、插件默认配置

在 `openclaw.json` 中配置全局默认值：

```json5
{
  "plugins": {
    "entries": {
      "diffs": {
        "enabled": true,
        "config": {
          "defaults": {
            "fontFamily": "Fira Code",
            "fontSize": 15,
            "lineSpacing": 1.6,
            "layout": "unified",
            "showLineNumbers": true,
            "diffIndicators": "bars",
            "wordWrap": true,
            "background": true,
            "theme": "dark",
            "mode": "both"
          }
        }
      }
    }
  }
}
```

> 💡 **提示：** 工具调用时的参数会覆盖插件默认值

---

## 🔍 九、故障排查

### 问题 1：插件加载失败

**错误：**
```
[plugins] diffs failed to load: Cannot find module '@pierre/diffs'
```

**解决：**
```bash
npm install -g @pierre/diffs
openclaw gateway restart
```

### 问题 2：PNG 渲染失败

**原因：** 缺少 Chromium 浏览器

**解决：**
```bash
# macOS
brew install --cask chromium

# 或在 openclaw.json 中指定路径
{
  "browser": {
    "executablePath": "/Applications/Chromium.app/Contents/MacOS/Chromium"
  }
}
```

### 问题 3：查看器 URL 无法访问

**检查：**
```bash
openclaw gateway status
curl http://127.0.0.1:18789/health
```

**可能原因：**
- Gateway 未启动
- 端口被占用
- 防火墙阻止

---

## 📚 十、相关文档

- **官方文档：** `/opt/homebrew/lib/node_modules/openclaw/docs/tools/diffs.md`
- **GitHub Release:** https://github.com/openclaw/openclaw/releases/tag/v2026.3.1
- **Diffs 库:** https://diffs.com

---

## 🎯 十一、使用场景

### 11.1 代码审查

```json
{
  "before": "const x = 1;",
  "after": "const x = 2;",
  "path": "src/example.ts",
  "mode": "view"
}
```

### 11.2 配置对比

```json
{
  "patch": "diff --git a/config.json b/config.json\n--- a/config.json\n+++ b/config.json\n@@ -1,5 +1,5 @@\n {\n-  \"port\": 8080,\n+  \"port\": 9090,\n   \"host\": \"localhost\"\n }",
  "mode": "both"
}
```

### 11.3 文档版本

```json
{
  "before": "# API 文档\n\n版本：1.0",
  "after": "# API 文档\n\n版本：2.0\n\n## 更新内容\n\n- 新增用户接口\n- 修复认证问题",
  "path": "docs/api.md",
  "mode": "image"
}
```

---

## ✅ 完成检查清单

- [ ] OpenClaw 版本 ≥ 2026.3.1
- [ ] 安装 `@pierre/diffs` 包
- [ ] 在 `openclaw.json` 中启用插件
- [ ] 重启 Gateway
- [ ] 验证插件状态为 `loaded`
- [ ] 测试 before/after 模式
- [ ] 测试 patch 模式
- [ ] 验证 viewer URL 可访问
- [ ] 验证 PNG 图片生成

---

**报告生成：** 大白 🐻‍❄️  
**最后更新：** 2026-03-02
