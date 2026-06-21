# 快速开始

## 安装依赖

```bash
pnpm install --shamefully-hoist
```

如果遇到 sharp 安装问题，尝试：
```bash
pnpm install --ignore-scripts
pnpm rebuild sharp
```

## 本地开发

```bash
pnpm run dev
```

浏览器访问：http://localhost:4321

## 写文章

在 `src/content/posts/` 目录创建新的 `.md` 文件：

```markdown
---
title: 文章标题
published: 2025-06-21
description: 文章描述
tags: [标签1, 标签2]
category: 分类名
draft: false
---

你的文章内容...
```

## 构建部署

```bash
pnpm run build
```

构建产物在 `dist/` 目录。

推送到 GitHub 后会自动部署到：https://dydydd.github.io

## 配置文件

所有配置在 `src/config/` 目录：

- `siteConfig.ts` - 站点基本配置
- `profileConfig.ts` - 个人资料
- `navBarConfig.ts` - 导航栏
- `commentConfig.ts` - 评论系统
- 更多...

## 文档

完整文档：https://docs-firefly.cuteleaf.cn
