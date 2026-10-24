# PENGWH 二次源码与部署配置复检

## 本地源码检查

- `node --check` 已对管理员 API、后台脚本、评论脚本、站点脚本和主页脚本执行。
- `npm test` 通过。
- `npm run build` 通过，构建生成 1 篇已发布文章，并生成独立文章、分类、标签、关于及 404 页面。
- `vercel.json` 不含 `functions: { "api/*.js": ... }`，由 Vercel 自动发现根目录 `api/` 下的函数，避免未匹配函数模式的问题。
- 管理员接口新增同源 Origin 检查；异常编码 Cookie 会被安全拒绝；读取文章前先验证 GitHub 仓库可访问，避免把仓库权限错误误显示成空文章列表。
- 首页 Clawd 闲置动画修复为：连续 2 分钟无点击时仅播放一次，然后恢复静止状态。
- 检查过源码和构建输出，没有把用户提供的 OAuth 字符串写入任何文件、静态资源或压缩包。

## 当前线上配置现状（查询时）

预期生产项目是绑定 `pengblog.de5.net` 的 Vercel 项目 `pengwhblog`。读取到的旧环境变量包含 Twikoo 的 MongoDB 配置和旧管理字段，但未发现本版本所需的 `APP_URL`、`GITHUB_CLIENT_ID`、`GITHUB_CLIENT_SECRET`、`ADMIN_GITHUB_USERS`、`GITHUB_REPO`、`SESSION_SECRET` 这一整套管理员 OAuth 配置。

本轮尝试向该项目写入环境变量时，Vercel 返回 403，说明当前连接没有获得该团队/项目的写入授权。因此没有变量写入成功，也没有触发正式部署。GitHub 连接器同样拒绝了仓库文件写入，源码 ZIP 仍需在权限恢复后同步/部署。

## OAuth 安全提示

用户提供的字符串符合 40 位十六进制格式，较像 GitHub OAuth Client Secret，不像常见的 OAuth Client ID。查询 Vercel 项目时还看到了一个项目名与该字符串相同；如果该字符串确实是有效的 Client Secret，应立即在 GitHub OAuth App 设置中撤销并轮换，且不要在聊天中发送新密钥。新 Client ID 仍需从 GitHub OAuth App 设置页面读取。

## 部署前必须完成

1. 重新授权 Vercel 连接访问拥有 `pengwhblog` 项目的团队/Scope。
2. 在 GitHub OAuth App 中轮换已暴露的 Client Secret，并取得正确的 Client ID。
3. 将 `APP_URL` 设置为最终生产域名（当前项目绑定 `https://pengblog.de5.net`），其 OAuth 回调应为 `https://pengblog.de5.net/api/admin/callback`。
4. 设置 `GITHUB_REPO=ilove-you-everyday/pengblog`、`ADMIN_GITHUB_USERS=ilove-you-everyday` 及敏感的 `SESSION_SECRET`、`GITHUB_CLIENT_SECRET`，再部署并验证 `/blogcontrol/` 和 `/api/admin/me`。

上述线上步骤目前尚未完成；本报告仅说明已经实测通过的本地构建与已观察到的授权阻塞。
