# 麦麦省钱搭子（mcd-savings-buddy）

基于麦当劳 MCP 的预算点餐 Skill，运行在 WorkBuddy 中。给出位置、取餐方式和预算，它完成门店查询、餐品搭配、优惠券比对和实付金额计算，确认后可下单。

## 它解决什么问题

麦当劳的门店、菜单、券包和活动分散在 App 的不同页面。想回答"我现在、在这个位置、用 40 块钱，最多能吃到什么"，需要在多个页面之间来回切换、自己心算比价。本 Skill 把麦当劳 MCP 的十几个工具编排成一条固定链路，让这个决策在一次对话里完成。

## 功能

- 门店查询：覆盖到店自取、得来速（Drive-Thru）、麦乐送外送、企业团餐四种渠道，支持按位置搜索和预约时段选择
- 预算搭配：内置官方团餐搭配策略（20 元以下小食起，50 元以上丰富组合），输入预算直接给出菜单级方案
- 比价省钱：查询我的券、门店券和促销活动，对多个组合逐一核价，输出最省方案及与原价的差额
- 下单管理：订单明细确认后下单，支持订单查询和取消；取消后重买走重新支付，不会重复下单
- 会员权益：麦享会积分余额与过期提醒、积分商城、抽奖、用餐援助

写操作（绑券、下单、取消、新增地址）一律先展示明细，经用户确认后执行，不会自动支付。

## 安装

### 1. 配置麦当劳 MCP 连接器

在麦当劳开放平台 open.mcd.cn/mcp 登录并申请 MCP Token，然后在 WorkBuddy 左侧边栏打开【专家·技能·连接器】，进入【自定义连接器】→【配置 MCP】，粘贴以下内容并将 `YOUR_MCP_TOKEN` 替换为实际 Token：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

保存后回到【自定义连接器】，启用 `mcd-mcp`。脱敏配置模板见 `mcp-config.example.json`。

### 2. 安装 Skill

clone 本仓库，将整个目录放入：

- 用户级：`~/.workbuddy/skills/mcd-savings-buddy/`
- 项目级：`<工作区>/.workbuddy/skills/mcd-savings-buddy/`

重启对话后自动生效，也可以直接说"用麦麦省钱搭子帮我点餐"。

## 使用示例

```text
> 我在深圳南山科技园，中午想吃麦当劳，预算40块，到店自取，算一下最省的方案

Skill 依次调用 query-nearby-stores 找门店、query-meals 拉菜单，按预算生成
"汉堡 + 薯条/小食 + 饮料"组合，再查可用券并用 calculate-price 核价，
输出组合明细、使用的券、实付金额和与原价的差额，等待确认下单。

> 看看附近哪家麦当劳能开车取餐

query-nearby-stores（beType=5），返回带 beCode 的门店列表和预约时段。

> 我的麦享会积分还剩多少？快过期的有哪些？

query-my-account，返回可用积分、累计积分和本月/下月将过期积分。

> 这单不要了，取消吧

确认订单后调用 cancel-order；之后想重买同一单走重新支付，不重新下单。
```

## 目标用户

- 午晚高峰需要快速决定"吃什么、在哪吃、怎么最省"的上班族和学生
- 手里有券不知道怎么叠最划算、关心积分过期的麦当劳会员
- 需要按人头预算批量搭配团餐的企业行政
- 想参考麦当劳 MCP + WorkBuddy Skill 最小实现的开发者

## 项目结构

```text
mcd-savings-buddy/
├── SKILL.md                  # Skill 本体
├── references/
│   └── mcd-tools.md          # mcd-mcp 工具速查表
├── README.md
├── MCP_INTEGRATION.md        # MCP 集成说明（Server / Tool / 流程 / 价值）
├── mcp-config.example.json   # 脱敏 MCP 配置示例
├── CONTEST_DECLARATION.md    # 参赛声明
└── workbuddy.md              # WorkBuddy 开发过程记录
```

## 说明

本项目为参赛者独立开发，非麦当劳官方产品，见 CONTEST_DECLARATION.md。输出仅供参考，不构成医疗、营养或其他专业建议；餐品信息、价格及供应状态以麦当劳官方渠道的实时结果为准。
