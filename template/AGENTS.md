# AI 编码助手协作规范

## 架构边界

- `core/` 包含业务模型、Use Case、Task、Step 和 Ports；不得依赖 CLI、Web 框架或具体基础设施 SDK。
- `interfaces/` 是输入/输出适配器。CLI、Web API、SDK 调用同一 Application API，不在入口层实现业务流程。
- `infrastructure/` 实现 Core Ports，例如 Compute、Database、Storage、Logging 和消息集成。
- `runtime/` 与 Bootstrap 负责装配具体实现、调度策略和资源生命周期。
- 依赖方向必须指向 Core：Interfaces -> Core；Infrastructure -> Core Ports；Bootstrap -> 各层。Core 不依赖外层。
- 不要为了未来可能性预先引入 Kafka、Redis、Web 框架或分布式调度器。

## Task、Step 与并发

- Task 是一个完整业务工作单元；Step 是 Task 内部步骤。Command 只负责解析入口参数、转换请求和调用 Application。
- TaskScheduler 管理 Task 并发；共享 Compute Provider 管理 CPU 计算。Task/Step 不得自行创建全局线程池或进程池。
- ProcessPool worker 必须是模块顶层函数；只传递可序列化 DTO/基础数据，不传递 Task、Logger、数据库连接、Kafka 客户端、Executor 或 Runtime 对象。
- 线程共享对象必须有明确线程安全约定。SQLite Connection 不跨线程/进程共享；按操作创建连接，保持事务短小。
- 外部子进程也消耗 CPU 和内存，资源规划不能只计算 Python worker 数量。
- 长任务使用 run_id/task_id 关联状态、日志和产物；分布式任务队列仅在确有需要时引入。

## 工具链

- 使用 uv 管理 Python 解释器、虚拟环境和依赖；不要将项目依赖安装到全局 Python。
- 保持 src 布局；运行依赖放在 project.dependencies，开发依赖放在 dependency-groups.dev。
- 修改 Python 版本时同步检查 .python-version、pyproject.toml、Dev Container 和 CI。
- 仅在确有必要时增加依赖，并说明原因。

## 常用检查

```bash
uv sync --all-groups
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

## 安全与完成标准

- 不提交 .env、令牌、密码、私钥或个人认证缓存。
- 不挂载整个宿主机用户目录。
- 未实际运行的检查不得报告为通过。
