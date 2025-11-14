# Blog Workflow Template 使用指南

这是一个基于 n8n 的自动化博客发布工作流模板,可以帮助你实现:

1. **自动选题** - 基于关键词库和已有内容,智能选择新主题
2. **内容研究** - 自动抓取 Google Top 3 文章作为参考
3. **AI 写作** - 生成 SEO 优化的高质量博客文章
4. **自动发布** - 推送到 GitHub 触发自动部署
5. **通知提醒** - 通过飞书机器人发送成功通知

---

## 工作流架构

```
定时触发 (每3天)
    ↓
列出已有博客 (GitHub API)
    ↓
提取标题列表 (Code)
    ↓
AI 选题规划 (OpenRouter + 结构化输出)
    ↓
抓取参考文章 (Firecrawl API)
    ↓
AI 内容创作 (OpenRouter)
    ↓
创建博客文件 (GitHub API)
    ↓
等待部署 (1分钟)
    ↓
发送通知 (飞书 Webhook)
```

---

## 配置步骤

### 1. 准备 API 凭证

你需要准备以下 API 凭证:

#### 必需:
- **GitHub Personal Access Token** - 用于读取和创建文件
  - 权限: `repo` (完整仓库访问)
  - 获取方式: GitHub Settings → Developer settings → Personal access tokens

- **OpenRouter API Key** - 用于调用 AI 模型
  - 注册地址: https://openrouter.ai
  - 推荐模型: `minimax/minimax-m2`

- **Firecrawl API Token** - 用于抓取参考文章
  - 注册地址: https://firecrawl.dev
  - 免费额度: 500 次/月

#### 可选:
- **飞书 Webhook URL** - 用于发送通知
  - 创建方式: 飞书群 → 群设置 → 机器人 → 添加自定义机器人

---

### 2. 替换占位符

在工作流 JSON 文件中,搜索并替换以下占位符:

#### API 凭证相关:
```
YOUR_GITHUB_CREDENTIAL_ID       → 你的 GitHub 凭证 ID
YOUR_OPENROUTER_CREDENTIAL_ID   → 你的 OpenRouter 凭证 ID
YOUR_FIRECRAWL_API_TOKEN        → 你的 Firecrawl API Token
YOUR_FEISHU_WEBHOOK_URL         → 你的飞书 Webhook URL (可选)
```

#### 项目配置相关:
```
YOUR_GITHUB_USERNAME            → 你的 GitHub 用户名
YOUR_REPO_NAME                  → 你的仓库名称
YOUR_BLOG_DIRECTORY             → 博客文件存放目录 (如 content/blog)
your-domain.com                 → 你的网站域名
cdn.your-domain.com             → 你的 CDN 域名
```

#### 产品信息相关 (在提示词中):
```
[YOUR_PRODUCT_NAME]             → 你的产品名称
[YOUR_PRODUCT_DESCRIPTION]      → 产品定位描述
[FEATURE_1] - [FEATURE_5]       → 核心功能列表
[LIMITATION_1] - [LIMITATION_2] → 产品限制 (诚实说明)
[USER_GROUP_1] - [USER_GROUP_3] → 目标用户群体
[PRICING_TIER_1] - [PRICING_TIER_3] → 定价方案
[TECH_ADVANTAGE_1] - [TECH_ADVANTAGE_5] → 技术优势
```

#### 关键词库相关:
```
[KEYWORD_CATEGORY_1]            → 关键词分类 (如 "TikTok 系列")
[keyword_1] - [keyword_n]       → 具体关键词
[long_tail_keyword_1]           → 长尾关键词
```

---

### 3. 配置关键词库

关键词库是工作流的核心,直接影响选题质量。建议按以下结构组织:

#### 关键词分类标准:

```markdown
#### 🔥 超高价值关键词 (月搜索量 > 10K)
- **[分类名称]** (总搜索量):
  - `primary keyword 1`
  - `primary keyword 2`

#### 🌟 高价值关键词 (月搜索量 1K-10K)
- **[分类名称]** (总搜索量):
  - `medium keyword 1`
  - `medium keyword 2`

#### 💡 中等价值关键词 (月搜索量 100-1K)
- **[分类名称]** (总搜索量):
  - `low keyword 1`
  - `low keyword 2`

#### 🆕 新兴/长尾关键词
- `long tail keyword 1`
- `long tail keyword 2`
```

