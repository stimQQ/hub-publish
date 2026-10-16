# hub-publish

给 GenGrades 写内容的 Claude Code 插件。装一次就同时有两样东西：

- **gengrades-write-guide 写作技能**：通用写法（事实带出处、不编数字、FAQ 先给结论、写清不适合谁、标题和描述长度、多语言）和提交流程。
- **hub 远程 MCP**（`https://hub.apimodels.app/mcp/gengrades`）：读 GenGrades 的写作规则，写草稿、加其他语言版本、上传图片、提交审核。

GenGrades 自己的规则（工具页、模型页怎么写）不在插件里，存在 hub 里，写之前由 MCP 读出来。改规则不用发新版插件。

## 安装

在 Claude Code 里：

```
/plugin marketplace add stimQQ/hub-publish
/plugin install hub-publish@hub-publish
/reload-plugins
```

不用填任何 key。第一次让它干活时（比如说「读一下 GenGrades 的写作规则」），Claude Code 会打开浏览器，跳到 gengrades.com 的授权页：用 Google 账号登录，看清楚是谁在请求、能做什么，点「授权」，再回到 Claude Code 就能用了。授权有效 180 天。

浏览器没有自动打开时，在 `/mcp` 里选 `plugin:hub-publish:hub`，点 Authenticate。

## 写好的内容怎么上线

登录的人都是投稿人：

- 能做的：读已发布的页面和写作规则，写草稿和译本，上传图片，把草稿提交审核。
- 不能做的：发布；修改已上线的页面和别人的草稿；同时有超过 30 条没发布的条目。

提交后，GenGrades 在 hub 后台「内容」里筛「待审核」：通过就点「审核通过并发布」，不通过就「退回」并写原因。作者通过插件能看到退回原因，改完再提交。

## 撤销授权

登录授权拿到的是你账号下的一把 key（名字是「客户端名 · MCP」），可以在 hub 的 `/v1/me/api-keys` 里随时吊销。重新登录会自动作废旧的那把。

## 更新

`/plugin` → Marketplaces → hub-publish → Update marketplace，再 `/reload-plugins`。也可以在 Marketplaces 里打开自动更新。

## 其他客户端和自动化

- 别的 MCP 客户端：添加 Streamable HTTP 服务 `https://hub.apimodels.app/mcp/gengrades`，支持 MCP 授权（OAuth）的客户端会自动走同样的登录；再把 `plugins/hub-publish/skills/gengrades-write-guide` 目录放进那个客户端的技能目录。
- 需要直接发布的自动化：用 hub 后台建的服务端密钥（prod，scopes 带 `content:publish`），请求头 `Authorization: Bearer <key>`。

## 目录

```
.claude-plugin/marketplace.json                       插件市场
plugins/hub-publish/
  .claude-plugin/plugin.json                          插件信息
  .mcp.json                                           hub 远程 MCP（GenGrades）
  skills/gengrades-write-guide/SKILL.md               写作和提交流程
  skills/gengrades-write-guide/references/chinese.md  中文版怎么写
```
