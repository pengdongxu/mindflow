# MindFlow 博客仓库设置指南

这个文件夹包含了您需要上传到 GitHub 仓库 `pengdongxu/mindflow` 的所有文件。

## 📁 文件结构

```
mindflow/
├── blog-index.json          # 博客索引文件(必需)
├── posts/                   # 博客文章目录
│   ├── transformer-evolution.md
│   ├── llm-finetuning.md
│   ├── ai-agent-design.md
│   └── tech-humanity.md
└── README.md               # 仓库说明(可选)
```

## 🚀 部署步骤

### 1. 将文件上传到 GitHub

您有两种方式上传文件:

#### 方式A: 使用 Git 命令行

```bash
# 克隆您的仓库
git clone https://github.com/pengdongxu/mindflow.git
cd mindflow

# 复制文件到仓库目录
# (将 mindflow-setup 文件夹中的所有文件复制到 mindflow 目录)

# 提交并推送
git add .
git commit -m "Add blog posts and index"
git push origin main
```

#### 方式B: 使用 GitHub 网页界面

1. 访问 https://github.com/pengdongxu/mindflow
2. 点击 "Add file" → "Upload files"
3. 上传 `blog-index.json` 和 `posts` 文件夹
4. 点击 "Commit changes"

### 2. 验证部署

上传完成后,访问以下URL验证文件是否可访问:

- **索引文件**: https://raw.githubusercontent.com/pengdongxu/mindflow/main/blog-index.json
- **示例文章**: https://raw.githubusercontent.com/pengdongxu/mindflow/main/posts/transformer-evolution.md

### 3. 测试网站

访问您的网站:
- **首页**: https://coldlight.cn/ (查看博客卡片)
- **博客详情**: https://coldlight.cn/blog-detail.html?post=transformer-evolution

## 📝 添加新博客文章

### 1. 创建 Markdown 文件

在 `posts/` 目录下创建新的 `.md` 文件,格式如下:

```markdown
---
title: 文章标题
date: 2026-01-20
category: deep-learning
tags: [标签1, 标签2]
---

# 文章标题

文章内容...
```

### 2. 更新 blog-index.json

在 `posts` 数组中添加新文章的元数据:

```json
{
  "id": "article-slug",
  "title": "文章标题",
  "title_en": "Article Title",
  "excerpt": "文章摘要...",
  "excerpt_en": "Article excerpt...",
  "category": "deep-learning",
  "date": "2026-01-20",
  "readTime": 8,
  "featured": false,
  "tags": ["标签1", "标签2"],
  "file": "posts/article-slug.md"
}
```

### 3. 提交更新

```bash
git add .
git commit -m "Add new blog post: article-slug"
git push origin main
```

## 🎨 分类说明

当前支持的分类:

- `deep-learning`: 深度学习 / Deep Learning 🧠
- `practice`: 实践经验 / Practice 💡
- `insight`: 技术洞察 / Technical Insights 🔍
- `thoughts`: 日常思考 / Thoughts 💭

您可以在 `blog-index.json` 的 `categories` 部分添加新分类。

## 🔧 故障排除

### 问题1: 博客文章不显示

**检查:**
1. `blog-index.json` 格式是否正确(使用 JSON 验证器)
2. 文件路径是否正确(区分大小写)
3. GitHub 仓库是否为公开(Public)

### 问题2: 文章内容加载失败

**检查:**
1. Markdown 文件是否存在于 `posts/` 目录
2. 文件名是否与 `blog-index.json` 中的 `file` 字段匹配
3. 浏览器控制台是否有错误信息

### 问题3: 中文显示乱码

**确保:**
- 所有文件使用 UTF-8 编码保存
- Git 配置正确处理 UTF-8: `git config --global core.quotepath false`

## 📞 需要帮助?

如果遇到问题,请检查:
1. 浏览器开发者工具的 Console 标签页
2. Network 标签页查看请求是否成功
3. GitHub 仓库的 Actions 标签页(如果启用了 CI/CD)

---

**祝您博客运行顺利!** 🎉
