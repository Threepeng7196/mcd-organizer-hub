# 麦麦主办方 · 团餐派对策划台

> 麦当劳程序员创意开发大赛参赛作品（基于麦当劳中国 MCP 能力开发的 WorkBuddy Skill）
> 本仓库由参赛者独立开发，非麦当劳官方产品，仅供参赛与学习交流。
> 建议仓库名：`mcd-organizer-hub`（你建仓库时可自定义）

## 项目介绍

一个面向**漫展 / 宅舞 / 社团活动主办方**的麦当劳团餐 + 派对一站式策划 Skill。
把"几十号人吃什么、预算怎么分、派对约哪家店哪天"这种主办方最头疼的体力活，压缩成几句话：

- **企业团餐批量采购**：按 人数 / 总预算 / 忌口（不吃辣、不吃牛肉…）/ 取餐方式 生成不超预算、不踩忌口、已叠加团餐满减的采购提案，逐单官方核价。
- **主题派对预约**：生日派对 / 主题派对 / 麦麦体验营 / 品鉴会 / 读书会 / 积分兑换活动，按 城市 → 门店 → 日期 → 场次 串成预约闭环。
- **分账与执行**：把整单按人 / 按组拆分（谁点了什么、人均多少），生成可分享的采购清单；下单前全部二次确认。
- **历史对账**：拉取历史订单，核对"提案价"与"实际支付"的差异。

目标用户：活动 / 漫展 / 宅舞 / 社团等需要为多人批量点麦当劳、或办麦当劳主题活动的组织者。

## 功能特性

- 企业团餐（beType=6）组合：`query-meals` 搭配 → `query-meal-assistance` 取助餐服务 → `query-promotions` 团餐满减 → `calculate-price` 核价
- 主题派对预约（shopId=5）链路：`mall-points-products` → `query-party-city` → `query-party-store` → `query-party-store-date` → `query-party-store-session` → `party-order-create`
- 多取餐方式支持：到店自提（beType=1）/ 得来速（beType=5）/ 麦乐送（外送地址）
- 忌口与预算硬约束：不满足的方案直接过滤，绝不"为了凑单超预算"
- 分账清单：按人 / 按组拆分，输出可分享文本
- 下单 / 预约前 100% 二次确认，不下静默单

## 安装方法

1. 申请麦当劳中国 MCP Token：访问 https://open.mcd.cn/mcp ，用手机号登录后点击「控制台 → 激活」申请 Token。
2. 在 WorkBuddy 中接入该 MCP：
   - 左侧【专家·技能·连接器】→【连接器】→ 右上角【自定义连接器】→【配置 MCP】
   - 填入 `mcp-config.example.json` 的内容，**把 `${MCD_MCP_TOKEN}` 替换为你自己的 Token**，保存并启用。
3. 把本仓库作为 WorkBuddy Skill 引入（将仓库 `SKILL.md` 所在的目录作为 Skill 目录，或在 WorkBuddy 中按 Skill 方式加载）。
4. 在对话框中直接说需求，AI 会调用麦当劳 MCP 工具完成操作。

> 也可使用 Cherry Studio / Cursor / Trae / VSCode 等支持 Streamable HTTP 的 MCP Client 接入，地址均为 `https://mcp.mcd.cn`。

## 使用示例

```
用户：周六漫展散场，12 个人、预算 600、2 个不吃辣、1 个不吃牛肉，帮我出一份团餐采购单
AI：（query-nearby-stores 定位门店 → query-meals 取团餐搭配 → query-meal-assistance(beType=6) 取助餐服务 → query-promotions 取满减 → calculate-price 核价）→
    输出：不超预算、避开忌口、已叠满减的候选套餐 + 人均价 + 分账清单，并请你确认后下单
```

```
用户：下个月给社团办个麦麦主题派对，北京，帮我看哪些门店哪些日期能约
AI：（mall-points-products 取派对商品 spuId → query-party-city → query-party-store(北京) → query-party-store-date → query-party-store-session）→
    输出：可预约门店 / 日期 / 场次表，确认后 party-order-create 预约
```

## 目录结构

```
.
├── README.md                # 本文件（必交）
├── CONTEST_DECLARATION.md   # 参赛声明（必交，内容不可改）
├── MCP_INTEGRATION.md       # MCP 接入说明（必交）
├── mcp-config.example.json  # 脱敏 MCP 配置示例（必交）
├── SKILL.md                 # Skill 源码/定义（必交：源代码）
└── workbuddy.md             # WorkBuddy 对话上下文（参与 WorkBuddy 专项奖时必交）
```

## 注意事项

- 本 Skill 涉及下单 / 预约等写操作时，一律先向用户说明将要调用的工具与参数，**得到确认后才执行**，不会静默下单或自动支付。
- 餐品、价格、供应状态、派对场次以麦当劳官方实时返回为准。
- 所有金额、券、积分数据仅通过 MCP 实时读取，不在本地留存凭证。
