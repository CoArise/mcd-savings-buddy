# MCP 集成说明（MCP_INTEGRATION.md）

本项目真实调用麦当劳 MCP 能力。本文档说明所使用的 MCP Server、Tool、调用流程与业务价值。

## 1. MCP Server

| 项 | 值 |
|---|---|
| Server 名称 | `mcd-mcp` |
| 地址 | `https://mcp.mcd.cn` |
| 协议类型 | streamablehttp |
| 鉴权方式 | HTTP Header `Authorization: Bearer <MCP Token>`（Token 在麦当劳开放平台 open.mcd.cn/mcp 申请） |
| 接入方式 | WorkBuddy 自定义连接器（配置模板见 `mcp-config.example.json`，Token 仅以环境变量占位符表示） |

## 2. 实际使用的 Tool

### 核心链路（已实际调用核验）

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `now-time-info` | 获取服务器当前时间 | 活动日历、预约时段等场景的时间基准 |
| `query-my-account` | 查询麦享会积分账户 | 积分余额/过期提醒/积分权益提示 |
| `query-nearby-stores` | 查询可点餐门店（自提 beType=1 / 得来速 beType=5；searchType=2 按城市+关键词搜索） | 第 1 步找门店，产出 `storeCode`、得来速 `beCode`、预约时段 |
| `query-meals` | 拉取门店菜单（orderType=1 到店 / 2 外送；beType 1/2/5/6 四渠道配对 beCode） | 第 2 步按预算搭配餐品，产出商品 `code` 与价格 |

### 编排使用（按 SKILL.md 工作流调用）

| 环节 | Tool |
|---|---|
| 外送门店/地址 | `delivery-query-stores`、`delivery-query-addresses`、`delivery-create-address` |
| 优惠匹配 | `query-my-coupons`、`query-store-coupons`、`query-promotions`、`campaign-calendar`、`available-coupons`、`auto-bind-coupons` |
| 算价下单 | `calculate-price`、`create-order`、`party-order-create` |
| 订单管理 | `query-order`、`order-list`、`cancel-order`（取消后重买走"重新支付"，复用原单支付凭证） |
| 会员增值 | `mall-points-products`、`mall-create-order`、`mall-order-detail`、`mall-order-list`、`draw-lottery`、`query-lottery-info`、`query-my-prizes`、`query-meal-assistance`、`query-meal-detail`、`list-nutrition-foods`、`query-survey-coupon` |
| 团餐派对 | `query-party-city`、`query-party-store`、`query-party-store-date`、`query-party-store-session` |

## 3. 调用流程

```text
用户需求（城市/位置 + 渠道 + 预算）
        │
        ▼
[0] now-time-info ── 仅活动/预约场景需要时间基准
        │
        ▼
[1] query-nearby-stores (beType=1|5) 或 delivery-query-stores
        │   产出: storeCode（+得来速/外送 beCode、reservationTimeOptions）
        ▼
[2] query-meals (storeCode + orderType + beType [+ beCode])
        │   产出: 商品 code、价格 → 按预算策略生成搭配组合
        ▼
[3] query-my-coupons / query-store-coupons / query-promotions
    → auto-bind-coupons（告知用户后）→ calculate-price 比价
        │   产出: 最省组合 = 商品清单 + 券 + 实付金额 + 省了多少
        ▼
[4] 用户确认明细与金额 → create-order（或 party-order-create）
        │
        ▼
[5] query-order / order-list 跟踪；cancel-order 取消
```

关键编排规则（写入 SKILL.md，保证调用正确性）：

- **beType/orderType/beCode 配对**：到店自取不传 beCode；得来速、外送、团餐必须传对应门店返回的 beCode。
- **预约场景**：`query-meals` 传 `reservationDate`（yyyy-MM-dd HH:mm），并完整展示 `reservationTimeOptions`。
- **写操作安全**：绑券、下单、取消、加地址均先展示、后执行；绝不自动支付。
- **重新支付**：用户取消后重买同一单时复用原单支付凭证，不重新下单。

## 4. 业务价值

1. **决策成本**：门店、菜单、券、价格原本分散在 App 不同页面，本 skill 将其合并为一次对话内的单次决策，直接回答"在此地、此预算下最省的吃法"。
2. **真实省钱**：券与促销通过 MCP 实时获取并参与算价，输出的"省了多少"是可复核的实付对比，而非营销话术。
3. **覆盖四渠道与团餐**：自取、得来速、麦乐送、团餐统一入口；团餐按官方预算档位策略自动搭配，对企业行政是刚需。
4. **资产不浪费**：积分余额与将过期积分主动提醒，衔接积分商城兑换，避免会员权益沉默。
5. **可复制的最小实现**：全项目仅由 SKILL.md + 参考文档构成，无自建后端，是"麦当劳 MCP + AI Skill"最容易复刻的参考架构。
