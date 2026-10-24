# workbuddy.md — WorkBuddy 开发对话上下文

> 本文件用于核验本项目是否真实使用腾讯 WorkBuddy 智能体开发，是参与 WorkBuddy 专项奖励（排行榜前 100 名提交本文件可获得 3,000 积分）的必需材料。
> 以下按真实开发时间线记录关键对话意图与产物。

## 项目

- 项目名称：麦麦防过期管家（McExpiryGuard）
- 仓库：`mcd-expiry-guard`
- 开发工具：腾讯 WorkBuddy

---

## 第 1 轮 · 活动解读与摸底

**我的诉求：**
> 详细分析「麦当劳程序员创意开发大赛」活动细则，并读取麦当劳 MCP 能力，帮我做选题判断。

**WorkBuddy 产出（关键结论）：**
- 报名截止 2026-10-25 23:59，排名完全看 GitHub 公开 Star 数，Star 数为 0 不进榜。
- 仓库必须包含五个标准文件：`README.md`、`CONTEST_DECLARATION.md`、`MCP_INTEGRATION.md`、`mcp-config.example.json`、`workbuddy.md`。
- 报名走官方 Issue 模板；`CONTEST_DECLARATION.md` 文件名与内容均不可改动。
- 麦当劳 MCP 共 35 个 Tool，覆盖点餐 / 优惠券 / 积分 / 商城 / 抽奖 / 主题活动 / 团餐等场景。

**我的决策：** 先读规则再动手；选题避开红海（省钱、营养、人格测试已有大量同质作品）。

---

## 第 2 轮 · 选题与立项

**我的诉求：**
> 帮我找一个「比较小众、但日常生活里不起眼却经常用到」的切入点，且必须强绑定麦当劳 MCP 能力。

**WorkBuddy 给出的候选与我的取舍：**

| 候选 | 判定 | 理由 |
| --- | --- | --- |
| 省钱助手 | ❌ 放弃 | 榜单前三全是省钱，同质化严重 |
| 营养 / 卡路里 | ❌ 放弃 | 已有大量作品 |
| 人格测试 | ❌ 放弃 | 传播性强但弱绑定 MCP、偏娱乐 |
| **账户防过期管家** | ✅ 选定 | 痛点真实高频（积分/券过期浪费反复发生）、几乎无人做、强绑定 `query-my-account` + `query-my-coupons` |

**最终立项：** 「麦麦防过期管家」——账户体检 + 过期预警 + 聪明花（把快过期资源与活动/兑换匹配）+ 可分享体检卡。

---

## 第 3 轮 · MCP 工具探查与功能设计

**我的诉求：**
> 读取麦当劳 MCP 工具列表，确认「防过期」能用哪些接口实现，并设计只读优先、写操作需确认的安全边界。

**WorkBuddy 产出的工具映射（已写入 `MCP_INTEGRATION.md`）：**

- 体检：`query-my-account`（含「即将过期积分」字段）、`query-my-coupons`（券 + 有效期）
- 时间基准：`now-time-info`
- 聪明花：`campaign-calendar`、`query-store-coupons`、`mall-points-products`、`calculate-price`
- 领券：`available-coupons`（只读查询）→ `auto-bind-coupons`（副作用，需显式确认）

**安全设计：** 读操作直接执行；`auto-bind-coupons`、`create-order` 等写操作必须等待用户确认。

---

## 第 4 轮 · 产出参赛文件

**我的诉求：**
> 生成完整的参赛文件包，包括五个必交文件、Skill 核心提示词、可视化报告卡源码。

**WorkBuddy 产出：**

- `README.md`（介绍 / 安装 / 示例 / 目标用户）
- `CONTEST_DECLARATION.md`（官方原文，一字未改）
- `MCP_INTEGRATION.md`（Server / Tool / 调用流程 / 业务价值）
- `mcp-config.example.json`（仅 `${MCD_MCP_TOKEN}` 环境变量占位符，无真实凭证）
- `SKILL.md`（智能体业务逻辑）
- `src/index.html`（纯前端、零依赖、可离线的「账户体检卡」生成器）

---

## 第 5 轮 · 合规与脱敏自查

**我的诉求：**
> 对仓库做一次提交前合规检查：确认无真实 Token、无他人个人信息、无指向违规站点的外链、无恶意脚本。

**WorkBuddy 产出（自查结论）：**
- ✅ Token 仅以 `${MCD_MCP_TOKEN}` 占位符形式出现
- ✅ 无真实手机号 / 邮箱 / 地址等他人个人信息
- ✅ 报告卡为纯静态 HTML，无外部网络请求（无 CDN 依赖、无外链）
- ✅ 写操作（领券 / 下单）均设计为显式确认后才执行

---

## 附：为什么这个项目真实依赖 WorkBuddy

1. 选题与竞品分析、MCP 能力梳理均由 WorkBuddy 通过对话完成；
2. Skill 的调用流程、安全边界、提示词均在此对话中设计并落地；
3. 参赛文件的撰写、结构编排、合规自查由 WorkBuddy 协助完成并输出为可提交文档。
