# AI 编码助手协作规范

## 架构边界

- Command 是外部接口适配器，只负责解析输入、调用配置映射并启动 WorkflowRunner。
- 每个 Workflow 表示一个完整、独立的业务闭环；多个输入会创建多个相互隔离的 Workflow 实例。
- Workflow 根据自己的输入决定执行哪些 Task，并负责 Task 之间的业务编排。
- Task 是独立业务单元。不要强制所有 Task 继承 BaseTask。
- 不建立正式 Step 抽象。Task 内部简单操作使用普通函数和模块。
- 可复用功能使用普通函数或组件模块，不建立 Component Registry。
- 不建立 Task Registry、DAG Engine、分布式调度或通用插件框架。
- Core 不依赖 CLI、HTTP、Pydantic、配置文件格式或具体基础设施 SDK。
- Interface Schema 负责外部输入校验和默认值；Mapper 将其转换为 Core Workflow Input。
- 依赖方向：Interfaces -> Core；Infrastructure 实现 Core Ports；Runtime/Bootstrap 负责装配。

## 并发与资源

- WorkflowRunner 固定使用 ThreadPoolExecutor，负责多个 Workflow 实例的并发。
- ComputeExecutor 使用一个共享 ProcessPoolExecutor，负责 CPU 密集计算。
- Runtime 创建并关闭两种执行资源；Workflow 和 Task 不得自行创建线程池或进程池。
- ProcessPool worker 必须是模块顶层函数；只传递可序列化函数参数和结果，不传递 Workflow、Task、Logger、数据库连接、Executor 或 Runtime 对象。
- Worker 默认只执行计算并返回结果；业务日志由主进程记录。
- Workflow 实例及其 Task 实例不得跨执行实例共享可变状态。
- 不提供 CPU、MEM、worker 数量等用户资源配置。

## 日志与运行记录

- Logging 是 Runtime 管理的横切基础设施；Core 只使用注入的 Logger。
- CLI 使用 Run Directory、JSONL 日志和 manifest.json。
- Run 与 Workflow Execution 使用独立日志文件；Task 日志写入所属 Workflow 日志。
- 业务字段（例如 label）只能作为可选 metadata，不能成为通用 Runtime 必需字段。
- Console 不输出异常堆栈；File JSONL 保存完整异常信息。
- 不允许多个进程直接写同一个日志文件。

## 配置与扩展

- 每个 Workflow 定义自己的业务输入模型和 Options。
- CLI 为实际暴露的 Workflow 定义对应 Config Schema、Loader 和 Mapper。
- Task 仅在参数复杂且有明确业务意义时定义自己的参数模型；不要求独立外部 Schema。
- 未来增加 Web 时，新增 Web Request Schema 和 Mapper，映射到同一个 Workflow Input；当前不预建 Web 实现。
- Database、Kafka、Storage 等按需通过 Port 和 Infrastructure Adapter 集成，不为未使用的组件引入依赖。

## 工具链

- 使用 uv 管理 Python 环境和依赖。
- 保持 src 布局；运行依赖放在 project.dependencies，开发依赖放在 dependency-groups.dev。
- 修改 Python 版本时同步检查 .python-version、pyproject.toml、Dev Container 和 CI。
- 仅在确有必要时增加依赖，并说明原因。

## 常用检查

    uv sync --all-groups
    uv run ruff check .
    uv run ruff format --check .
    uv run pytest

## 安全与完成标准

- Manifest 保存完整有效配置，但必须递归脱敏密码、Token、密钥等敏感字段。
- 不提交 .env、令牌、密码、私钥或个人认证缓存。
- 未实际运行的检查不得报告为通过。