#### 如何获取关键词数据:

1. **免费工具**:
   - Google Keyword Planner
   - Ubersuggest (免费额度)
   - AnswerThePublic

2. **付费工具** (更精准):
   - Ahrefs
   - SEMrush
   - Moz Keyword Explorer

---

### 4. 自定义提示词

工作流包含两个核心提示词:

#### 提示词 1: 选题规划器 (AI Agent - Topic Planner)

**功能**: 分析已有内容 + 关键词库 → 输出新选题

**可自定义内容**:
- 产品信息 (定位、功能、用户、定价)
- 关键词库 (按优先级分类)
- 内容角度库 (Tutorial, Comparison, Review, Tips, Workflow, FAQ, News)
- 选题决策逻辑

**输出格式**: JSON (结构化输出)
```json
{
  "title": "...",
  "slug": "...",
  "primaryKeyword": "...",
  "secondaryKeywords": [...],
  "contentAngle": "...",
  "searchIntent": "...",
  "targetAudience": "...",
  "uniqueValue": "...",
  "suggestedSearchQuery": "...",
  "estimatedSearchVolume": "...",
  "difficultyScore": "...",
  "reasoning": "..."
}
```

#### 提示词 2: 内容创作器 (AI Agent - Content Writer)

**功能**: 参考文章 + 选题信息 + 产品信息 → 输出完整博客

**可自定义内容**:
- 产品详细信息 (功能、优势、限制、定价)
- 写作风格和语气
- 文章结构模板
- SEO 优化规则
- 图片素材路径
- 内部链接策略

**输出格式**: Markdown (包含 YAML Frontmatter)

---

### 5. 配置 Frontmatter 格式

根据你的博客系统,调整 Frontmatter 格式:

**当前模板** (通用格式):
```yaml
---
title: "..."
description: "..."
date: "2025-XX-XX"
updated: "2025-XX-XX"
category: ...
tags: [...]
cover: ...
author:
  name: ...
  avatar: ...
slug: ...
---
```

**常见变体**:

- **Next.js 博客**:
  ```yaml
  ---
  title: "..."
  excerpt: "..."
  coverImage: "..."
  date: "2025-XX-XX"
  author:
    name: "..."
    picture: "..."
  ogImage:
    url: "..."
  ---
  ```

- **Hugo**:
  ```yaml
  ---
  title: "..."
  date: 2025-XX-XX
  draft: false
  tags: [...]
  categories: [...]
  featured_image: "..."
  ---
  ```

- **Jekyll**:
  ```yaml
  ---
  layout: post
  title: "..."
  date: 2025-XX-XX
  categories: [...]
  tags: [...]
  image: "..."
  ---
  ```

---

### 6. 测试工作流

在正式启用前,建议测试每个节点:

1. **手动触发** - 点击 "Execute Workflow" 按钮
2. **检查节点输出** - 确保每个节点都正常工作
3. **验证 API 调用** - 检查 API 额度消耗
4. **查看生成内容** - 确认文章质量符合预期
5. **测试 GitHub 推送** - 验证文件是否正确创建

---

## 成本估算

基于每 3 天运行一次 (每月约 10 次):

| 服务 | 免费额度 | 超出成本 | 月成本估算 |
|------|---------|---------|----------|
| OpenRouter (minimax-m2:free) | 无限 | $0 | **$0** |
| Firecrawl | 500次/月 | $0.005/次 | **$0** (30次 < 500) |
| GitHub API | 5000次/小时 | 免费 | **$0** |
| 飞书 Webhook | 无限 | 免费 | **$0** |

**总计**: **$0/月** (使用免费额度)

---

## 常见问题

### Q1: 如何更改发布频率?

修改 "Schedule Trigger" 节点的 `daysInterval` 参数:
```json
"daysInterval": 3  // 每 3 天触发一次
```

### Q2: 如何禁用飞书通知?

