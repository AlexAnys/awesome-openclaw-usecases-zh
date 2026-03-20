# 膳食规划与食物不耐受追踪

发现番茄让你长湿疹、乳制品让你胀气，靠的不是直觉——靠的是数据。但手动记录每顿饭的食材、查营养成分、交叉比对症状日志，几周下来没人能坚持。

这个用例将 OpenClaw 变成一个完整的膳食管理系统：通过 Mealie MCP 管理食谱库和膳食计划，通过 USDA FoodData Central API 自动查询营养成分，自动生成购物清单，并持续追踪食物与症状的关联——最终帮你定位触发不适的具体食物。

## 与 health-symptom-tracker 的区别

[health-symptom-tracker](./health-symptom-tracker.md) 是一个轻量级的饮食-症状日志，核心是 Telegram 记录 + 周度模式分析。本用例在此基础上增加了三个维度：

| | health-symptom-tracker | 本用例 |
|---|---|---|
| **食谱管理** | 无 | Mealie MCP 食谱库（45 个工具） |
| **营养数据** | 无 | USDA API / 中国食物成分表 |
| **购物清单** | 无 | 自动生成、去重、分类 |
| **排除饮食方案** | 无 | 结构化的消除-重引入协议 |
| **记录方式** | 自由文本 | 结构化食材 + 营养成分 |

简单说：health-symptom-tracker 是"记录本"，本用例是"营养师 + 厨房助手 + 侦探"。

## 痛点

- **记录断裂**：手动输入食材、查营养成分、记症状——三个独立动作，任何一个断掉就失去关联数据。"我坚持了两周的食物日记，然后出差三天就再也没捡起来。"
- **触发因素隐藏太深**：茄科蔬菜（番茄、辣椒、茄子）引起的慢性炎症反应可能延迟 24-72 小时，人脑很难跨天关联。一位 HN 用户追踪了两周数据后，AI 发现番茄是其湿疹的触发因素——"我之前完全没听说过'茄科蔬菜'这个概念。"
- **排除饮食执行难**：消除饮食法要求 2-4 周完全排除某类食物，然后逐一重引入。没有系统帮你规划替代食谱、确保营养不缺失、提醒重引入时间点，很难完整执行。
- **购物与计划脱节**：手动从食谱提取食材、合并重复项、检查库存——每周重复一次，耗时且容易遗漏。

## 功能介绍

- **食谱库管理**：通过 Mealie MCP 导入、创建、搜索食谱，按标签（无茄科、低 FODMAP、无乳糖）组织
- **智能膳食规划**：根据营养目标、已知不耐受食物、季节食材生成周计划
- **自动营养查询**：每道菜的食材自动匹配 USDA FoodData Central（或中国食物成分表），计算宏量和微量营养素
- **购物清单生成**：从周计划自动提取食材，合并重复项，按超市分区分组，检查库存避免重复购买
- **症状-食物关联分析**：持续记录饮食和症状，分析延迟反应模式，识别可疑触发食物
- **排除饮食协议**：生成结构化的消除-重引入计划，追踪每个阶段，确保替代食谱满足营养需求
- **微量营养素优化**：分析膳食计划中的营养缺口，推荐食物而非补剂——"在碗里加南瓜籽和葵花籽就能补充维生素 E 和锌"

## 所需技能

- **Mealie MCP Server**（`rldiao/mealie-mcp-server`）：食谱 CRUD、膳食计划、购物清单——共 45 个工具
- 定时任务（cron job）用于提醒和周度分析
- 文件存储用于症状日志和营养档案
- Telegram 或 Slack 用于交互界面
- HTTP 请求用于 USDA FoodData Central API 调用

## 设置方法

### 1. 部署 Mealie 并连接 MCP

首先需要一个运行中的 Mealie 实例（Docker 部署最简单），然后通过 MCP 连接到 OpenClaw：

