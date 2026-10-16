# hub-publish

把内容写进 backend-hub 并发布到产品站点（GenGrades 等）的 Claude Code 插件。装一次就同时有两样东西：

- **publish 写作技能**：通用写法（事实带出处、不编数字、FAQ 先给结论、写清不适合谁、标题和描述长度、多语言）和发布流程。
- **hub 远程 MCP**（`https://hub.apimodels.app/mcp`）：读产品的写作规则、写草稿、加其他语言版本、上传图片、发布、定时发布、下线。

每个产品自己的规则（比如 GenGrades 的工具页、模型页怎么写）不在插件里，存在 hub 里，写之前由 MCP 读出来。加新产品、改规则都不用发新版插件。

## 安装

1. 在 hub 后台为要发布的产品新建一把服务端密钥：项目 → API Keys → 新建，env 选 **prod**，scopes 填：

   ```
   content:read
   content:write
   content:publish
   files:write
   files:read
   ```

   只想让它写草稿、由人在后台发布，就去掉 `content:publish`。明文只显示一次，先复制好。

2. 在 Claude Code 里：

   ```
   /plugin marketplace add stimQQ/hub-publish
   /plugin install hub-publish@hub-publish
   ```

   安装时会提示填 **Hub API key**，粘贴上一步的密钥。它存在系统的凭据存储里，不写进项目文件。

3. 运行 `/reload-plugins`（或重开 Claude Code），`/mcp` 里能看到 `plugin:hub-publish:hub` 已连接。

之后直接说「写一篇 X 的评测发到站上」「给这页加中文版」就行。用户没说发布时，只存草稿。

## 一个 key 对应一个产品

key 决定写进哪个产品。要给另一个产品发内容，在那个项目里新建 key，在 `/plugin` → hub-publish 里换掉 Hub API key。

## 产品的写作规则

在 hub 后台 → 项目 → 内容 → 「写作规则」里编辑（markdown）。写作技能每次动笔前都会先读它，冲突时以它为准。适合写进去的：每类页面写进哪个内容类型、块的顺序和写法、slug 规则、哪些说法不能用、没测过的东西怎么写、哪些条目不能由 agent 新建。

## 权限和限制

- 只接受项目的服务端密钥；写和发布要 prod 的 key（内容不分环境）。
- 发布门槛、字段校验、站点刷新和后台完全一致：分数低于内容类型的 `min_lint_score` 不能发布。
- MCP 里不能改内容类型结构、归档、回滚、改项目设置，这些在后台做。
- 发布、定时发布、下线都记审计，操作人是这把 key。

## 更新

`/plugin` → Marketplaces → hub-publish → Update marketplace，再 `/reload-plugins`。也可以在 Marketplaces 里打开自动更新。

## 其他客户端

插件只做了 Claude Code 版。别的 MCP 客户端可以手动接：添加一个 Streamable HTTP 服务，地址 `https://hub.apimodels.app/mcp`，请求头 `Authorization: Bearer <key>`，再把 `plugins/hub-publish/skills/publish` 目录放进那个客户端的技能目录。

## 目录

```
.claude-plugin/marketplace.json        插件市场
plugins/hub-publish/
  .claude-plugin/plugin.json           插件信息和 Hub API key 配置项
  .mcp.json                            hub 远程 MCP
  skills/publish/SKILL.md              写作和发布流程
  skills/publish/references/chinese.md 中文版怎么写
```
