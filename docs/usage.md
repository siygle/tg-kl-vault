## 使用

命令：

```
/sub [url] 订阅（url 为可选）
/unsub [url] 取消订阅（url 为可选）
/list 查看当前订阅
/set 设置订阅
/check 立刻抓取所有订阅并推播新文章
/feedcheck 检查订阅清单里的 feed 是否还有效
/setfeedtag [sub id] [tag1] [tag2] 设置订阅标签（最多设置三个Tag，以空格分隔）
/setinterval [interval] [sub id] 设置订阅刷新频率（可设置多个sub id，以空格分隔）
/activeall 开启所有订阅
/pauseall 暂停所有订阅
/import 导入 OPML 文件
/export 导出 OPML 文件
/unsuball 取消所有订阅
/help 帮助
```

**`/check` 和 `/feedcheck` 不一样**：`/check` 是「立刻去抓，有新文章就推给我」，`/feedcheck` 是「这些订阅还活着吗」——只探测、只回报，不会写入任何东西，也不会推播文章。

`/feedcheck` 会并发探测每个订阅源，把有问题的排在最前面：

- ❌ 连不上／HTTP 错误（404、403、逾时……）
- 🧩 抓得到但不是有效的 RSS/Atom（例如网站改版后返回 HTML）
- 📭 能解析但一篇文章都没有
- 🪦 还能抓，但最新一篇超过 180 天——可能已停更
- ⏸ 已被暂停抓取（排程器会略过这类源，`/list` 里也会标出来）

每一列同时并列**排程器记录的历史**（连续失败次数、上次错误、上次成功抓取时间）与**当下探测结果**。两者不一致就是最有用的讯息：「记录显示坏掉、现在探测正常」是暂时性故障，「记录正常、现在 404」则是刚坏。

### 为什么 `/check` 会推播很久以前的文章

判断「新文章」靠的是 `contents` 去重帐本（feed 网址 + GUID 的雜凑）。帐本一旦破洞，旧文章就会被当成新的：feed 换了 GUID（换部落格引擎、http→https、网址结尾斜线变动）、帐本被 `retention_days` 清掉、或来源被暂停很久后才重新抓取，都会触发。

`[fetch] max_item_age_days`（预设 30 天）是第二道闸门：发布日期比这个久的文章一律不推播，但仍会写进帐本标记为已读，所以只会被判断一次。没有日期的文章无从判断，维持照送。设 0 可关闭这道闸门。

### 书签（Bookmarks）

以聊天室为单位的书签库。每则推播讯息下方会出现 🔖 按钮，一键收藏；也可用指令收藏任意网址。收藏后会立即回覆，背景 worker 会自动补上分类标签（预设走 Gemini 免费层，无 API key 时退回本地关键字启发式）后再编辑讯息。

```
/bm [url] 收藏网址（不带参数时，回覆一则含连结的讯息即可收藏该连结）
/bookmarks 分页浏览书签（每页 5 笔，可进详细页编辑／删除／改标签）
/bmsearch [关键字] 关键字搜寻（标题／网址／备注，前 10 笔）
/bmnote [id] [文字] 为书签加备注（也可从详细页的 📝 按钮进入）
/bmtag [id] [slug…] 手动设定标签（标签为固定英文分类，空格分隔）
/bmdel [id] 删除书签（详细页的 🗑 按钮有确认步骤）
```

- **标签为固定英文 slug 分类表**，AI 只能从表中挑选；手动标签可在详细页的「🏷 标签」网格中点选切换。
- **归属为每聊天室**：群组成员共用同一个书签库，任何成员可读取／新增；删除／改标签需为建立者或群组管理员。
- 搜寻使用 SQLite `LIKE`：**仅 ASCII 不分大小写**（中日韩字元区分大小写），且 `%`、`_` 会被当成字面字元。
- 网址正规化会移除常见追踪参数（`utm_*`、`fbclid` 等，但**保留** `ref` 与 `si`），不会移除 `www.` 或结尾斜线 — 因此 `www.x.com/a` 与 `x.com/a` 会是两笔不同书签。
- 在 `/settings → 🔖 书签` 可开关每则推播的 🔖 按钮、开关 AI 自动标签，以及汇出书签（Markdown，依标签分组）。
- **AI 后端可选 MCP 远端 agent**：在 `[bookmark.ai]` 设 `provider = "mcp"` 并填好 `[bookmark.ai.mcp]`（[pi-mcp-bridge](https://github.com/siygle/pi-mcp-bridge) 端点），标签就改由你自己的 agent 产生；连不上时自动退回本地启发式。
- **文章摘要**：只要设定了 MCP 端点，每则推播就会多一个 **📝** 按钮。点下去会请远端 agent 去抓该文章内容并汇整，稍候以回覆讯息带出摘要（非同步，长任务也能等）。此按钮同样可在 `/settings → 🔖 书签` 内开关。

### Channel 订阅使用方法

1. 将 Bot 添加为 Channel 管理员
2. 发送相关命令给 Bot

Channel 订阅支持的命令：

```
/sub @ChannelID [url] 订阅
/unsub @ChannelID [url] 取消订阅
/list @ChannelID 查看当前订阅
/check @ChannelID 检查当前订阅
/unsuball @ChannelID 取消所有订阅
/activeall @ChannelID 开启所有订阅
/setfeedtag @ChannelID [sub id] [tag1] [tag2]  设置订阅标签（最多设置三个Tag，以空格分隔）
/import 导入 OPML 文件
/export @ChannelID 导出 OPML 文件
/pauseall @ChannelID 暂停所有订阅
```

**ChannelID 只有设置为 Public Channel 才有。如果是 Private Channel，可以暂时设置为 Public，订阅完成后改为 Private，不影响 Bot 推送消息。**

例如要给 t.me/debug 频道订阅 [阮一峰的网络日志](http://www.ruanyifeng.com/blog/atom.xml) RSS 更新：

1. 将 Bot 添加到 debug 频道管理员列表中
2. 给 Bot 发送 `/sub @debug http://www.ruanyifeng.com/blog/atom.xml` 命令
