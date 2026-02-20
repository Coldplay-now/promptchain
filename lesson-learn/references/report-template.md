# 报告模板

本文档提供 LessonLearn 报告的完整模板和不同场景的输出示例。

---

## 完整报告模板

以下是报告文件的完整结构模板，AI 在生成报告时应遵循此格式：

```markdown
---
project: my-project
created: 2025-01-15
last_updated: 2025-01-15
total_reviews: 1
---

# my-project 经验教训报告

## 目录
- [第 1 次复盘 - 2025-01-15 - 架构设计与测试策略](#review-1)

---

## 第 1 次复盘 - 2025-01-15 {#review-1}

### 项目快照

| 项 | 值 |
|----|-----|
| 技术栈 | React + TypeScript + Node.js |
| 代码规模 | 约 15,000 行，42 个源文件 |
| 项目阶段 | 迭代期（v2.0 功能开发中） |
| 聚焦维度 | 架构设计、测试策略 |

### 做得好的

#### 1. API 层的错误处理统一且规范

**发现：** `src/api/client.ts` 中实现了统一的错误拦截器，所有 API 调用的错误处理收口在一处。

**价值：** 新增接口时不需要重复写 try-catch，错误上报和用户提示风格一致。

**相关文件：** `src/api/client.ts:23-45`

#### 2. 数据库迁移脚本管理规范

**发现：** 每次 schema 变更都有对应的迁移脚本，且包含 up/down 两个方向，文件名用时间戳排序。

**价值：** 可以可靠地回滚数据库变更，团队成员拉取代码后一键同步数据库。

**相关文件：** `db/migrations/`

### 需要改进的

#### 1. OrderService 承担了过多职责

**发现：** `src/services/order.ts` 已达到 820 行，包含了订单创建、支付对接、库存扣减、通知发送四个不同关注点。

**根因：** 早期为了快速上线把逻辑都堆在一个服务中，后续功能迭代一直在这个文件上追加。

**影响：**
- 每次修改支付逻辑都有可能影响库存扣减
- 多人同时修改时频繁冲突（Git 历史显示该文件最近 30 天被修改了 18 次）
- 单元测试难以编写，因为 mock 的依赖太多

**建议：** 按关注点拆分为 4 个独立服务：
1. `OrderService` — 仅负责订单 CRUD 和状态流转
2. `PaymentService` — 支付对接逻辑
3. `InventoryService` — 库存扣减和预留
4. `NotificationService` — 通知发送

先从耦合最松的 `NotificationService` 开始拆，风险最低。

**相关文件：** `src/services/order.ts`

#### 2. 前端组件缺乏加载和错误状态处理

**发现：** 翻阅 `src/components/` 下的 15 个页面组件，只有 3 个处理了加载状态，0 个处理了请求失败的情况。

**根因：** 没有建立统一的异步状态管理模式，每个组件各自为战。

**影响：**
- 网络慢时页面空白无反馈
- 请求失败时用户看到的是白屏而非错误提示

**建议：**
1. 创建一个 `useAsync` hook 封装加载/成功/失败三种状态
2. 创建 `<AsyncBoundary>` 组件统一处理 loading 和 error UI
3. 从访问量最高的 3 个页面开始改造

**相关文件：** `src/components/Dashboard.tsx`, `src/components/OrderList.tsx`

### 行动项

- [ ] 将 NotificationService 从 OrderService 中拆分出来
- [ ] 创建 useAsync hook 和 AsyncBoundary 组件
- [ ] 给 Dashboard 和 OrderList 页面添加加载和错误状态
- [ ] 为拆分后的 OrderService 补充单元测试
```

---

## 增量追加示例

当用户第二次执行复盘时，在现有报告基础上追加：

### 更新元数据

```markdown
---
project: my-project
created: 2025-01-15
last_updated: 2025-02-10
total_reviews: 2
---
```

### 更新目录

```markdown
## 目录
- [第 1 次复盘 - 2025-01-15 - 架构设计与测试策略](#review-1)
- [第 2 次复盘 - 2025-02-10 - 依赖管理与 Git 规范](#review-2)
```

### 在文件末尾追加

