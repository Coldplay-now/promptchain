# 绘图模板集合

本文档提供常见可视化场景的即用模板。每个模板包含 Mermaid 和 SVG 两种实现，方便按需选用。

---

## 模板一：系统架构图

**适用场景：** 展示系统组件及其关系，如微服务架构、前后端分层、部署拓扑。

### Mermaid 实现

```mermaid
block-beta
  columns 3

  space:3

  subgraph "客户端层"
    Web["Web App"]
    Mobile["Mobile App"]
    CLI["CLI Tool"]
  end

  space:3

  subgraph "API 网关层"
    Gateway["API Gateway\nNginx / Kong"]
  end

  space:3

  subgraph "服务层"
    UserSvc["用户服务"]
    OrderSvc["订单服务"]
    PaySvc["支付服务"]
  end

  space:3

  subgraph "数据层"
    DB[("PostgreSQL")]
    Cache[("Redis")]
    MQ[["Kafka"]]
  end

  Web --> Gateway
  Mobile --> Gateway
  CLI --> Gateway
  Gateway --> UserSvc
  Gateway --> OrderSvc
  Gateway --> PaySvc
  UserSvc --> DB
  UserSvc --> Cache
  OrderSvc --> DB
  OrderSvc --> MQ
  PaySvc --> DB
```

### 替代：Mermaid flowchart 实现

```mermaid
flowchart TB
    subgraph Client["客户端"]
        Web["Web App"]
        Mobile["Mobile App"]
    end

    subgraph Gateway["网关层"]
        GW["API Gateway"]
    end

    subgraph Services["服务层"]
        US["用户服务"]
        OS["订单服务"]
        PS["支付服务"]
    end

    subgraph Data["数据层"]
        DB[("PostgreSQL")]
        Cache[("Redis")]
    end

    Client --> GW
    GW --> US & OS & PS
    US & OS & PS --> DB
    US --> Cache

    style Client fill:#e8f4fd,stroke:#4a90d9
    style Gateway fill:#fef3e2,stroke:#f5a623
    style Services fill:#e8f8e8,stroke:#50b86c
    style Data fill:#f3e8fd,stroke:#9b59b6
```

---

## 模板二：业务流程图

**适用场景：** 展示业务操作步骤、审批流程、工作流。

### Mermaid 实现

```mermaid
flowchart TD
    Start([用户提交订单]) --> Check{库存检查}

    Check -->|充足| Lock[锁定库存]
    Check -->|不足| Notify[通知缺货]
    Notify --> End1([流程结束])

    Lock --> Pay{支付}
    Pay -->|成功| Confirm[确认订单]
    Pay -->|失败| Release[释放库存]
    Pay -->|超时| Release

    Release --> End2([流程结束])

    Confirm --> Ship[安排发货]
    Ship --> Track[物流追踪]
    Track --> Receive([确认收货])

    style Start fill:#4a90d9,color:#fff
    style Receive fill:#50b86c,color:#fff
    style End1 fill:#e74c3c,color:#fff
    style End2 fill:#e74c3c,color:#fff
    style Check fill:#f5a623,color:#fff
    style Pay fill:#f5a623,color:#fff
```

---

## 模板三：时序图（API 调用链）

**适用场景：** 展示多个参与者之间的消息传递顺序，如 API 调用链、微服务通信。

### Mermaid 实现

```mermaid
sequenceDiagram
    actor User as 用户
    participant FE as 前端
    participant GW as API 网关
    participant Auth as 认证服务
    participant Biz as 业务服务
    participant DB as 数据库

    User->>FE: 点击登录
    FE->>GW: POST /api/login
    GW->>Auth: 验证凭据
    Auth->>DB: 查询用户信息
    DB-->>Auth: 用户数据

    alt 验证成功
        Auth-->>GW: Token
        GW-->>FE: 200 + Token
        FE->>GW: GET /api/dashboard (带 Token)
        GW->>Auth: 校验 Token
        Auth-->>GW: 有效
        GW->>Biz: 获取仪表盘数据
        Biz->>DB: 查询
        DB-->>Biz: 结果
        Biz-->>GW: 数据
        GW-->>FE: 200 + 数据
        FE-->>User: 显示仪表盘
    else 验证失败
        Auth-->>GW: 401
        GW-->>FE: 401 Unauthorized
        FE-->>User: 显示错误提示
    end
```

---

## 模板四：数据模型（ER 图）

**适用场景：** 展示数据库表结构和表间关系。

### Mermaid 实现

```mermaid
erDiagram
    USER {
        int id PK
        string name
        string email UK
        datetime created_at
    }

    ORDER {
        int id PK
        int user_id FK
        decimal total_amount
        string status
        datetime created_at
    }

    ORDER_ITEM {
        int id PK
        int order_id FK
        int product_id FK
        int quantity
        decimal price
    }

    PRODUCT {
        int id PK
        string name
        decimal price
        int stock
        int category_id FK
    }

    CATEGORY {
        int id PK
        string name
        int parent_id FK
    }

    USER ||--o{ ORDER : "下单"
    ORDER ||--|{ ORDER_ITEM : "包含"
    PRODUCT ||--o{ ORDER_ITEM : "被购买"
    CATEGORY ||--o{ PRODUCT : "归类"
    CATEGORY ||--o{ CATEGORY : "子分类"
```

---

## 模板五：状态机

**适用场景：** 展示对象的生命周期状态转换。