删除或禁用 "Feishu - Send Notification" 节点即可。

### Q3: 如何更换 AI 模型?

修改 "OpenRouter Chat Model" 节点的 `model` 参数:
```json
"model": "anthropic/claude-3.5-sonnet"  // 付费但质量更高
```

推荐免费模型:
- `minimax/minimax-m2:free` (当前使用)
- `google/gemini-flash-1.5` (速度快)
- `meta-llama/llama-3.1-8b-instruct:free` (开源)

### Q4: 生成的文章质量不高怎么办?

**优化建议**:
1. **完善关键词库** - 添加更多高质量关键词
2. **优化提示词** - 提供更详细的产品信息和写作示例
3. **更换模型** - 使用付费模型 (如 GPT-4, Claude)
4. **人工润色** - 将 AI 生成内容作为初稿,手动优化

### Q5: 如何避免重复选题?

工作流已内置去重逻辑:
- 读取已有博客文件列表
- 提取已有标题
- AI 选题时排除已覆盖主题

如需更精细控制,可修改 "Extract Existing Blog Titles" 节点的代码。

### Q6: 能否支持多语言博客?

可以! 需要:
1. 创建多个工作流实例 (每个语言一个)
2. 修改提示词为对应语言
3. 调整 `filePath` 为不同目录 (如 `content/blog/en`, `content/blog/zh`)

---

## 高级优化

### 1. 添加内容审核

在 "AI Agent - Content Writer" 和 "GitHub - Create Blog Post" 之间插入审核节点:

```javascript
// Code 节点示例
const content = $json.output;

// 检查文章长度
if (content.length < 1000) {
  throw new Error('Article too short');
}

// 检查关键词密度
const keyword = $('AI Agent - Topic Planner').item.json.output.primaryKeyword;
const density = (content.match(new RegExp(keyword, 'gi')) || []).length / content.split(' ').length;
if (density < 0.01 || density > 0.03) {
  throw new Error('Keyword density out of range');
}

return { json: { approved: true } };
```

### 2. 批量生成

修改 "Schedule Trigger" 为 "Manual Trigger",配合 "Loop Over Items" 节点:

```javascript
// 生成 5 个选题
const topics = [];
for (let i = 0; i < 5; i++) {
  topics.push({ index: i });
}
return topics.map(t => ({ json: t }));
```

### 3. A/B 测试标题

在发布前生成多个标题变体,选择最优:

```javascript
// 修改提示词要求输出 3 个标题候选
"titleVariants": [
  "How to Remove AI Watermark (2025 Guide)",
  "5 Ways to Remove AI Video Watermarks Easily",
  "Best AI Watermark Remover Tools in 2025"
]
```

### 4. 集成 SEO 分析

使用 Google Search Console API 分析发布后的表现:
- 展示次数
- 点击率
- 平均排名
- 关键词覆盖

---

## 维护建议

### 每周:
- [ ] 检查 API 额度使用情况
- [ ] 审查生成的文章质量
- [ ] 更新关键词库 (新增热门词)

### 每月:
- [ ] 分析博客流量和排名
- [ ] 优化表现不佳的文章
- [ ] 调整选题策略

### 每季度:
- [ ] 评估工作流 ROI
- [ ] 更新产品信息
- [ ] 重构提示词模板

---

## 许可和免责声明

**使用条款**:
- ✅ 允许个人和商业使用
- ✅ 允许修改和二次开发
- ✅ 允许分享和传播

**免责声明**:
- AI 生成的内容需人工审核
- 请遵守各平台的服务条款
- 确保内容符合版权法规
- 作者不对内容质量和法律问题负责

---

## 参考资源

**n8n 官方文档**:
- https://docs.n8n.io

**SEO 最佳实践**:
- Google Search Central: https://developers.google.com/search
- Moz Beginner's Guide: https://moz.com/beginners-guide-to-seo

**AI 写作工具对比**:
- OpenRouter Models: https://openrouter.ai/models
- LLM Leaderboard: https://huggingface.co/spaces/lmsys/chatbot-arena-leaderboard

---

**祝你的博客流量飞升! 🚀**

如有问题,欢迎提 Issue 或 PR。