```text
## Mealie MCP Setup

Install the Mealie MCP server:

fastmcp install mealie-mcp-server \
  --env-var MEALIE_BASE_URL=https://your-mealie-instance.com \
  --env-var MEALIE_API_KEY=your-mealie-api-key

Verify by asking me: "List all recipes in Mealie"
— I should return your recipe library via get_recipes.
```

> **凭证安全**：`MEALIE_API_KEY` 通过 MCP 环境变量传递，不要写入提示词或记忆文件。

### 2. 配置营养查询

注册 USDA FoodData Central API key（免费，覆盖 38 万+ 食品），配置智能体自动查询：

```text
## Nutrition Lookup

USDA_API_KEY is set in environment.

When I log a meal or create a meal plan:
1. For each ingredient, query USDA FoodData Central:
   GET https://api.nal.usda.gov/fdc/v1/foods/search?query={ingredient}&api_key=$USDA_API_KEY
2. Extract per-serving: calories, protein, fat, carbs, fiber, and key micros (iron, calcium, vitamin D, B12, zinc)
3. Sum across the meal, log to ~/clawd/memory/nutrition-log.json
4. Flag if any daily target is below 80% by dinner time
```

### 3. 膳食规划与购物清单

以下提示词让智能体每周日生成下周膳食计划并自动创建购物清单：

```text
## Weekly Meal Planning

Every Sunday at 9 AM:
1. Review my known intolerances from ~/clawd/memory/intolerance-profile.json
2. Check what's in my pantry (~/clawd/memory/pantry.json)
3. Generate a 7-day meal plan via Mealie:
   - Respect all flagged intolerances (exclude trigger foods)
   - Target: ~2000 kcal/day, 100g+ protein
   - Prefer recipes already in my Mealie library (use get_recipes with tag filters)
   - Fill gaps with new recipe suggestions (create_recipe if approved)
4. Create the meal plan in Mealie (create_mealplan_bulk for the week)
5. Generate shopping list (add_recipe_to_shopping_list for each planned recipe)
6. Cross-check pantry inventory, remove items I already have
7. Post the plan + shopping list to my Telegram channel for review

If I say "swap Tuesday dinner", suggest 3 alternatives matching the same nutritional targets and intolerance constraints.
```

### 4. 食物不耐受追踪与排除饮食

这是本用例的核心差异化功能——从被动记录升级为主动侦测：

```text
## Food Intolerance Tracking

When I report a symptom in the "health" Telegram topic:
1. Log symptom with timestamp and severity (1-5) to ~/clawd/memory/symptom-log.json
2. Cross-reference with meals from the past 72 hours (delayed reactions are common)
3. Update suspect scores in ~/clawd/memory/intolerance-profile.json

Weekly analysis (every Sunday, before meal planning):
- Calculate correlation between each food category and symptom occurrences
- Weight by time delay (24h reactions score higher than 72h)
- If a food category exceeds suspicion threshold (>0.7 correlation over 3+ incidents):
  → Flag as "suspect" and recommend elimination trial
  → Post to Telegram: "Nightshade vegetables appeared in 4/5 of your eczema episodes. Recommend 3-week elimination trial?"

## Elimination Protocol

When I confirm an elimination trial:
1. Tag all recipes containing the suspect category in Mealie (e.g., tag: "contains-nightshade")
2. Generate a 3-week meal plan that excludes the entire category
3. Verify nutritional completeness — suggest substitutes if any micro drops below target
   (e.g., removing tomatoes? Add red bell pepper for vitamin C — wait, that's also nightshade.
    Use broccoli and citrus instead.)
4. Set reintroduction reminders:
   - Week 4, Day 1: Reintroduce one item (e.g., cooked tomato, small portion)
   - Log symptoms for 72 hours before next reintroduction
5. After full reintroduction cycle, generate report:
   - Which specific items triggered reactions vs. safe ones
   - Confidence level based on data points
   - Recommended permanent exclusions vs. "tolerable in small amounts"
```

### 5. 购物清单增强（可选）