### Mermaid 实现

```mermaid
stateDiagram-v2
    [*] --> Draft: 创建

    Draft --> Pending: 提交审核
    Draft --> Cancelled: 取消

    Pending --> Approved: 审批通过
    Pending --> Rejected: 审批驳回
    Pending --> Draft: 退回修改

    Approved --> Active: 激活
    Rejected --> Draft: 重新编辑
    Rejected --> Cancelled: 放弃

    Active --> Suspended: 暂停
    Active --> Completed: 完成
    Active --> Cancelled: 终止

    Suspended --> Active: 恢复

    Completed --> [*]
    Cancelled --> [*]

    note right of Pending: 等待审批人处理\n超时自动提醒
    note right of Active: 业务运行中
```

---

## 模板六：甘特图（项目排期）

**适用场景：** 展示项目时间线、任务排期、迭代计划。

### Mermaid 实现

```mermaid
gantt
    title Q1 产品迭代计划
    dateFormat YYYY-MM-DD
    axisFormat %m/%d

    section 需求分析
        用户调研           :done, research, 2025-01-06, 5d
        需求文档           :done, prd, after research, 3d

    section 设计
        UI 设计            :active, design, after prd, 7d
        技术方案           :active, tech, after prd, 5d

    section 开发
        用户模块           :dev1, after tech, 10d
        订单模块           :dev2, after dev1, 8d
        支付集成           :dev3, after dev1, 12d

    section 测试
        集成测试           :test, after dev2, 5d
        性能测试           :perf, after dev3, 3d

    section 发布
        灰度发布           :crit, release, after test, 3d
        全量上线           :crit, launch, after release, 1d
```

---

## 模板七：思维导图

**适用场景：** 知识梳理、头脑风暴、层级概念展示。

### Mermaid 实现

```mermaid
mindmap
  root((前端工程化))
    构建工具
      Vite
      Webpack
      Turbopack
    代码质量
      ESLint
      Prettier
      TypeScript
    测试
      单元测试
        Jest
        Vitest
      E2E 测试
        Playwright
        Cypress
    CI/CD
      GitHub Actions
      GitLab CI
    包管理
      pnpm
      npm
      yarn
```

---

## 模板八：对比图

**适用场景：** 两个或多个方案/技术的多维度对比。

### HTML 实现

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<style>
  body { font-family: -apple-system, sans-serif; padding: 2rem; background: #f8f9fa; }
  .table-wrapper { overflow-x: auto; }
  table { border-collapse: collapse; width: 100%; background: white; border-radius: 8px; overflow: hidden; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
  th, td { padding: 12px 16px; text-align: left; border-bottom: 1px solid #eee; }
  th { background: #2c3e50; color: white; font-weight: 600; }
  th:first-child { background: #34495e; }
  tr:hover { background: #f5f8ff; }
  .good { color: #27ae60; font-weight: 600; }
  .mid { color: #f39c12; font-weight: 600; }
  .bad { color: #e74c3c; font-weight: 600; }
</style>
</head>
<body>
  <div class="table-wrapper">
    <table>
      <tr>
        <th>维度</th><th>方案 A</th><th>方案 B</th><th>方案 C</th>
      </tr>
      <tr>
        <td><strong>性能</strong></td>
        <td class="good">优秀</td>
        <td class="mid">一般</td>
        <td class="good">优秀</td>
      </tr>
      <tr>
        <td><strong>易用性</strong></td>
        <td class="mid">中等</td>
        <td class="good">简单</td>
        <td class="bad">复杂</td>
      </tr>
      <tr>
        <td><strong>维护成本</strong></td>
        <td class="good">低</td>
        <td class="good">低</td>
        <td class="bad">高</td>
      </tr>
    </table>
  </div>
</body>
</html>
```

---

## 模板九：时间线

**适用场景：** 版本演进、历史事件、里程碑。

### Mermaid 实现

```mermaid
timeline
    title 产品发展历程
    2023 Q1 : 项目立项
             : 组建核心团队
    2023 Q2 : MVP 发布
             : 首批 100 用户
    2023 Q3 : 完成 A 轮融资
             : 团队扩张至 20 人
    2023 Q4 : 2.0 版本发布
             : 用户突破 10,000
    2024 Q1 : 国际化版本
             : 进入东南亚市场
    2024 Q2 : 开放 API 生态
             : 第三方插件市场上线
```

---

## 模板十：象限图（优先级矩阵）

**适用场景：** 二维评估、优先级排序、技术选型。

### Mermaid 实现

```mermaid
quadrantChart
    title 技术债优先级矩阵
    x-axis "低影响" --> "高影响"
    y-axis "低成本" --> "高成本"

    "升级依赖版本": [0.3, 0.2]
    "统一错误处理": [0.7, 0.3]
    "重构数据层": [0.8, 0.8]
    "抽取公共组件": [0.5, 0.4]
    "迁移到 TypeScript": [0.6, 0.9]
    "优化数据库索引": [0.9, 0.2]
    "清理废弃 API": [0.4, 0.1]
    "重写认证模块": [0.85, 0.7]
```

---

## 使用建议

1. **直接复制修改**：这些模板可以直接复制，替换为你的实际内容
2. **混合使用**：同一个项目可能需要多种图表类型，按需组合
3. **保持简洁**：模板展示了较完整的结构，实际使用时可以按需精简
4. **样式统一**：如果在同一文档中使用多张图，保持配色和风格一致
