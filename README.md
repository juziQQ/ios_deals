# ios_deals

面向新版青龙的 iOS 工具类优惠监控项目。

项目包含两条独立任务链：

- **iOS 工具线索摘要**：抓取 Reddit / RSS / Apple 榜单，AI 预筛后做 Apple 多区价格核验，推送值得关注的工具类优惠线索
- **iOS Watchlist 定向盯价**：按自定义列表持续监控目标 App 的价格变化，支持限免 / 降价 / 达到目标价提醒

---

## 功能

- iOS 工具类优惠线索抓取
- Apple 多区价格核验
- AI 预筛
- 真实限免 / 真实降价判断
- Watchlist 定向盯价
- 青龙 `notify.py` 通知

---

## 青龙要求

仅支持使用 `/ql/data` 目录结构的新版青龙。

本项目按以下路径设计：

```text
/ql/data/scripts/ios_deals
/ql/data/db/ios_deals.db
/ql/data/scripts/notify.py
```

---

## 任务命令

订阅拉取后，任务命令请保持为：

```bash
task ios_deals/ios_digest.py
task ios_deals/ios_watchlist.py
```

**不要改成 `python3 /ql/data/scripts/...`。**  
本项目按青龙任务环境运行设计，通知加载依赖青龙 `notify.py`。

---

## 环境变量

只需要以下 3 个可选变量：

```bash
QWEN_API_KEY=
DEEPSEEK_API_KEY=
GEMINI_API_KEY=
```

### AI 说明

- 主模型：`qwen3.5-flash`
- 第一备份：`deepseek-chat`
- 第二备份：`gemini-2.5-flash-lite`

固定调用顺序：

```text
Qwen -> DeepSeek -> Gemini
```

行为说明：

- 有 `QWEN_API_KEY`：优先使用 Qwen
- Qwen 不可用时：自动切到 DeepSeek
- DeepSeek 不可用时：自动切到 Gemini
- 没填的模型会自动跳过
- 所有模型都失败时：自动退回规则预筛

---

## 项目文件

```text
ios_deals/
├── .gitignore
├── README.md
├── ai_filter.py
├── common.py
├── ios_digest.py
├── ios_watchlist.py
├── feeds.json
└── watchlist_ids.json
```

### 运行后自动生成

```text
/ql/data/scripts/ios_deals/ai_cache.json
/ql/data/db/ios_deals.db
```

这些运行文件**不要提交到 GitHub**。

---

## 两个入口脚本

### 1. iOS 工具线索摘要

入口：

```bash
task ios_deals/ios_digest.py
```

功能：

- 抓取 Reddit / RSS / Apple 榜单候选
- 规则预过滤
- AI 预筛
- Apple 多区价格核验
- 推送当天值得关注的工具类优惠线索

### 2. iOS Watchlist 定向盯价

入口：

```bash
task ios_deals/ios_watchlist.py
```

功能：

- 读取 `watchlist_ids.json`
- 定向查询目标 App 当前价格
- 记录价格历史
- 监控限免 / 降价 / 达到目标价
- 推送价格变化提醒

---

## 配置文件

### 1. feeds.json

用于配置抓取源。

默认包含：

- Reddit AppHookup 线索
- appstore-discounts RSS
- Apple Top Paid 榜单

### 2. watchlist_ids.json

用于配置定向监控 App 列表。

示例：

```json
{
  "apps": [
    {
      "id": 904237743,
      "name": "Things 3",
      "countries": ["us", "cn", "tr"],
      "enabled": true,
      "target_price": 0,
      "notify_on_any_drop": true,
      "notify_on_free": true,
      "tags": ["效率", "买断"]
    }
  ]
}
```

字段说明：

- `id`：App Store `app_id`
- `name`：备注名
- `countries`：监控区服
- `enabled`：是否启用
- `target_price`：目标价格，达到后提醒
- `notify_on_any_drop`：任意降价是否提醒
- `notify_on_free`：限免是否提醒
- `tags`：自定义标签

---

## 安装方式

### 方式一：青龙订阅（推荐）

建议按“公开仓库订阅”方式安装。

#### 订阅建议填写

- 名称：`ios_deals`
- 类型：`公开仓库`
- 链接：`https://github.com/juziQQ/ios_deals.git`
- 分支：`main`
- 文件后缀：`py json`
- 白名单：`^(ios_digest|ios_watchlist|common|ai_filter)\.py$|^(feeds|watchlist_ids)\.json$`

订阅后保持任务命令为：

```bash
task ios_deals/ios_digest.py
task ios_deals/ios_watchlist.py
```

### 方式二：手动放入脚本目录

把仓库文件放到：

```text
/ql/data/scripts/ios_deals
```

然后在青龙中手动创建两条任务：

```bash
task ios_deals/ios_digest.py
task ios_deals/ios_watchlist.py
```

---

## 推荐任务频率

### iOS 工具线索摘要

建议每天 1～3 次，例如：

```cron
0 9,15,21 * * *
```

### iOS Watchlist 定向盯价

建议每 4 小时一次，例如：

```cron
0 */4 * * *
```

---

## 日志说明

日志分为几个阶段：

- 抓取
- 规则预过滤
- AI 预筛
- Apple 核验
- 最终推送

AI 日志采用摘要式输出，例如：

```text
[AI] 链路：qwen(qwen3.5-flash) -> deepseek(deepseek-chat)
[AI] 跳过：gemini(gemini-2.5-flash-lite)（未配置）
[AI] 使用 qwen 成功（3717 ms）
```

---

## 数据说明

### Watchlist 配置文件

```text
/ql/data/scripts/ios_deals/watchlist_ids.json
```

### AI 缓存

```text
/ql/data/scripts/ios_deals/ai_cache.json
```

### 价格历史 / 提醒去重数据库

```text
/ql/data/db/ios_deals.db
```

---

## 引入 / 致谢

本项目为面向青龙环境的自定义 Python 项目，使用或参考了以下公开资源与接口：

- **青龙**
  - 用于订阅管理、定时任务、环境变量与通知体系
- **青龙 `notify.py`**
  - 作为统一通知入口
- **Apple iTunes Lookup API**
  - 用于 App 本体价格核验
- **Apple RSS / Marketing Tools**
  - 用于获取 Top Paid 榜单线索
- **Reddit `/r/AppHookup`**
  - 用于获取优惠帖子线索
- **appstore-discounts RSS**
  - 用于补充优惠 RSS 数据源
- **阿里百炼 Qwen**
  - 用于主 AI 预筛
- **DeepSeek API**
  - 用于第一备份 AI 预筛
- **Gemini API**
  - 用于第二备份 AI 预筛

---

## 免责声明

本项目仅用于个人学习、研究与自动化信息聚合。

请使用者自行遵守：

- GitHub 仓库与数据源的使用条款
- Apple 接口相关条款
- Reddit 平台规则
- 各 AI API 服务商计费与调用规范

本项目不提供任何商业化保证，不对因接口变化、限流、风控、数据错误或价格变化导致的问题承担责任。

---

## 适合人群

适合：

- 在青龙上长期跑脚本
- 关注 iOS 工具类 App 优惠
- 需要多区价格核验
- 希望把 AI 预筛与真实价格验证结合起来

不适合：

- 想监控所有内购 SKU 变化
- 想监控游戏类优惠
- 想做通用 App 全品类推荐
