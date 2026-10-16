---
name: gengrades-write-guide
description: 把内容写进 backend-hub 并发布到产品站点（GenGrades 等用 hub 内容模块的网站）。新建或更新工具页、模型页、对比页、教程和资讯文章，给条目加中文等其他语言版本，上传封面和截图，按这个产品存在 hub 里的写作规则和 GEO 评分写，达到发布门槛后发布、定时发布或下线。用户说「写一篇 X 的评测发到站上」「把这篇同步进数据库」「给这页加中文版」「更新某页的价格」「上传封面」「把某篇下线」时使用。需要插件配置里填好那个产品的 hub key。
---

# 写内容并发布到 hub

hub 是多个产品共用的后端，每个产品（GenGrades 等）是 hub 里的一个项目，网站从 hub 的内容接口读条目渲染页面。
这个 skill 用插件自带的 MCP（服务名 `hub`，工具全名形如 `mcp__plugin_hub-publish_hub__content_guide`）读规则、写草稿、
发布。插件里填的 key 决定写进哪个产品、能不能发布。

本文件是所有产品通用的写法。**每个产品自己的规则在 hub 里（`content_guide` 返回的 `writing_guide`），两者冲突时以
writing_guide 为准。**

## 0. 动笔之前，每次都做

1. 调 `content_guide`（不带参数）。看：`project`（产品、支持的语言）、`writing_guide`、`collections`（每类的必备块
   `required_blocks`、发布门槛 `min_lint_score`、页面地址模板）、`lint.rules`（评分规则）、`uploads`（能传什么图）、
   `credential.scopes`（这把 key 能不能发布）。
2. 按 writing_guide 决定写进哪个 collection。它没说清、或者用户的要求和它对不上时，先问用户，不要自己猜。
3. 调 `content_guide(collection="…")` 拿展开后的 schema、答案块字段（`blocks`）和关联字段（`relations`）。
4. 查重：`list_entries(collection=…, q=名字)`。已经有这个主题的条目就更新它，不要新建第二条。
   两个页面打同一个主词会互相抢排名。

## 1. 事实：不编，每条带出处

- 关于别人产品的每条事实（价格、参数、上下文长度、发布日期、跑分、政策）都要有 `source_url` 和 `checked_at`。
  `checked_at` 是你实际打开来源核对的日期，不能晚于今天。没打开过原文的说法不写。
- 不编任何数字、用户量、评分、证言、日期。查不到就不写这一项；lint 扣分也不编。
- 我们没亲自测过，就不写分数、不写测试日期、不写「我们测试发现」；schema 里 `verified` 一类字段如实填 false。
  别人公开的测试要写明是谁、在哪测的，来源放进 `sources`。官方数字和第三方测试分开写。
- 引用别人的原话最多一句，其余用自己的话概括。
- 作者（`author`）只关联真实存在的作者条目，没有就不填，不编人。

## 2. 写法：让 AI 搜索敢引用

- **summary**：一两句直接定义，主词做主语：它是什么、给谁用、解决什么问题。不用营销开头
  （Welcome、Discover、Unleash、「欢迎来到」「全新」）。
- **每段第一句能单独摘出来当答案**。AI 引用时不会替你补上下文。
- **写清不适合谁、做不好什么**（`limits`），至少一条，不软化。这是最稀缺、也最容易被引用的信息。
- **FAQ** 至少 3 条，建议 5–6 条：问题用用户真实的问法，以 `?` 或 `？` 结尾；答案第一句直接给结论，再说为什么、
  什么情况下不适用。不用 it / this / 它 / 这 开头。
- **对比**：点名替代品，承认对方擅长的地方，按人群给结论。
- **sources** 至少 3 条，都是你实际打开过的页面。
- 正文（markdown 字段）用 H2 分节，不写 H1（页面已经有）；不用引用块（`>`）；不贴外部图片链接。

## 3. 标题、描述和多语言

- `seo.title` 30–62 显示列，`seo.description` 110–160 显示列。中文、日文、韩文一个字算 2 列。标题里要有主词。
  量的方法：

  ```python
  import unicodedata
  width = lambda s: sum(2 if unicodedata.east_asian_width(c) in "WF" else 1 for c in s)
  ```

- 标题不承诺页面上没有的东西（没测过就不写 tested、实测、打分）。
- 非默认语言的版本不是逐字翻译：主词按当地人的搜法定（中文查百度联想，确认这个词搜出来的是我们这类内容）。
  中文排版见 `references/chinese.md`。

## 4. 保存、看 lint、修

1. `create_entry(collection, data, seo, locale?, slug?)` 建草稿。不传 slug 就按标题生成；writing_guide 对 slug 有要求时照它。
2. 返回里看 `lint`：`score`、`threshold`、`can_publish`、`missing`（缺什么）。按 `missing` 补**真实内容**；补不了的
   如实告诉用户差什么，不要为凑分编内容。