如果你使用 Apple 设备，可以结合 `grocery-list-skill`（`densign01/grocery-list-skill`）将购物清单同步到 Apple Reminders：

```text
## Grocery List Enhancement

After generating the weekly shopping list in Mealie:
1. Export items via get_shopping_list_items
2. Group by store section (produce, dairy, meat, pantry, frozen)
3. Use grocery-list-skill to push to Apple Reminders "Groceries" list
4. Include quantities from recipes (e.g., "3 onions", "500g chicken breast")
5. Skip pantry staples I always have (configurable in ~/clawd/memory/pantry-staples.json)
```

## 国内适配

国际方案依赖 USDA 数据库和 Mealie，国内用户可做以下替换：

| 国际方案 | 国内替代 |
|---|---|
| USDA FoodData Central | [中国食物成分表](https://www.chinanutri.cn/)（中国疾控中心营养所） |
| Mealie 食谱库 | 下厨房 / 美食天下 API 或本地食谱 JSON |
| 超市购物清单 | 美团买菜 / 叮咚买菜 / 盒马 小程序 |
| Apple Reminders 同步 | 微信提醒 或 滴答清单 |

中式食谱的特殊考量：
- **调味料追踪**：酱油（含大豆）、蚝油（含贝类）、豆瓣酱（含小麦）等中式调味料是常见过敏原，需要在食谱中显式标注
- **外卖整合**：通过识别外卖订单截图来记录非自制餐食的食材（视觉模型解析菜品名 → 匹配食物成分表）
- **时令食材**：中国农历节气对应的时令蔬果，在膳食规划中优先推荐当季食材

## 关键洞察

- **72 小时窗口是关键**：食物不耐受反应常延迟 24-72 小时，人脑无法可靠地跨天关联。这正是 AI 追踪的核心价值——自动回溯三天内所有摄入食物并计算相关性。
- **结构化数据 > 自由文本**：使用 Mealie 管理食谱意味着每顿饭的食材是结构化的，不需要从"吃了个三明治"中猜测具体用了什么食材。
- **营养安全网**：排除饮食最大的风险是营养缺失。自动营养查询确保你在排除某类食物时不会意外丢失关键营养素。
- **从追踪到行动的闭环**：发现触发食物只是第一步——系统自动调整膳食计划、更新购物清单、推荐替代食谱，形成完整闭环。
- **先做两周纯记录**：不要急于排除。先积累至少两周的完整饮食-症状数据，让模式自然浮现。

## 灵感来源

- **茄科蔬菜发现案例**：一位 HN 用户使用 AI 追踪两周饮食-症状数据后，发现番茄是湿疹的触发因素，随后通过排除饮食验证了这一发现。他在 [Hacker News 讨论](https://news.ycombinator.com/item?id=46499027)中分享了完整过程，包括使用 USDA API 自动查询营养成分、生成分类购物清单、以及通过微量营养素分析优化膳食的方法。
- **Mealie MCP 生态**：GitHub 上已有 10+ 个 Mealie MCP 实现（`rldiao/mealie-mcp-server` 最活跃，提供 45 个工具），覆盖食谱管理、膳食计划、购物清单的完整生命周期。
- **购物清单自动化**：`densign01/grocery-list-skill` 展示了从食谱 URL 抓取食材、合并重复项、同步到 Apple Reminders 的完整流程。
- **AI 饮食管家**：叮咚买菜等国内平台已开始探索 AI 辅助的个性化膳食推荐，包括过敏原识别和替代品推荐。

## 相关链接

- [Mealie — 自托管食谱管理器](https://mealie.io/)
- [Mealie MCP Server（rldiao）](https://github.com/rldiao/mealie-mcp-server) — 45 个 MCP 工具
- [USDA FoodData Central API](https://fdc.nal.usda.gov/api-guide/)
- [grocery-list-skill](https://github.com/densign01/grocery-list-skill) — 购物清单 → Apple Reminders
- [中国食物成分表](https://www.chinanutri.cn/)
