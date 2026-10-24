# PENGWH 管理后台部署配置

后台地址：`https://pengblog.de5.net/blogcontrol/`（以该域名仍为正式站点为前提；OAuth 的 APP_URL 与 GitHub 回调必须使用同一主机）。主页导航菜单也加入了“管理入口”。后台使用 GitHub OAuth；不在白名单内的账号无法进入。会话使用 AES-256-GCM 加密并保存在 `HttpOnly; Secure; SameSite=Lax` Cookie 中，服务端不把 GitHub token 暴露给浏览器 JavaScript。

## 1. 创建 GitHub OAuth App

在 GitHub 登录后打开 **Settings → Developer settings → OAuth Apps → New OAuth App**：

当前 Vercel 项目中确认绑定的站点域名为 `pengblog.de5.net`。部署目标如果不是这个域名，先确认最终域名再创建 OAuth App。

- Application name：`PENGWH Admin`
- Homepage URL：`https://pengblog.de5.net`
- Authorization callback URL：`https://pengblog.de5.net/api/admin/callback`

创建后记下 **Client ID**，生成 **Client secret**。域名必须和部署后实际使用的主域名一致。Client secret 只应放入 Vercel 的 Sensitive 环境变量，不能提交进 Git 或写入前端。若某个密钥曾出现在聊天文本或公开 URL/项目名称中，应先在 GitHub OAuth App 中撤销并生成新密钥，再使用新值。

## 2. 在 Vercel 添加环境变量

打开项目 **Settings → Environment Variables**，至少添加以下变量，勾选 Production（建议同时勾选 Preview 时使用单独 OAuth App）：

| 变量名 | 值 |
|---|---|
| `APP_URL` | `https://pengblog.de5.net`（如果更换正式域名，必须同步更新 GitHub OAuth 回调） |
| `GITHUB_CLIENT_ID` | OAuth App 的 Client ID |
| `GITHUB_CLIENT_SECRET` | OAuth App 的 Client secret，标记为 Sensitive/Secret |
| `ADMIN_GITHUB_USERS` | 允许登录的 GitHub 用户名，多个用英文逗号分隔，例如 `ilove-you-everyday` |
| `SESSION_SECRET` | 至少 32 个字符的随机长密钥，标记为 Sensitive/Secret |
| `GITHUB_REPO` | 保存此网站源码的仓库，格式为 `所有者/仓库名`，例如 `ilove-you-everyday/pengblog` |

`SESSION_SECRET` 可以在终端运行 `openssl rand -base64 48` 生成。不要把生成结果发给任何人。`GITHUB_REPO` 必须是该 OAuth 登录账号可写入的仓库；GitHub OAuth App 需授予 `repo` 权限，后台仅对 `content/posts/*.md` 执行内容读取和写入。

## 3. 重新部署并验证

保存环境变量后，在 Vercel 执行一次 **Redeploy**。然后访问 `/blogcontrol/`，点击“使用 GitHub 登录”。登录账号必须出现在 `ADMIN_GITHUB_USERS` 白名单中。成功后可新建、编辑或删除 Markdown 文章；发布/修改/删除都会直接创建 GitHub commit，网站会在 Vercel 下一次成功构建后更新。后台还包含“评论管理”入口，会嵌入现有 Twikoo 管理界面。后台文章列表读取仓库 `content/posts/`，因此不要把 `GITHUB_REPO` 指向另一个仓库。

## 4. 评论审核、邮箱与 IP 信息

登录 GitHub 管理后台后，点击“评论管理”。此页面加载本项目已有的 Twikoo 服务，并使用 Twikoo 自身的管理面板完成评论审核、隐藏/恢复和删除。为了避免仅凭 GitHub 登录就直接暴露评论数据库里的敏感字段，Twikoo 管理面板还会要求其自己的管理员验证；这不是 GitHub 密钥，也不需要写进前端代码。

邮箱和 IP 只有在 Twikoo 管理权限验证通过、且评论服务实际保存/取得了对应字段时才可能显示。旧评论没有记录的字段无法凭空补回；不同部署代理也可能影响服务端取得真实客户端 IP 的能力。请不要把评论者邮箱或 IP 公开、截图传播或用于与评论审核无关的目的。前台已增加隐私提示，说明评论邮箱和请求信息可能被评论服务处理。

如果 Twikoo 管理面板尚未设置管理员凭据，请按 [Twikoo 官方常见问题](https://twikoo.js.org/faq) 的管理面板说明完成初始化/重设；不要将 Twikoo 管理密码、GitHub Client Secret 或 `SESSION_SECRET` 发给别人。

## 本次复检说明

源码复检增加了管理员写入接口的同源 Origin 校验、异常 Cookie 解码防护，并让文章列表先验证目标 GitHub 仓库是否真实可访问，避免把仓库权限或路径错误误报成“没有文章”。本地 `npm test` 与 `npm run build` 通过。OAuth 真实登录仍需有效的 Client ID/Secret、Vercel 环境变量写入权限和一次线上部署验证。

## 安全说明与限制

- OAuth 回调校验随机 `state`，会话 Cookie 使用 AES-GCM 加密、8 小时过期、HttpOnly 和 SameSite=Lax。
- 不在白名单的用户即使完成 GitHub 登录也无法使用后台。
- 此后台不会提供公开注册、密码登录或在前端保存 token 的功能。
- GitHub OAuth 的 `repo` scope 较宽，因此建议专门使用一个仅有此博客仓库写权限的 GitHub 账号，并把它加入白名单。
- 文章操作会通过 GitHub Contents API 生成 commit，正式网站是否立即更新取决于 Vercel 构建成功与否。
- 评论管理使用 Twikoo 官方面板及其独立管理员验证；GitHub 登录不会绕过 Twikoo 自身的隐私保护。
