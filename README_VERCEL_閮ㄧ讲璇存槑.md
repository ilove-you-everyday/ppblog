# PENGWH 部署说明

## 当前版本

- 首页及所有内容页共用响应式导航：页面顶部是普通透明导航，向下滚动后才收缩成毛玻璃悬浮胶囊。桌面端显示文字链接，手机端顶部使用三横线菜单，滚动后在胶囊上直接显示文字导航。
- 已包含文章：`Claude Opus 5.5：重新定义 AI 编程的边界`。
- 文章源文件位于 `content/posts/`，列表 JSON 会由构建脚本同步写入 `public/data/posts.json` 与 `dist/data/posts.json`。构建同时生成 `/articles/`、`/categories/`、`/tags/`、`/about/` 和每篇文章的独立详情路由。
- 已移除 `functions` 通配规则，由 Vercel 自动识别根目录 `api/` 中的函数。

## 文章怎么添加

在 `content/posts/` 新建独立 `.md` 文件，填写标题、唯一 slug、日期、可选分类及 `published: true`，再写 Markdown 正文。参考 `文章怎么写.md`。提交源码后，Vercel 的构建命令会生成 `dist/` 并把文章列表同步到两个 JSON 位置。

## Vercel 配置

项目根目录应直接对应 Git 仓库根目录，包含 `package.json`、`public/`、`api/`、`scripts/`、`content/` 和 `vercel.json`。

- Framework Preset：Other
- Install Command：`npm install`
- Build Command：`npm run build`
- Output Directory：`dist`

`vercel.json` 不应添加 `functions` 通配规则，以避免函数匹配不到 `api/` 中的 Serverless Function。Twikoo 的真实评论可用性仍取决于外部评论服务配置。

## 本地验证

运行 `npm test` 与 `npm run build`。本地构建成功不代表线上域名已更新；部署后如仍看不到文章，应确认 Vercel 连接的是包含最新文件的 Git 分支、Root Directory 正确，并且部署使用本次提交。
