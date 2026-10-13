# WorkBuddy 对话上下文（workbuddy.md）

> 本文件用于核验你「真实使用腾讯 WorkBuddy 智能体」开发，参与 WorkBuddy 专项奖励（前 100 名提交可得 3000 积分）。
> 以下内容整理自本次 WorkBuddy 开发会话（2026-10-09 ~ 2026-10-13），真实记录需求、接入、调试与产出过程。

## 开发对话要点

### 需求：这个 Skill 要解决什么问题

- 背景：我是漫展 / 宅舞类活动的主办方，经常要为几十上百人的活动批量订麦当劳团餐，或预约麦当劳主题派对场地。
- 痛点：算人均预算、凑满减不超支、按人数/忌口配餐、分账，以及派对按「城市 → 门店 → 日期 → 场次」预约，全靠手工在 App 里反复试，低效且容易超预算。
- 目标：做一个面向**活动主办方**的 Skill——输入人数 / 预算 / 忌口，自动生成不超预算的企业团餐提案并核价分账；并打通主题派对预约链路。

### 接入：如何在 WorkBuddy 里接入麦当劳 MCP

1. 在 WorkBuddy 内对话，让 AI 协助走完「麦当劳 MCP Token 申请」：登录 open.mcd.cn → 控制台 → 激活，拿到 Bearer Token。
2. AI 读取并合并本地 MCP 配置（`~/.workbuddy/mcp.json`），新增 `mcd-mcp`（`streamableHttp`，`https://mcp.mcd.cn`），保留已有服务不覆盖；真实 Token 只存用户目录，**不进项目代码**，项目内仅保留 `${MCD_MCP_TOKEN}` 占位（见 `mcp-config.example.json`）。
3. 在 WorkBuddy【连接器】→【自定义连接器】中启用并「信任」`mcd-mcp`。

### 调试：遇到的坑 & 如何验证 MCP 工具可用

- **验证 Token**：直接对 `https://mcp.mcd.cn` 发 MCP 协议 `initialize`，返回 `serverInfo: mcd-mcp 1.0.0`，确认 Token 有效。
- **列工具**：`tools/list` 实际返回 **35 个工具**，据此核准团餐 / 派对链路的真实工具名（`query-meal-assistance`、`query-promotions`、`query-meals`、`calculate-price`、`create-order`、`mall-points-products`、`query-party-city/store/store-date/store-session`、`party-order-create` 等），文档不编造任何工具名。
- **入参坑**：`query-nearby-stores` 有必传参数 `beType`（1=到店自提 / 5=得来速 / 6=企业团餐）与 `searchType`，且按位置搜需带 `city` 或 `keyword`——光说「附近有开麦吗」会缺参，实测后在 Skill 里补全默认值与追问逻辑。
- **限流**：服务端 600 次/分钟，Skill 内对批量查询做了节流。
- **安全自查**：每次提交前扫描项目，确认无真实 Token / PAT 泄露（只允许占位符出现）。

### 效果：最终能跑通的使用示例

- 「我们社团 40 人、总预算 2000 元、有 3 人不吃牛肉，帮我配团餐方案」→ 调 `query-meal-assistance` + `query-promotions` 生成满减最优搭配 → `calculate-price` 核价 → 输出人均/分账明细，确认后再 `create-order`（写操作二次确认）。
- 「10 月想在 XX 市办 50 人主题派对」→ `query-party-city` → `query-party-store` → `query-party-store-date` → `query-party-store-session` 返回可选场次 → `party-order-create` 下单（二次确认）。
- 环境：Windows 11 + WorkBuddy，MCP 配置在 WorkBuddy 连接器内，本 Skill 为纯文档 + 配置形态（Skill 定义见 `SKILL.md`）。

## 开发过程时间线（本会话）

1. 用 WorkBuddy 检索 GitHub 官方仓库 `M-China/mcd-developer-innovation-challenge`，读懂规则与必交文件。
2. 在 WorkBuddy 内拉取官方 100 条 open issues 盘点赛道红海，确定差异化方向「活动主办方团餐 & 派对」。
3. 让 WorkBuddy 生成并写实 README / MCP_INTEGRATION / SKILL / 声明文件骨架。
4. 由 WorkBuddy 协助完成 Token 申请接入、MCP 连通性验证、35 工具清单核准与入参调试。
5. git 初始化、提交、推送公开仓库（凭据经临时注入，无残留）。
