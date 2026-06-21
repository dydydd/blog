# Firefly 主题配置完成

## ✅ 已完成的配置

### 1. 站点基本信息（src/config/siteConfig.ts）
- **站点标题**: 人生实验室
- **站点副标题**: Li's Blog
- **站点 URL**: https://dydydd.github.io
- **站点描述**: 人生实验室 - 一个技术爱好者的个人博客，记录技术与生活的探索之旅。
- **主题色**: 色相 165（青绿色）
- **导航栏**: 使用科学实验图标 + "人生实验室" 标题
- **站点开始日期**: 2025-06-21

### 2. 个人资料配置（src/config/profileConfig.ts）
- **名字**: Li
- **签名**: 技术爱好者 / Tech enthusiast
- **社交链接**:
  - GitHub: https://github.com/dydydd
  - Email: admin@ld.ac.cn
  - RSS: /rss/

### 3. 导航栏配置（src/config/navBarConfig.ts）
简化的导航菜单：
- 主页
- 文章（归档、分类、标签）
- 友链
- 关于我

已禁用的页面：
- 打赏（sponsor: false）
- 留言板（guestbook: false）
- 番组计划（bangumi: false）
- 相册（gallery: false）

### 4. 关于页面（src/content/spec/about.md）
已更新为你的个人信息

### 5. GitHub Pages 部署（.github/workflows/deploy.yml）
- 使用 pnpm 9.14.4
- Node.js 22
- 自动构建并部署到 GitHub Pages

## 📝 下一步操作

### 1. 安装依赖并构建
```bash
pnpm install --shamefully-hoist
pnpm run build
```

**注意**: 如果 sharp 安装失败，可以尝试：
```bash
# 清理并重新安装
rm -rf node_modules pnpm-lock.yaml
pnpm install --shamefully-hoist

# 或者跳过 sharp 的构建（使用预编译二进制）
pnpm install --ignore-scripts
pnpm rebuild sharp
```

### 2. 本地预览
```bash
pnpm run dev
```
访问 http://localhost:4321 查看你的博客

### 3. 创建第一篇文章
在 `src/content/posts/` 目录下创建 `.md` 文件：

```markdown
---
title: 我的第一篇文章
published: 2025-06-21
description: 这是我的第一篇博客文章
tags: [随笔]
category: 生活
draft: false
---

文章内容...
```

### 4. 提交并推送到 GitHub
```bash
git add .
git commit -m "feat: 配置 Firefly 主题"
git push origin master
```

GitHub Actions 会自动构建并部署到 https://dydydd.github.io

### 5. 启用 GitHub Pages
1. 进入仓库 Settings → Pages
2. Source 选择 "GitHub Actions"
3. 等待部署完成

## 🎨 进一步自定义

### 更换头像
替换 `src/assets/images/avatar.avif` 文件

### 自定义主题色
编辑 `src/config/siteConfig.ts` 中的 `themeColor.hue` 值（0-360）

### 启用评论系统
编辑 `src/config/commentConfig.ts`，支持：
- Twikoo
- Waline
- Giscus
- Disqus
- Artalk

### 更多配置
查看 `src/config/` 目录下的其他配置文件：
- `backgroundWallpaper.ts` - 背景壁纸
- `effectsConfig.ts` - 樱花特效
- `musicConfig.ts` - 音乐播放器
- `licenseConfig.ts` - 文章许可证
- 等等...

完整文档：https://docs-firefly.cuteleaf.cn

## 🔧 常见问题

### Q: Sharp 安装失败？
A: 尝试 `pnpm install --shamefully-hoist` 或参考上面的解决方案

### Q: 构建时间很长？
A: Firefly 使用图像优化，首次构建会慢一些。可以：
- 减少 `src/assets/images/` 中的图片
- 在 `siteConfig.ts` 中禁用 OG 图片生成（已默认禁用）

### Q: 如何删除示例文章？
A: 删除 `src/content/posts/` 中不需要的 `.md` 文件

### Q: 如何自定义样式？
A: 编辑 `src/styles/` 目录下的 CSS 文件
