# mcd-mcp 工具速查表

Server：`mcd-mcp`（`https://mcp.mcd.cn`，streamablehttp，Bearer Token 鉴权）。
下表按点餐链路分组。标注（已核验）的工具在本 skill 开发过程中实际调用并验证过。

## 链路分组

| 环节 | 工具 | 用途 |
|---|---|---|
| 时间 | `now-time-info`（已核验） | 服务器当前时间，活动日历/预约前先取 |
| 门店（到店） | `query-nearby-stores`（已核验） | 按收藏（searchType=1）或位置（searchType=2，必传 city+keyword）查可点餐门店；beType=1 自提 / 5 得来速 |
| 门店（外送） | `delivery-query-stores` | 查可外送门店，返回外送用 beCode |
| 地址 | `delivery-query-addresses` / `delivery-create-address` | 查询 / 新增外送收货地址 |
| 菜单 | `query-meals`（已核验） | 按门店与渠道拉取餐品列表，含商品 code 与价格 |
| 餐品详情 | `query-meal-detail` | 单个餐品详情 |
| 营养 | `list-nutrition-foods` | 餐品营养信息 |
| 优惠 | `query-my-coupons` / `query-store-coupons` / `query-promotions` / `campaign-calendar` / `available-coupons` | 我的券 / 门店券 / 促销 / 活动日历 / 本单可用券 |
| 绑券 | `auto-bind-coupons` | 自动绑定可用券（写操作） |
| 算价 | `calculate-price` | 按商品组合 + 券计算实付 |
| 下单 | `create-order` / `party-order-create` | 普通下单 / 团餐派对下单 |
| 订单 | `query-order` / `order-list` / `cancel-order` | 查单 / 列表 / 取消 |
| 团餐派对 | `query-party-city` / `query-party-store` / `query-party-store-date` / `query-party-store-session` | 派对城市 / 门店 / 日期 / 场次查询 |
| 积分 | `query-my-account`（已核验）/ `mall-points-products` / `mall-create-order` / `mall-order-detail` / `mall-order-list` | 积分账户 / 积分商城兑换 |
| 抽奖 | `draw-lottery` / `query-lottery-info` / `query-my-prizes` | 抽奖 / 奖项 / 我的奖品 |
| 其他 | `query-meal-assistance` / `query-survey-coupon` | 用餐援助 / 问卷领券 |

## 核心工具参数（已核验）

### now-time-info

无参数。返回时间戳、格式化时间、年月日、星期、时区。

### query-my-account

无参数。返回可用积分、累计积分、冻结、已用、本月/下月将过期积分（币种：麦享会积分）。

### query-nearby-stores

| 参数 | 必填 | 说明 |
|---|---|---|
| beType | 是 | 1=到店自提，5=得来速 |
| searchType | 是 | 1=搜索收藏餐厅（默认），2=按位置搜索 |
| city | searchType=2 时必填 | 城市名 |
| keyword | searchType=2 时必填 | 位置关键词 |

返回 `storeName`、`address`、`distance`、`storeCode`（唯一）、`beCode`（仅得来速非空）、`reservation`、`reservationTimeOptions`、`today`。收藏为空时返回 code=600050 业务提示，改用 searchType=2。

### query-meals

| 参数 | 必填 | 说明 |
|---|---|---|
| storeCode | 是 | 门店编码 |
| orderType | 是 | 1=到店（自取+得来速），2=外送（麦乐送+团餐） |
| beType | 是 | 1=到店取餐，2=麦乐送，5=得来速，6=企业团餐 |
| beCode | 渠道决定 | 得来速/外送/团餐必传；到店自取不传 |
| reservationDate | 预约场景 | 格式 `yyyy-MM-dd HH:mm` |

> 其余工具以 MCP Server 实际返回的 schema 为准；本表仅记录开发时核验过的内容，避免臆造参数。
