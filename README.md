# Second Brain

> **你的第二大脑 - 存储你的一切，理解你的思考，在你需要时精准呈现。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status: Development](https://img.shields.io/badge/Status-Development-red)](https://github.com/XueLuo9527/second-brain)

---

## 🎯 项目愿景

Second Brain 是一个**专注于个人知识管理**的应用，聚焦两个核心功能：

1. **📦 存储** - 统一保存你的文件、笔记、链接
2. **🔍 检索** - AI 驱动的智能搜索，精准找到你需要的内容

### 我们不做的事情

- ❌ 不做协作（聚焦个人体验）
- ❌ 不做社交（不分享、不公开）
- ❌ 不做全能（不做任务管理、日历、通讯）
- ❌ 不锁定用户（开放格式，数据随时导出）

---

## 🚀 核心特性

### 1. 统一存储

- 📄 **多格式支持** - PDF、图片、文档、Markdown、网页
- 🏷️ **智能分类** - 文件夹 + 标签双重管理
- 🔒 **隐私优先** - 本地存储优先，数据你完全掌控
- ☁️ **可选同步** - 云端备份（可选付费功能）

### 2. 智能检索

- 🔎 **全文搜索** - 标题、内容、标签一键搜索
- 🧠 **语义搜索** - AI 理解你的意图，不只是关键词匹配
- 💡 **智能建议** - "你可能在找..."
- 📊 **知识图谱** - 可视化展示笔记关联

### 3. 笔记编辑

- ✍️ **Markdown 编辑** - 支持富文本切换
- 🔗 **双向链接** - [[笔记标题]] 轻松关联
- 🤖 **AI 辅助** - 自动总结、续写、润色
- 💾 **自动保存** - 再也不怕丢失

---

## 📊 与竞品对比

| 功能 | Second Brain | Obsidian | Notion |
|------|-------------|----------|--------|
| **核心定位** | 个人存储 + 检索 | 本地知识库 | 全能工作空间 |
| **数据存储** | 本地优先 | 本地 | 云端 |
| **搜索能力** | ⭐⭐⭐⭐⭐ (AI 驱动) | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **隐私保护** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **学习成本** | 低 | 中 | 高 |
| **价格** | 免费 + 可选付费 | 免费 + 付费 | 免费 + 付费 |

---

## 🏗️ 技术栈

### 前端

- **框架：** Next.js 15 (App Router)
- **语言：** TypeScript
- **UI：** Tailwind CSS + shadcn/ui
- **编辑器：** TipTap (富文本) / Markdoc (Markdown)
- **可视化：** three.js / D3.js (知识图谱)

### 后端

- **本地方案：** Tauri (Rust) + SQLite
- **云端方案：** Next.js API + PostgreSQL (Supabase)
- **搜索：** SQLite FTS5 + Qdrant (向量搜索)

### AI 能力

- **语义搜索：** sentence-transformers (本地嵌入)
- **AI 问答：** RAG + Llama 3 / Qwen
- **OCR：** Tesseract.js

---

## 📦 开发路线图

### Phase 1: MVP（2026 Q2）

- [ ] 用户认证
- [ ] 文件上传和管理
- [ ] Markdown 笔记编辑
- [ ] 基础搜索（全文）
- [ ] 本地存储
- [ ] 基础知识图谱

**预计完成：** 2026-06

### Phase 2: AI 增强（2026 Q3）

- [ ] 语义搜索
- [ ] AI 问答
- [ ] 智能标签
- [ ] 相关笔记推荐

**预计完成：** 2026-09

### Phase 3: 增长（2026 Q4）

- [ ] 浏览器扩展
- [ ] 移动端 App
- [ ] API 和集成
- [ ] 高级分析

**预计完成：** 2026-12

---

## 🚀 快速开始

### 开发环境

```bash
# 克隆仓库
git clone https://github.com/XueLuo9527/second-brain.git
cd second-brain

# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 访问 http://localhost:3000
```

### 构建生产版本

```bash
npm run build
npm start
```

---

## 📖 详细文档

- **[产品调研报告](./PRODUCT_RESEARCH.md)** - 市场分析、竞品对比、商业模式
- **[技术架构](./docs/ARCHITECTURE.md)** - 系统设计、技术选型
- **[API 文档](./docs/API.md)** - REST API 接口说明
- **[用户指南](./docs/USER_GUIDE.md)** - 使用教程

---

## 🤝 参与贡献

我们欢迎各种形式的贡献！

### 贡献方式

1. **🐛 报告 Bug** - 提交 Issue
2. **💡 功能建议** - 提交 Feature Request
3. **📝 文档改进** - PR 修改文档
4. **💻 代码贡献** - PR 提交代码

### 贡献流程

```
1. Fork 仓库
2. 创建功能分支 (git checkout -b feature/AmazingFeature)
3. 提交更改 (git commit -m 'Add some AmazingFeature')
4. 推送到分支 (git push origin feature/AmazingFeature)
5. 开启 Pull Request
```

---

## 📄 开源协议

MIT License - 详见 [LICENSE](./LICENSE) 文件

---

## 📞 联系方式

- **项目仓库：** https://github.com/XueLuo9527/second-brain
- **问题反馈：** https://github.com/XueLuo9527/second-brain/issues
- **讨论区：** https://github.com/XueLuo9527/second-brain/discussions

---

## 🙏 致谢

感谢以下开源项目：

- [Next.js](https://nextjs.org/) - React 框架
- [Tailwind CSS](https://tailwindcss.com/) - 原子化 CSS
- [TipTap](https://tiptap.dev/) - 富文本编辑器
- [Qdrant](https://qdrant.tech/) - 向量数据库
- [Obsidian](https://obsidian.md/) - 灵感来源

---

**Made with ❤️ by Second Brain Team**

*最后更新：2026-03-14*
