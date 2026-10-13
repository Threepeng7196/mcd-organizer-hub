# MCP 接入说明（MCP_INTEGRATION）

## 使用的 MCP Server

- **名称**：麦当劳中国 MCP Server（mcd-mcp）
- **官方开放平台**：https://open.mcd.cn/mcp
- **接入地址**：`https://mcp.mcd.cn`
- **传输协议**：Streamable HTTP
- **鉴权方式**：请求头 `Authorization: Bearer <YOUR_MCP_TOKEN>`（Token 在 open.mcd.cn 用手机号登录后申请）
- **限流**：每 Token 600 次/分钟（超限返回 429）

## 实际使用的 Tool

本 Skill 围绕「企业团餐（beType=6）」与「主题派对（shopId=5）」两条链路组织能力，实际调用的工具如下：

### 一、企业团餐批量采购（beType=6）

| Tool | 用途 |
|---|---|
| query-nearby-stores | 到店场景定位门店（beType=1 到店自提 / beType=5 得来速） |
| delivery-query-addresses | 外送场景下查询用户已有配送地址（团餐说"我想点团餐"时） |
| delivery-query-stores | 根据地址 ID 查询可配送门店 |
| delivery-create-address | 新建配送地址（地址不存在时） |
| query-meals | 团餐餐品列表（含官方搭配规则），按规则组合 |
| query-meal-assistance | **团餐必调**：beType=6 在 calculate-price 前获取当前门店可用助餐服务 |
| query-promotions | 团餐场景可用的满减/满折促销规则（仅当前有效、适用在售餐品） |
| query-store-coupons / query-my-coupons / available-coupons | 查门店可用券 / 卡包券 / 可领券 |
| auto-bind-coupons | 一键领取麦麦省可用券 |
| calculate-price | 含优惠核价（必须先用 query-meal-assistance 取助餐服务） |
| create-order | 创建订单并返回支付链接（需用户确认） |

### 二、主题派对预约（shopId=5）

| Tool | 用途 |
|---|---|
| mall-points-products | 查可兑换商品列表，筛选派对/周边类目，取得 spuId |
| mall-product-detail | 商品详情（确认 spuId、shopId=5 方可预约派对） |
| query-party-city | 派对场次城市列表（需先有 spuId） |
| query-party-store | 指定城市下支持主题派对的门店列表 |
| query-party-store-date | 指定门店可预约日期列表 |
| query-party-store-session | 指定门店 + 日期下的场次列表 |
| party-order-create | 创建派对订单（仅 shopId=5） |

### 三、辅助 / 对账

| Tool | 用途 |
|---|---|
| query-my-account | 查积分账户（积分兑换相关） |
| campaign-calendar | 查营销活动日历（结合派对/团餐时间点） |
| now-time-info | 取官方服务器时间（判断餐档/活动时效） |
| query-meal-detail | 餐品组成 / 替换项查询 |
| list-nutrition-foods | 官方营养数据（忌口/营养参考） |
| order-list / query-order | 非商城历史订单查询与对账 |
| mall-order-list / mall-order-detail | 麦麦商城订单查询与对账 |

## 调用流程

### 企业团餐流程
1. 收集需求：人数、总预算、忌口清单、取餐方式（自提/得来速/麦乐送）、城市或配送地址。
2. 定位门店：到店 → `query-nearby-stores`(beType=1/5)；外送 → `delivery-query-addresses` → `delivery-query-stores`（缺地址则 `delivery-create-address`）。
3. 取团餐数据：`query-meals` 取搭配规则 → `query-meal-assistance`(beType=6) 取助餐服务 → `query-promotions` 取满减 → 按需 `query-store-coupons`/`auto-bind-coupons` 领券。
4. 组合 + 核价：多候选方案经 `calculate-price` 逐一官方核价，过滤超预算 / 踩忌口的方案。
5. 分账 + 二次确认：输出按人/按组分摊清单，用户确认后 `create-order` 返回支付链接。

### 主题派对流程
1. `mall-points-products` 取派对/周边商品 spuId。
2. `mall-product-detail`(spuId) 确认 shopId=5。
3. `query-party-city`(spuId) → `query-party-store`(城市) → `query-party-store-date`(门店) → `query-party-store-session`(门店+日期)。
4. 用户选定场次 → `party-order-create` 预约（需用户确认）。

## 业务价值

活动主办方最痛的不是"吃什么"，而是"几十号人怎么凑、预算怎么分、派对约哪家哪天"。
现有同类作品大多从"普通聚餐"出发，没人用主办方的真实视角（忌口矩阵、整桌预算、分账、批量预约）来做。
本 Skill 把企业团餐与主题派对两条官方原子能力串成可落地的闭环，显著降低多人活动的决策与执行成本。
