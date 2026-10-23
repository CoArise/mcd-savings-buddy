# workbuddy.md — WorkBuddy 开发过程记录

本参赛项目（麦麦省钱搭子 mcd-savings-buddy）全程使用腾讯 WorkBuddy 智能体开发，本文件为对话上下文摘要，用于 WorkBuddy 联动活动核验。

## 开发环境

- 工具：腾讯 WorkBuddy 智能体（桌面端，Agent 模式）
- 时间：2026 年 10 月 23 日
- 工作区：`麦当劳`（macOS）

## 开发过程摘要

1. **需求理解**：以麦当劳官方活动仓库 `mcd-developer-innovation-challenge` 的 README.md 与 activityGuidelines.md 为依据，确认参赛项目必须包含 README.md、CONTEST_DECLARATION.md（官方原文）、MCP_INTEGRATION.md、mcp-config.example.json、源代码等文件，且须真实调用麦当劳 MCP。
2. **MCP 连通性核验**：通过 WorkBuddy 中已配置的 `mcd-mcp` 自定义连接器（https://mcp.mcd.cn，Bearer Token 鉴权）实际调用 `now-time-info`、`query-my-account`、`query-nearby-stores` 三个工具，全部成功返回，确认服务可用、账号已授权（并读得真实麦享会积分数据），同时记录了 `query-nearby-stores` 在收藏为空时返回业务码 600050 的行为。
3. **Skill 规范加载**：调用 WorkBuddy 内置 skill-creator 技能，按其规范生成 skill 骨架（`init_skill.py`），并遵循渐进式披露原则组织内容。
4. **创意构思**：选定「麦麦省钱搭子」——按预算搭配餐品 + 匹配优惠券 + 计算最优价格 + 确认后下单的点餐决策 Skill。
5. **开发实现**：
   - 编写 `SKILL.md`（五步工作流：确认渠道 → 找门店 → 按预算搭配 → 匹配优惠算价 → 确认后下单；含 beType/orderType/beCode 配对规则、预约规则、写操作先确认后执行等安全约束）。
   - 编写 `references/mcd-tools.md` 工具速查表（仅记录实际核验过的参数 schema，避免臆造）。
   - 按活动要求补齐 README.md、MCP_INTEGRATION.md、mcp-config.example.json（仅 `${MCD_MCP_TOKEN}` 环境变量占位符）、CONTEST_DECLARATION.md（官方原文未改动）。
6. **验证**：通过 skill-creator 的 `package_skill.py` 完成结构与规范校验。

## 结论

本项目为"麦当劳 MCP + WorkBuddy Skill"的真实最小可运行实现：无自建后端，源代码即 SKILL.md 及其参考文档，安装进 WorkBuddy 后即可通过自然语言驱动麦当劳 MCP 完成找店、搭配、比价、下单全链路。
