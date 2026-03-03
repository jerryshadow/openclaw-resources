# OpenClaw 配置差异报告

**对比时间：** 2026-03-02 16:03 GMT+8  
**Git 版本：** HEAD (上个提交)  
**当前版本：** ~/.openclaw/openclaw.json

---

## 📊 差异摘要

| 文件 | Git 版本大小 | 当前大小 | 变化 |
|------|-------------|---------|------|
| `openclaw.json` | 9,251 bytes | 9,509 bytes | **+258 bytes** |

**差异行数：** 2 处变更

---

## 🔍 详细差异

### 变更 1：Xiaobu Agent Heartbeat 间隔调整

**位置：** `agents.list[].heartbeat.every`

```diff
- "every": "30m",
+ "every": "10m",
```

**说明：**
- **之前：** 每 30 分钟执行一次 heartbeat
- **现在：** 每 10 分钟执行一次 heartbeat
- **影响：** Xiaobu 的 AI 新闻监控任务频率提高到 3 倍

**查看对比：**
- 交互式：http://127.0.0.1:18789/plugins/diffs/view/222c925de4ec25c03355/643bb370be02e1bf774a2f96929860a38cad9e139a49cad6
- 模式：unified
- 主题：dark

---

### 变更 2：启用 Diffs 插件

**位置：** `plugins` 配置段

#### 2.1 plugins.allow 列表

```diff
  "plugins": {
    "allow": [
      "memory-lancedb-pro",
-      "telegram"
+      "telegram",
+      "diffs"
    ],
```

#### 2.2 plugins.entries 配置

```diff
    "entries": {
      "telegram": {
        "enabled": true
+      },
+      "diffs": {
+        "enabled": true,
+        "config": {
+          "defaults": {
+            "layout": "unified",
+            "theme": "dark",
+            "showLineNumbers": true,
+            "mode": "both"
+          }
+        }
      },
```

**说明：**
- **新增插件：** `diffs` (v2026.3.1)
- **功能：** 只读 diff 查看器和 PNG 渲染
- **默认配置：**
  - `layout`: unified (统一视图)
  - `theme`: dark (深色主题)
  - `showLineNumbers`: true (显示行号)
  - `mode`: both (同时生成 URL 和图片)

**查看对比：**
- 交互式：http://127.0.0.1:18789/plugins/diffs/view/b47b37b5934b7b3a8a0f/b69e1d68e05a611b6f3e782af54b464eca1d50a7fd1810e4
- 模式：split (分屏对比)
- 主题：dark

---

## 📝 完整 Diff (Unified Patch)

```diff
--- a/openclaw.json
+++ b/openclaw.json
@@ -222,7 +222,7 @@
         "workspace": "/Users/shadowyingyan/.openclaw/workspace-xiaobu",
         "agentDir": "/Users/shadowyingyan/.openclaw/agents/xiaobu/agent",
         "heartbeat": {
-          "every": "30m",
+          "every": "10m",
           "target": "telegram",
           "accountId": "xiaobu",
           "to": "-1003740708256:topic:2",
@@ -341,7 +341,8 @@
   "plugins": {
     "allow": [
       "memory-lancedb-pro",
-      "telegram"
+      "telegram",
+      "diffs"
     ],
     "slots": {
       "memory": "memory-lancedb-pro"
@@ -349,6 +350,17 @@
     "entries": {
       "telegram": {
         "enabled": true
+      },
+      "diffs": {
+        "enabled": true,
+        "config": {
+          "defaults": {
+            "layout": "unified",
+            "theme": "dark",
+            "showLineNumbers": true,
+            "mode": "both"
+          }
+        }
       },
       "memory-lancedb-pro": {
         "enabled": true,
```

---

## ✅ 影响评估

| 变更 | 影响范围 | 风险等级 | 需要重启 |
|------|---------|---------|---------|
| Heartbeat 间隔 | Xiaobu Agent | 低 | 否 |
| Diffs 插件 | 全局工具 | 低 | 是 ✅ |

---

## 🔄 操作历史

1. **编辑配置** - 添加 diffs 插件配置
2. **重启 Gateway** - `openclaw gateway restart`
3. **验证加载** - `openclaw plugins list` → `diffs: loaded`
4. **生成报告** - 使用 diffs 工具对比配置差异

---

## 📌 备注

- 两个变更都是**新增功能**，没有删除或修改现有功能
- Diffs 插件已成功加载并可用
- Heartbeat 频率提高可能会增加 API 调用次数

---

**生成工具：** OpenClaw Diffs Plugin  
**报告格式：** Markdown + Interactive Diff Views
