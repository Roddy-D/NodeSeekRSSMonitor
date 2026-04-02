# NodeSeek Keywords Monitor — Cloudflare Workers

基于 Cloudflare Workers + D1 的 NodeSeek 论坛关键词监控 Telegram Bot。

## 功能

- 关键词监控，支持子串匹配和正则匹配
- 支持按版块过滤，多版块逗号分隔（如 `trade,info`）
- 每分钟自动轮询 RSS，新帖匹配后推送 Telegram 通知
- 首次运行静默初始化，不发历史通知
- 防洪上限 + 溢出汇总，避免刷屏
- RSS 连续失败告警
- 发送失败自动重试（30 次指数退避）
- 自动清理过期数据（seen_posts 7 天，notifications 30 天）

## 部署步骤

1. **Cloudflare Dashboard → Workers & Pages → Create → 从 Hello World! 开始**
2. **进入 Worker Settings → Variables and Secrets**，添加以下环境变量：

   | 名称                   | 值                    | 说明                                         |
   | ---------------------- | --------------------- | -------------------------------------------- |
   | `TELEGRAM_BOT_TOKEN` | 你的 Bot Token        | 从[@BotFather](https://t.me/BotFather) 获取     |
   | `ALLOWED_USER_ID`    | 你的 Telegram 用户 ID | 从[@userinfobot](https://t.me/userinfobot) 获取 |
   | `WEBHOOK_SECRET`     | 随便编一个随机字符串  | 可选，建议设置                               |
3. **Worker Settings → Bindings → D1 Database**：

   - 先在 D1 页面创建一个数据库，名字随意，之后去 Worker 内绑定
   - Variable name 填：`DB`
   - 选择刚创建的数据库
4. **粘贴 `worker.js` 代码，部署**
5. **Worker Settings → Triggers → Cron Triggers**：

   - 添加 `* * * * *`（每分钟执行一次）
6. **访问 `https://你的worker域名/setup`** 完成 Telegram Webhook 注册和数据库初始化（可能需要等待一段时间）
7. **给 Bot 发 `/start` 测试**

## Bot 命令

| 命令                               | 说明                       |
| ---------------------------------- | -------------------------- |
| `/add <关键词> [--regex] [版块]` | 添加监控关键词             |
| `/remove <序号或关键词>`         | 删除关键词                 |
| `/pause <序号或关键词>`          | 暂停关键词                 |
| `/resume <序号或关键词>`         | 恢复关键词                 |
| `/list`                          | 查看监控列表               |
| `/history [数量]`                | 查看推送记录（默认 10 条） |
| `/categories`                    | 查看可用版块               |
| `/status`                        | 查看运行状态               |

## 可用版块

| 代码       | 名称 | 代码            | 名称 |
| ---------- | ---- | --------------- | ---- |
| `daily`  | 日常 | `tech`        | 技术 |
| `info`   | 情报 | `review`      | 测评 |
| `trade`  | 交易 | `carpool`     | 拼车 |
| `dev`    | Dev  | `photo-share` | 贴图 |
| `expose` | 曝光 | `sandbox`     | 沙盒 |

## 使用示例

```
/add 搬瓦工                        # 全部版块监控 搬瓦工
/add 搬瓦工 trade                  # 仅交易版
/add 搬瓦工 trade,info             # 交易 + 情报
/add 补货 --regex info,trade     # 正则匹配，情报 + 交易
/add 搬瓦工.*(CN2|GIA) --regex     # 正则匹配全部版块
/remove 2                        # 删除列表中第 2 条
/remove 搬瓦工                     # 按关键词删除
/pause 1                         # 暂停第 1 条
/resume 1                        # 恢复第 1 条
```