```markdown
---

## 第 2 次复盘 - 2025-02-10 {#review-2}

### 项目快照

| 项 | 值 |
|----|-----|
| 技术栈 | React + TypeScript + Node.js |
| 代码规模 | 约 18,000 行，51 个源文件 |
| 项目阶段 | 迭代期（v2.1 发布后） |
| 聚焦维度 | 依赖管理、Git 规范 |

### 上次行动项回顾

- [x] 将 NotificationService 从 OrderService 中拆分出来（已完成）
- [x] 创建 useAsync hook 和 AsyncBoundary 组件（已完成）
- [ ] 给 Dashboard 和 OrderList 页面添加加载和错误状态（进行中，Dashboard 已完成）
- [ ] 为拆分后的 OrderService 补充单元测试（未开始）

### 做得好的

#### 1. NotificationService 的拆分干净利落

**发现：** 上次复盘提出的 NotificationService 拆分已完成，新的 `src/services/notification.ts` 仅 120 行，职责清晰。拆分过程没有引入任何回归 bug。

**价值：** 验证了"从耦合最松的模块开始拆"的策略是正确的，为后续拆分 PaymentService 积累了信心。

**相关文件：** `src/services/notification.ts`

### 需要改进的

#### 1. 依赖锁文件不一致

**发现：** 项目中同时存在 `package-lock.json` 和 `yarn.lock`，两个文件都有 Git 追踪。`package-lock.json` 上次更新在 3 个月前，已严重过时。

**根因：** 团队中有人用 npm，有人用 yarn，没有统一约定。

**影响：** 不同开发者安装到的依赖版本可能不同，导致"在我机器上能跑"的问题。

**建议：**
1. 团队统一使用一个包管理器（建议 yarn，因为 yarn.lock 更新更频繁，说明更多人在用）
2. 删除 `package-lock.json`，在 `.gitignore` 中忽略它
3. 在 `package.json` 中添加 `"packageManager"` 字段锁定版本

**相关文件：** `package-lock.json`, `yarn.lock`

### 行动项

- [ ] 统一包管理器为 yarn，删除 package-lock.json
- [ ] 在 package.json 中添加 packageManager 字段
- [ ] 配置 CI 检查锁文件一致性
- [ ] 继续完成 OrderList 页面的异步状态改造
- [ ] 启动 PaymentService 的拆分工作
```

---

## 不同规模的报告示例

### 快速复盘（小型项目）

快速复盘的报告更简洁，聚焦 1-2 个维度，每个维度 1-2 条教训：

```markdown
## 第 1 次复盘 - 2025-03-20 {#review-1}

### 项目快照

| 项 | 值 |
|----|-----|
| 技术栈 | Python + FastAPI |
| 代码规模 | 约 2,000 行，8 个源文件 |
| 项目阶段 | 初创期（MVP 开发中） |
| 聚焦维度 | 代码组织 |

### 做得好的

#### 1. 路由和业务逻辑分离清晰

**发现：** `routes/` 目录只做请求解析和响应包装，业务逻辑全部在 `services/` 中。
**价值：** 替换 Web 框架时只需改 routes 层，核心逻辑不受影响。
**相关文件：** `routes/`, `services/`

### 需要改进的

#### 1. 所有配置硬编码在代码中

**发现：** 数据库连接串、API 密钥、端口号散落在 3 个不同文件中。
**根因：** MVP 阶段为了快，没抽取配置。
**建议：** 创建 `config.py`，用 pydantic-settings 从环境变量加载配置，现在做成本很低，越晚改越多文件要动。
**相关文件：** `main.py:12`, `services/db.py:5`, `services/email.py:8`

### 行动项

- [ ] 创建 config.py，将所有配置集中管理
- [ ] 添加 .env.example 文件方便新人上手
```

### 深度复盘（大型项目）

深度复盘会增加可视化辅助，行动项按优先级分层：

```markdown
### 模块依赖关系

以下是当前模块间的依赖关系，箭头指向被依赖方：

​```mermaid
flowchart TD
    API["API Layer"] --> Auth["Auth Service"]
    API --> Order["Order Service"]
    API --> User["User Service"]
    Order --> Payment["Payment Service"]
    Order --> Inventory["Inventory Service"]
    Order --> Notification["Notification Service"]
    Order --> User
    Payment --> User
    Inventory --> Cache["Cache Layer"]

    style Order fill:#e74c3c,stroke:#c0392b,color:#fff
    style API fill:#4a90d9,stroke:#2c5f8a,color:#fff
​```

> Order Service 是当前的风险中心，被 4 个模块依赖，同时依赖 3 个模块。

### 行动项

**短期（本迭代）：**
- [ ] 拆分 NotificationService
- [ ] 修复 3 个已知的 N+1 查询

**中期（下 2-3 个迭代）：**
- [ ] 拆分 PaymentService 和 InventoryService
- [ ] 建立统一的异步状态管理模式

**长期（季度规划）：**
- [ ] 引入集成测试覆盖核心业务流程
- [ ] 评估是否需要引入事件驱动架构解耦服务间通信
```

---

## 报告命名与存储

### 默认路径

```
docs/lessons-learned.md
```

### 自定义路径

如果用户指定了路径，按用户要求存储。常见的替代位置：

```
.notes/lessons-learned.md
LESSONS.md
docs/review/lessons-learned.md
```

### 文件命名

单文件追加模式（默认）：所有复盘记录在一个文件中，通过目录导航。
