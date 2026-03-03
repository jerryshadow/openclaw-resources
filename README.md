# OpenClaw 资料库 🦞

> OpenClaw 学习资料、配置指南与技能合集

---

## 📚 目录

- [快速开始](#快速开始)
- [配置指南](#配置指南)
- [技能合集](#技能合集)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

---

## 快速开始

### 安装 OpenClaw

```bash
npm install -g openclaw
```

### 初始化配置

```bash
openclaw configure
```

### 启动 Gateway

```bash
openclaw gateway start
```

### 访问控制台

```bash
openclaw dashboard
```

---

## 配置指南

### 1. 多模型配置

支持阿里云百炼、MiniMax、OpenAI 等多个模型提供商。

### 2. 渠道接入

- ✅ Telegram
- ✅ Discord
- ✅ 飞书
- ✅ 钉钉
- ✅ 企业微信

### 3. 插件系统

- memory-lancedb-pro - 向量记忆存储
- diffs - 代码差异可视化
- mcporter - MCP 协议支持

---

## 技能合集

### 内置技能

| 技能 | 说明 |
|------|------|
| web_search | 网络搜索 |
| web_fetch | 网页抓取 |
| browser | 浏览器自动化 |
| canvas | 画布渲染 |
| cron | 定时任务 |
| message | 消息发送 |

### 社区技能

- codex-deep-search - 深度网络搜索
- claude-code-hooks - Claude Code 集成
- xiaohongshu-skill - 小红书自动化
- humanizer - AI 文本优化

---

## 常见问题

### Q: Gateway 启动失败？

```bash
openclaw gateway status
openclaw doctor
```

### Q: 如何查看日志？

```bash
openclaw logs
tail -f /tmp/openclaw/openclaw-*.log
```

### Q: 如何重置配置？

```bash
openclaw configure
```

---

## 相关资源

- [官方文档](https://docs.openclaw.ai)
- [GitHub 仓库](https://github.com/openclaw/openclaw)
- [技能商店](https://clawhub.com)
- [Discord 社区](https://discord.gg/clawd)

---

## 许可证

MIT License

---

**最后更新：** 2026-03-03
