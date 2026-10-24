# MCP 集成说明（MCP Integration）

本项目「麦麦防过期管家」基于麦当劳中国 MCP Server 开发，核心目标是**帮助用户管理账户内「即将过期」的积分与优惠券**，避免隐性浪费。

## 1. 使用的 MCP Server

| 项 | 值 |
| --- | --- |
| Server 名称 | `mcd-mcp` |
| 接入地址 | `https://mcp.mcd.cn` |
| 传输协议 | Streamable HTTP |
| 认证方式 | 请求头 `Authorization: Bearer <MCP_TOKEN>` |
| 官方文档 | <https://github.com/M-China/mcd-mcp-server> |

## 2. 使用的 Tool 与用途

| Tool | 用途 | 在本项目中的角色 | 是否只读 |
| --- | --- | --- | --- |
| `query-my-account` | 查询积分账户（可用 / 累计 / 冻结 / 即将过期积分） | 账户体检核心数据源 | ✅ 只读 |
| `query-my-coupons` | 查询名下全部优惠券及有效期 | 券过期预警核心数据源 | ✅ 只读 |
| `now-time-info` | 获取当前时间 | 计算「距离过期还有几天」的基准 | ✅ 只读 |
| `available-coupons` | 查询当前可领取的「麦麦省」券列表 | 「一键领券」前置查询 | ✅ 只读 |
| `campaign-calendar` | 查询当月营销活动日历 | 「聪明花」匹配当期活动 | ✅ 只读 |
| `query-store-coupons` | 查询指定门店可用优惠券 | 「聪明花」匹配门店优惠 | ✅ 只读 |
| `mall-points-products` | 查询商城可用积分兑换的商品 | 用即将过期积分兑换的建议来源 | ✅ 只读 |
| `calculate-price` | 按商品列表（含优惠券）计算金额 | 「聪明花」方案的实付价测算 | ✅ 只读 |
| `auto-bind-coupons` | 一键领取全部可领优惠券 | 「一键领券」执行动作 | ⚠️ 有副作用，需用户显式确认 |

### 2.1 接口实测字段（2026-10-24 真实联调）

`query-my-account` 返回的积分账户字段。**注意：官方字段拼写为 `Mouth`，不是 `Month`，取值时必须按原样使用。**

| 字段 | 含义 |
| --- | --- |
| `availablePoint` | 可用积分 |
| `accumulativePoint` | 累计积分 |
| `currentMouthExpirePoint` | **本月将过期积分**（本项目核心预警字段） |
| `nextMouthExpirePoint` | 下月将过期积分 |
| `lastMouthExpirePoint` | 上月已过期积分 |
| `expiredPoint` | 已过期积分 |
| `frozenPoint` | 冻结积分 |
| `usedPoint` | 已使用积分 |
| `accountId` / `currency` | 账户 ID / 货币类型（麦享会积分） |

`query-my-coupons` 返回的是**结构化文本**（不是 JSON），每张券含「优惠 / 有效期 / 领取时间 / 标签」；有效期形如 `2026-10-24 00:00-2026-11-22 23:59`，需按 `-` 拆出右段作为到期时间。

> 联调环境：MCP Server `mcd-mcp v1.0.0`，协议版本 `2025-03-26`；`tools/list` 实际返回 **36 个工具**，比公开 README 多出 `query-promotions`（促销查询）、`query-survey-coupon`（问卷券）。

## 3. 调用流程

### 流程 A：账户体检 + 过期预警（默认、纯只读）

```
用户请求
  └─> query-my-account        → 读取积分账户（含「即将过期积分」）
  └─> query-my-coupons        → 读取券列表 + 有效期
  └─> now-time-info           → 获取当前日期
  └─> 本地计算分级：
        30 天 / 7 天 / 3 天 / 今天到期
  └─> 输出「账户体检报告」
  └─>（可选）交给 src/index.html 生成可视化体检卡
```

### 流程 B：聪明花（只读 + 比价）

```
以「即将过期资源」为输入
  └─> campaign-calendar      → 当前可参与的活动
  └─> query-store-coupons    → 门店可用优惠（结合用户所在门店）
  └─> mall-points-products   → 积分可兑换商品（优先消耗即将过期积分）
  └─> 组合出 N 个候选方案
  └─> calculate-price        → 计算每个方案实付价 / 优惠金额
  └─> 按「单位过期积分 / 单位券面值 的利用效率」排序，给出 Top 3
```

### 流程 C：一键领券（有副作用，需确认）

```
用户明确要求「帮我领券」
  └─> available-coupons      → 列出可领券
  └─> 向用户展示清单，等待显式确认  ←—— 副作用前置确认
  └─> 用户确认后：auto-bind-coupons → 一键领取
  └─> 回报领取结果
```

> 设计原则：**读操作直接执行，写操作（领券 / 下单）必须先经用户确认**，避免误领、误消费。

## 4. 业务价值

1. **降低会员资产浪费**：麦当劳会员积分、优惠券是「有时效」的资产，本项目把「什么时候会过期、怎么花最划算」变成一条清晰的、可行动的提醒，直接减少用户损失。
2. **盘活存量数据**：`query-my-account`、`query-my-coupons` 这类「读自己的账户」的工具，平时很少被普通用户主动打开；本项目把它们变成了一个「体检」入口，让存量能力产生日常价值。
3. **从「提醒」到「行动」闭环**：不仅告诉你「快过期」，还通过 `campaign-calendar` / `mall-points-products` / `calculate-price` 给出「现在怎么花最划算」的可执行方案，把提醒转化为下单 / 兑换动作。
4. **传播性**：生成的「账户体检卡」可分享，带动更多人关注自己的账户时效资产，形成自然传播。
