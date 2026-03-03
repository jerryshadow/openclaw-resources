# OpenClaw

> OpenClaw 配置、使用与插件相关

## 关联主题

- [[browser-plugins|浏览器插件]] - 浏览器控制插件
- [[rules|规则/铁律]] - OpenClaw 使用规则

## 重要笔记

### memory-lancedb-pro 插件配置 (2026-02-27)

- 插件位置: `~/.openclaw/workspace/plugins/memory-lancedb-pro`
- 数据库路径: `~/.openclaw/memory/lancedb-pro`
- 配置: 已配置 Jina embedding + reranker
- 特性: 混合检索 (Vector + BM25), 跨编码器 Rerank, 多 Scope 隔离

### 关键配置项

```json
{
  "embedding": {
    "provider": "openai-compatible",
    "model": "jina-embeddings-v3",
    "baseURL": "https://api.jina.ai/v1",
    "dimensions": 1024
  },
  "retrieval": {
    "mode": "hybrid",
    "rerank": "cross-encoder",
    "rerankModel": "jina-reranker-v2-base-multilingual"
  }
}
```

### 插件代码修改后必须清 jiti 缓存

修改 `.ts` 文件后需执行:
```bash
rm -rf /tmp/jiti/
openclaw gateway restart
```

### Codex Deep Search Skill (2026-02-27)

- Skill 位置: `~/.openclaw/workspace/skills/codex-deep-search`
- 结果目录: `~/.openclaw/data/codex-search-results`
- 回调群组: `-5279273879`
- 使用方式:
  ```bash
  # 后台模式（推荐）- 自动回调到群组
  nohup bash ~/.openclaw/workspace/skills/codex-deep-search/scripts/search.sh \
    --prompt "你的研究查询" \
    --telegram-group "-5279273879" \
    --timeout 120 &
  
  # 同步模式（短查询）
  bash ~/.openclaw/workspace/skills/codex-deep-search/scripts/search.sh \
    --prompt "快速事实查询" \
    --output "/tmp/search-result.md" \
    --timeout 60
  ```
- 触发词: "deep search", "详细搜索", "帮我查一下"
- 支持参数: --prompt, --output, --model, --timeout, --telegram-group, --task-name

### Codex Deep Search 群组回调工作流程

**实现原理：**

1. **参数传入**: 通过 `--telegram-group` 传入目标群组 ID（负数表示群组）

2. **Telegram 消息发送**: 脚本完成后调用 OpenClaw CLI
   ```bash
   openclaw message send \
     --channel telegram \
     --target "-5279273879" \
     --message "$MSG"
   ```

3. **Wake Hook**: 同时发送 webhook 通知 OpenClaw 继续处理
   ```bash
   curl -X POST "http://localhost:18789/hooks/wake" \
     -H "Authorization: Bearer ${HOOK_TOKEN}" \
     -d "{\"text\":\"[DEEP_SEARCH_DONE]\",\"mode\":\"now\"}"
   ```

**工作流程图：**
```
用户触发 → 后台执行 Codex 搜索 → 完成后 
  → 发送 Telegram 消息到指定群组 
  → 同时发送 Wake Hook 通知 OpenClaw
```

**适用场景：** 异步任务完成后主动推送结果到指定群组

### Claude Code Hooks (2026-02-27)

- Skill 位置: `~/.openclaw/workspace/skills/claude-code-hooks`
- 结果目录: `~/.openclaw/data/claude-code-results`
- Claude CLI: `/Users/shadowyingyan/.local/bin/claude`
- Hook 脚本: `~/.claude/hooks/notify-agi.sh`

**使用方式:**
```bash
# 派发任务到 Claude Code，自动回调到 Telegram 群组
~/.openclaw/workspace/skills/claude-code-hooks/scripts/dispatch-claude-code.sh \
  -p "实现一个 Python 爬虫" \
  -n "my-scraper" \
  -g "-5279273879" \
  --workdir "/path/to/workdir"
```

**参数:**
- `-p, --prompt`: 任务提示（必需）
- `-n, --name`: 任务名称
- `-g, --group`: Telegram 群组 ID
- `-w, --workdir`: 工作目录

**工作流程:**
1. dispatch 脚本写入 task-meta.json
2. 启动 Claude Code (claude CLI)
3. Claude Code 完成后触发 Stop Hook
4. Hook 读取结果，写入 latest.json
5. 发送 Telegram 消息到指定群组
6. 写入 pending-wake.json 唤醒 AGI

### Humanizer Skill (2026-02-27)

- Skill 位置: `~/.openclaw/workspace/skills/humanizer`
- 功能: 去除 AI 写作痕迹，使文本更自然
- 使用: 当需要编辑或审阅文本，使其听起来更自然、像人写的时候使用

### SEO Content Writer (2026-02-27)

- Skill 位置: `~/.openclaw/workspace/skills/seo-content-writer`
- 功能: 生成 SEO 和 GEO 优化的内容
- 使用方式:
  ```
  /seo:write-content "主题" keyword="目标关键词" type="内容类型"
  ```
- 参数:
  - 主题（必需）
  - keyword="目标关键词"（必需）
  - type="内容类型"（可选，默认 blog post）

### Humanizer Skill (2026-02-27)

- Skill 位置: `~/.openclaw/workspace/skills/humanizer`
- 功能: 去除 AI 写作痕迹，使文本更自然
- 使用: 当需要编辑或审阅文本，使其听起来更自然、像人写的时候使用

### SEO 文章创作工作流程 (2026-02-27)

**完整流程：**

1. **研究选题**: 使用 web_search 收集相关主题和真实案例
   ```
   web_search "OpenClaw interesting use cases success stories"
   ```

2. **使用 SEO Content Writer 生成初稿**
   - 提供主题、关键词、内容类型
   - Skill 会生成符合 SEO 最佳实践的文章结构
   - 输出包含 Meta Description、SEO Metadata、CORE 自评分数

3. **使用 Humanizer 优化文风**
   - 根据 Humanizer 规则去除 AI 写作痕迹：
     - 删除夸张表达（"革命性"、"划时代"）
     - 减少空洞副词（"此外"、"值得注意的是"）
     - 使用更自然的句式
     - 加入个人化叙述
     - 缩短句子长度
     - 删除机械列表格式
   - 注意：Humanizer 优化建议用 write 而非 edit（文件太长精确匹配会失败）

4. **保存到本地**
   - 路径: `~/.openclaw/workspace/data/<标题>.md`

**输出示例:** `openclaw-use-cases.md` - 1850字关于 OpenClaw 真实用例的文章

> 双向链接: [[browser-plugins]], [[rules]], [[writing]], [[openclaw-cli]], [[agent-send]]

### MCPorter WebSearch MCP (2026-03-01)

- **安装**: `npm install -g mcporter`
- **配置**: `~/.openclaw/workspace/config/mcporter.json`
- **启用**: `openclaw config set skills.entries.mcporter.enabled true`
- **API Key**: 专用 MCP Key `sk-a3659cb5ac9a4a8e878dbdbc0f10e21a`
- **调用格式**: `mcporter call WebSearch.bailian_web_search query:"搜索内容" count:5`
- **⚠️ 注意**: 必须用命名参数 `query:"xxx" count:N`，不能用位置参数
- **文档**: [[2026-03-01-mcporter-websearch-config]]


> 双向链接：[[browser-plugins]], [[rules]], [[writing]], [[openclaw-cli]], [[agent-send]], [[2026-03-01-mcporter-websearch-config]]