3. 报 `content.invalid_data` 时，`params` 里有逐个字段的错误路径，照着改。
4. 关联字段（`relations` 里列出的）填目标条目的 **id**，用 `list_entries` 查，不是填名字。目标不存在时，
   按 writing_guide 判断能不能新建（比如厂商条目），拿不准就问。
5. 改已有条目：只动几个字段用 `update_entry(entry_id, fields={…})`（值为 null 删掉这个字段），整份重写用 `data`。
   先 `get_entry` 读当前内容和 `revision`，保存时带 `expected_revision`，别人刚改过会返回 `content.conflict`，
   这时重新读再改，不要覆盖。
6. 已发布的条目改完立即生效，站点会自动刷新。

## 5. 其他语言版本

1. 默认语言版本定稿后，`translate_entry(entry_id, locale, data, seo)` 一步建好这个语言的草稿。
2. 数据结构和默认语言版本一致；事实、数字、来源、日期、关联 id、文件 id 都不变，只换文字。不多也不少。
3. 这个语言已经有版本时会返回 `content.translation_exists`：用 `list_entries(locale=…)` 找到它，用 `update_entry` 改。
4. 每个语言单独过 lint、单独发布。

## 6. 图片（封面、截图、分享图）

- 只用有权使用的图：我们自己做的、自己截的、用户给的。不用别人官网、新闻稿、搜索结果里的图和品牌 logo，
  除非用户确认可以用。
- 只放进 schema 里有的文件字段（`widget: "file"`）或 `seo.og_file_id`；schema 里没有封面字段就不要硬塞，告诉用户。
- 上传流程：
  1. `create_upload(purpose, mime, size_bytes, filename)`，purpose 和允许的类型、大小看 `content_guide` 的 `uploads`。
  2. 用返回的地址上传原文件，带上返回的所有 headers：
     `curl -sS -X PUT --upload-file ./cover.png -H "Content-Type: image/png" "<upload.url>"`
  3. `complete_upload(file_id)`。
  4. `get_file(file_id)` 直到 `status` 是 `ready`（处理是异步的，一般几秒），再把 `file_id` 填进条目。

## 7. 发布

- 发布 = 线上可见、站点刷新、推送给搜索引擎。**用户没说发布，就只存草稿**，告诉用户条目 id 和发布后的地址。
- `publish_entry(entry_id)`：分数低于门槛会返回 `content.lint_below_threshold`，`params.missing` 是缺的项。
- `schedule_entry(entry_id, publish_at)`：带时区的未来时间，如 `2026-11-01T09:00:00+08:00`。
- `unpublish_entry(entry_id)`：回到草稿，内容和修订都保留。归档、删除、改内容类型结构不在 MCP 里，要在 hub 后台做。
- 返回 `auth.scope_missing` 且缺 `content:publish`：这把 key 只能写草稿，请用户在 hub 后台发布。

## 8. 汇报

- 说清：条目 id、每个语言的状态（草稿 / 已发布 / 定时）、页面地址、lint 分数和门槛、还缺什么、用了哪些来源。
- 只有 `publish_entry` 返回 `status: "published"` 才说「已发布」。
- 网站有缓存，发布后通常几分钟内可见。不要反复请求线上网站去确认（可能触发防火墙），需要的话只看一次。

## 错误码

| code | 意思 | 怎么办 |
|---|---|---|
| 401 `auth.invalid_credentials` | key 没填、填错或已作废 | 请用户在 `/plugin` 里重新配置 hub-publish 的 key |
| `content.server_key_required` | 用的不是项目的服务端密钥 | 请用户在 hub 后台为这个项目新建服务端密钥 |
| `content.prod_key_required` | 用的是 sandbox key | 写内容要 env 为 prod 的 key |
| `auth.scope_missing` | key 缺权限（`params.scopes`） | 告诉用户缺哪项 |
| `content.invalid_data` | 不符合 schema 或规则 | 按 `params` 里的字段路径改 |
| `content.slug_taken` | slug 已被占用 | 先查是不是同一个主题，是就更新那条 |
| `content.conflict` | 别人刚改过 | 重新 `get_entry` 再改 |
| `content.translation_exists` | 这个语言已有版本 | 改那一条 |
| `content.invalid_transition` | 当前状态不允许这个操作（比如对已发布的条目再发布） | 先 `get_entry` 看 `status` |
| `content.lint_below_threshold` | 分数不够 | 按 `params.missing` 补真实内容 |
| `content.invalid_publish_at` | 定时时间不对 | 用带时区的未来时间 |
| `files.*` | 上传不符合用途的类型、大小 | 看 `content_guide` 的 `uploads` |
