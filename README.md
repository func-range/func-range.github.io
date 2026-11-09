# FuncRange 官网

创意区间工作室的静态站点，部署在 GitHub Pages（`https://func-range.github.io`）。

```
index.html      首页（云存档入口、社区、关于我们）
cloud.html      云存档控制台（登录账户、查看游戏与云存档、下载存档）
community.html  社区（一键加入 QQ 群、前往 B 站主页）
help.html       云存档帮助（登录、下载、常见问题）
image/          站点图片
```

站点顶部统一提供「前往云存档 / 社区 / 关于我们 / 帮助」四个入口。首页首屏直接引导进入云存档；社区是独立页面，提供 QQ 群和 B 站入口；关于我们位于首页对应区块。

## 云存档控制台

`cloud.html` 直接调用 [Der Online Server](https://github.com/newDerFolder/der-online-server) 的 HTTP API，不依赖任何构建工具或第三方库：

- 登录 / 注册账户（令牌默认保存 30 天，可取消“保持登录”）；注册时显示名称默认使用用户名
- 账户设置：修改登录用户名（需要当前密码）和昵称，不会让其它设备退出登录
- 查看账户下玩过的所有游戏、存档数量、占用空间与最近更新时间
- 显示等级、经验进度、今日云存档用量和存储配额（等级越高，每日次数和总空间越大）
- 每个游戏列出存档卡片，显示存档名、大小和更新时间，一键下载
- 下载原始字节，文件名取槽位名，例如 `AutoSlot.tres`

控制台会读取登录、`/me` 和下载响应里的 `progress` 数据或 `X-Account-*`、`X-Daily-Save-*` 响应头来更新等级和用量，并把最近一次结果缓存在浏览器中。服务端返回的 `game_not_allowed`、`daily_limit_reached`、`storage_quota_exceeded` 会显示成对应的中文提示。

页面只保留面向玩家的内容，不展示版本号、SHA-256、跨域、部署等技术细节。服务地址在登录页底部填写，保存在浏览器 `localStorage` 中。部署和跨域配置见下文。

## 部署前必须配置服务器

浏览器是跨域访问服务器的，所以服务器 `config.json` 必须允许本站来源：

```json
{
  "cors_allowed_origins": ["https://func-range.github.io"]
}
```

本地调试时加上本地来源，或临时使用 `"*"`：

```json
{ "cors_allowed_origins": ["*"] }
```

**HTTPS 要求**：GitHub Pages 强制 HTTPS。如果服务器地址仍是 `http://…`，浏览器会以“混合内容”拦截所有请求。正式使用前请给服务器启用 HTTPS：

- 推荐用 Nginx / Caddy 反向代理到 `http://127.0.0.1:18080`，并把 `trust_proxy_headers` 设为 `true`；或
- 在 `config.json` 中配置 `tls_cert_file` / `tls_key_file`，直接由 exe 提供 HTTPS。

页面检测到「HTTPS 页面 + HTTP 服务器」时会显示明确提示，不会静默失败。

## 本地调试

```bash
# 1) 启动服务器（示例端口 18080）
der-online-server.exe

# 2) 在站点目录启动静态服务
python -m http.server 8080
```

打开 `http://127.0.0.1:8080/cloud.html`，服务器地址填 `http://127.0.0.1:18080`，并确保 `cors_allowed_origins` 包含 `http://127.0.0.1:8080`（或 `"*"`）。

## 安全说明

- 登录令牌存放在 `localStorage`（勾选“保持登录”）或 `sessionStorage`（不勾选）。在公共电脑上不要勾选保持登录。
- “退出登录”会调用服务器登出接口使当前令牌立即失效，并清除本地令牌。
- 页面不会把密码写入任何本地存储。
- 服务器不解析 `.tres`，下载内容与上传内容逐字节一致（实测 SHA-256 一致）。

## 部署

推送到 `main` 分支后，在仓库 Settings → Pages 中选择从 `main` 分支根目录发布即可。修改 `cors_allowed_origins` 后需要重启服务器进程生效。
