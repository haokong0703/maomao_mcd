✨ 功能特性
- 🍽️ 菜单浏览：5 大分类（汉堡 / 小食 / 甜品 / 饮品 / 儿童餐），猫猫化命名，标注「可定制」商品。
- 🐾 菜品选择：详情弹窗选择特制项（去酸黄瓜、辣度、冰量、糖度、主餐、玩具），数量步进。
- 🛒 购物车：按「商品 + 特制签名」去重、增减 / 删除、实时小计与合计。
- 💰 结算下单：选择「到店自取 / 麦乐送」→ 调用计价（返回 takeWayList、满减）→ 下单（到店必传 takeWayCode，外送必传 addressId + 备注 ≤ 50 字）。
- 📋 订单状态管理：状态机 待支付 → 已支付 → 制作中 → 待取餐 → 已完成；待支付可「模拟支付」；每 5 秒自动刷新；已出餐前可取消（原因码 3=点错了）。
- 🛡️ 错误处理：所有 mcd-mcp 调用包裹 try/catch，统一 Toast 报错 + 加载态；内置「模拟下单失败」开关验证错误分支（已覆盖 EMPTY_CART / NO_ADDRESS 等校验）。
🧱 技术栈
- 纯原生 HTML + CSS + JavaScript，零第三方依赖，无需 npm install。
- 脚本为非模块写法（挂全局 window.McdMcp），因此双击 index.html 即可直接运行（也支持 file://）。
- 适配层 mcd-mcp.js 同时兼容浏览器全局与 Node require（用于冒烟测试）。
📁 目录结构
cat-order-tool/
├── index.html        # 页面骨架（门店/菜单/购物车/我的订单/弹窗）
├── styles.css        # 猫猫主题样式（暖橙奶白、圆角卡片、爪印元素）
├── mcd-mcp.js        # ★ 适配层：统一 McdMcp 接口 + Mock / Agent 双实现
├── app.js            # 交互逻辑 + 订单状态管理 + 错误处理
├── test_smoke.js     # 冒烟测试：验证 Mock 全流程字段契约（node 运行）
├── serve.js          # 零依赖静态服务器（npm start 用）
├── package.json      # 脚本入口（test / start）
├── .gitignore
└── LICENSE
🚀 快速开始
方式一：直接打开（最省事）
双击 index.html，浏览器即可使用（Mock 模式，无需联网）。
方式二：本地静态服务器（推荐）
npm start
# 浏览器访问 http://localhost:5173
（serve.js 仅用 Node 内置模块，零依赖。PORT 环境变量可自定义端口。）
🔌 与 mcd-mcp 对接
浏览器页面无法直接调用 MCP 服务，因此本工具把业务与传输解耦：
业务层 app.js  ──仅依赖──▶  McdMcp 接口（mcd-mcp.js）
                                  ├── MockMcdMcpClient      （默认，内存假数据，离线可跑）
                                  └── AgentMcdMcpClient     （真实桥接，代理侧执行 mcd-mcp 工具）
切换真实环境
编辑 app.js 顶部第 11 行：
// 默认 Mock（演示 / 本地开发）
const mcd = new McdMcp.MockMcdMcpClient();
// 接入真实 mcd-mcp 时改为：
// const mcd = new McdMcp.AgentMcdMcpClient();
再在 AgentMcdMcpClient 各方法体里，把 _call(toolName, payload) 替换为对真实 mcd-mcp 工具的调用（每个方法上方已注释真实工具名与请求体示例，由 WorkBuddy 代理侧执行）。
接口契约对照表
      业务方法 (McdMcp)
      对应 mcd-mcp 工具
      说明
      queryMeals
      query-meals
      菜单浏览（需 storeCode + orderType）
      queryMealDetail
      query-meal-detail
      菜品详情 + 特制选项
      calculatePrice
      calculate-price
      计价（到店返回 takeWayList）
      createOrder
      create-order
      下单（核心）
      queryOrder
      query-order
      订单状态查询
      cancelOrder
      cancel-order
      取消订单（需 cancelReasonCode）
      orderList
      order-list
      历史订单
      queryNearbyStores
      query-nearby-stores
      附近门店（取 storeCode / beCode）
关键字段约定（与真实 mcd-mcp 对齐）
- orderType：1 = 到店，2 = 外送。
- beType：1 到店自取（不传 beCode）/ 2 麦乐送（必传 beCode+addressId）/ 5 得来速（必传 beCode）/ 6 团餐（必传 beCode+addressId）。
- 到店场景必传 takeWayCode；外送场景必传 addressId + 备注（≤ 50 字）。
- cancelReasonCode：1 改主意了 / 2 重复下单 / 3 点错了 / 4 地址填错 / 5 送达时间错 / -1 其它。
