# 人生实验室

个人技术博客，基于 [Firefly](https://github.com/whyoume545/astro-ThemeFirefly) 主题（Astro 7 + Svelte 5 + Tailwind CSS 4），部署在 GitHub Pages，线上地址：[ld.ac.cn](https://ld.ac.cn)。

## 内容结构

```text
src/
├── content/
│   └── posts/                  # 文章（Markdown frontmatter）
├── assets/
│   └── images/posts/<slug>/    # 每篇文章的配图，与 slug 同名目录
├── constants/
│   ├── icons.ts                # 由 scripts/generate-icons.js 自动生成
│   └── lqips.json              # 图片 LQIP 数据，构建时自动生成
└── config/
    ├── sidebarConfig.ts        # 侧边栏分类
    └── siteConfig.ts           # 站点 URL、标题等

.github/workflows/deploy.yml    # push master 后自动构建并发布 Pages
```

分类：AI编程 / 海外订阅 / 网络与自托管 / 编程入门 / 网络安全 / 硬件与系统 / 随笔。

## 写作流程

```sh
# 1. 新建文章（生成带 frontmatter 的模板文件）
pnpm new-post <slug>

# 2. 本地预览
pnpm dev
```

frontmatter 约定：`title / published（精确到秒，决定首页排序）/ description / tags / category / draft / image`。

图片引用使用相对路径 `../../assets/images/posts/<slug>/xxx.png`，图片实体放在对应目录下；构建时自动生成 webp 与 LQIP，`src/constants/lqips.json` 的变更需随文章一起提交。

## 常用命令

所有命令在项目根目录执行（包管理器锁定 pnpm，npm/yarn 会被 `only-allow` 拦截）：

| 命令 | 作用 |
| :-- | :-- |
| `pnpm install` | 安装依赖 |
| `pnpm dev` | 本地开发服务器 `localhost:4321` |
| `pnpm build` | 生产构建到 `./dist/`（含图标生成、LQIP、字体子集化、Pagefind 索引） |
| `pnpm preview` | 本地预览构建产物 |
| `pnpm new-post <slug>` | 新建文章模板 |
| `pnpm icons` | 重新生成 `src/constants/icons.ts` |
| `pnpm lint` / `pnpm format` | Biome 检查 / 格式化 |

## 发布

提交前本地必须跑 `pnpm build` 通过。构建成功后提交推送到 `master`，GitHub Actions 自动部署：

```sh
pnpm build
git add src/content/posts/<slug>.md src/assets/images/posts/<slug>/ src/constants/lqips.json
git commit -m "Add <post title>"
git push origin master
```

`dist/` 不入库。

## 已知注意点

- `astro check` 的语言服务（volar）尚未适配 TypeScript 7，peer 警告不影响构建部署
- `@astrojs/mdx` 与 `@astrojs/markdown-satteri` 之间存在版本耦合，升级时留意 peer 提示
- pako 3、katex 0.19 等依赖使用具名导出，改代码时不要按旧的 default import 写
