# MCP 集成说明 · mcd-mcp

> 本文说明猫猫点餐工具实际使用的麦当劳 MCP Server、Tool、调用流程与业务价值。
> 配套文件：`mcp-config.example.json`（脱敏配置）、`CONTEST_DECLARATION.md`（合规声明）。

## 一、使用的 MCP Server

| 项 | 内容 |
| --- | --- |
| Server 名称 | `mcd-mcp` |
| 能力域 | 麦当劳点餐（菜单、计价、下单、订单查询 / 取消、门店） |
| 接入方式 | 由 WorkBuddy 代理侧承载；浏览器前端通过适配层桥接调用 |
| 当前默认模式 | `MockMcdMcpClient`（内存假数据，字段对齐真实契约，离线可跑通全流程） |
| 真实模式 | `AgentMcdMcpClient`（桥接至代理侧执行的 mcd-mcp 工具，需授权与沙箱环境） |

> **诚实说明**：本仓库默认以 Mock 实现演示；`AgentMcdMcpClient` 是**文档化桥接**——其每个方法标注了对应的真实 `mcd-mcp` 工具名与请求体示例，真实调用需在 WorkBuddy 代理侧将方法体替换为对 MCP 工具的调用，并持有相应授权。仓库内不包含任何真实 MCP 凭证。

## 二、实际使用的 Tool 清单

业务层只依赖统一接口 `McdMcp`，每个方法对应一个真实 `mcd-mcp` 工具：

| # | mcd-mcp 工具 | 业务方法 | 用途 |
| --- | --- | --- | --- |
| 1 | `query-meals` | `queryMeals` | 浏览菜单（按门店 / 订单类型） |
| 2 | `query-meal-detail` | `queryMealDetail` | 获取菜品详情与可定制项（特制） |
| 3 | `calculate-price` | `calculatePrice` | 计价，返回 `takeWayList`、满减等 |
| 4 | `create-order` | `createOrder` | 下单（核心） |
| 5 | `query-order` | `queryOrder` | 订单状态查询 |
| 6 | `cancel-order` | `cancelOrder` | 取消订单（含原因码） |
| 7 | `order-list` | `orderList` | 历史订单列表 |
| 8 | `query-nearby-stores` | `queryNearbyStores` | 附近门店查询 |

## 三、调用流程

```
[浏览器 app.js]
      │  仅依赖 McdMcp 统一接口
      ▼
[McdMcp 适配层]
      ├── MockMcdMcpClient（默认）：内存假数据，字段对齐真实契约
      └── AgentMcdMcpClient（真实）：桥接至 WorkBuddy 代理侧
                      │
                      ▼
              [mcd-mcp 工具]  ← 代理侧执行，需授权
```

1. 前端 `app.js` 在初始化时实例化一个 `McdMcp` 实现（默认 `MockMcdMcpClient`）。
2. 所有点餐操作（菜单 → 详情 → 加购 → 计价 → 下单 → 查单 → 取消）均通过该实例的异步方法完成，并以 `try/catch` 统一捕获 `McdError`。
3. 切换真实环境：将 `app.js` 顶部的 `new McdMcp.MockMcdMcpClient()` 改为 `new McdMcp.AgentMcdMcpClient()`，并在 WorkBuddy 代理侧将 `AgentMcdMcpClient` 各方法体替换为对 `mcd-mcp` 工具的调用（示例见 `mcd-mcp.js` 内注释）。

## 四、关键字段契约

- `orderType`：`1` = 到店，`2` = 外送。
- `beType`（取餐 / 履约方式）：`1`=到店自取，`2`=麦乐送，`5`=得来速，`6`=团餐。
- 到店（`orderType=1`）下单**必传** `takeWayCode`（取自 `calculate-price` 返回的 `takeWayList`）。
- 外送（`orderType=2`）下单**必传** `addressId` 与备注（≤50 字）。
- `cancelReasonCode`：`1`=改主意、`2`=重复下单、`3`=点错了、`4`=地址填错、`5`=送达时间错、`-1`=其它。
- 下单返回 `orderId` + `payUrl`；支付在真实环境由用户在官方渠道完成，Mock 提供 `mockPay` 仅用于演示状态推进。

## 五、业务价值

1. **快速构建点餐应用**：通过 MCP 标准化接口，前端无需关心后端细节，即可在数小时内构建完整点餐闭环。
2. **演示与真实双轨**：`Mock` 实现保证演示 / 评审可离线跑通；`Agent` 实现提供清晰的真实对接契约，降低接入成本。
3. **零依赖、低门槛**：纯前端 + 单文件适配层，无构建步骤，双击即开，便于评审与二次开发。
4. **趣味化体验**：猫猫主题降低使用门槛，验证「MCP + 创意前端」结合的产品潜力。

## 六、安全与合规提示

- 真实 `mcd-mcp` 调用须在**沙箱 / 测试账号**进行，切勿对生产账户发起未授权操作。
- MCP 配置一律使用环境变量（见 `mcp-config.example.json`），严禁提交明文密钥。
- 详见 `CONTEST_DECLARATION.md`。
