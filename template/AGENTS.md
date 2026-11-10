# AI 编码助手协作规范

## 项目定位

- 本项目是本地数据分析应用模板，不是通用任务调度框架或 Web 平台框架。
- 顶层模块为 CLI、Core、Report、Store；HTTP 是可选的本地接口入口。
- 优先采用普通函数、数据类和标准库能力。只有存在真实替换需求时才引入抽象。
- 不得擅自扩大任务范围；发现额外问题时先报告并说明影响。

## 模块边界

- CLI 负责参数、配置、命令编排、运行生命周期和退出码。
- Core 负责业务模型、业务规则和分析流程；不得依赖 CLI、Pydantic 配置模型、HTTP、Report、Store 或具体路径。
- Report 负责业务报告生成、Report Dataset 契约和前端交互；不得复制 Core 业务算法。
- Store 负责 Workspace、路径解析、安全文件访问和持久化基础操作；不解释业务目录内部含义。
- HTTP 负责页面服务、请求校验和调用业务入口；不得在路由中实现业务算法或直接拼接业务内部目录。
- CLI 和 HTTP 可以调用同一 Core workflow。Core 不反向依赖任何入口。
- 不预建 Runtime、Bootstrap、Ports、Adapters、Repository Framework、Task Registry、Component Registry、DAG Engine、插件系统或分布式 Worker。

## Workspace 规范

- `workpath` 是应用工作空间。
- 模板统一管理 `.app/`、`shared/`、`businesses/` 三个公共入口。
- `.app/` 保存 workspace 元数据、Run Manifest 和日志；`shared/` 是跨业务共享区域；`businesses/` 是业务命名空间。
- 每个 Business 根目录必须有 `business.json`。其中声明 Business ID、展示信息、Group ID / Label 以及 Group Manifest 相对路径。
- Group Manifest 声明 Group 的运行历史、latest successful Run、报告入口和 Dataset。
- Business 内部目录结构由业务决定。不得在公共 Workspace API、HTTP 路由或通用 Store 中写死 `data/`、`reports/`、`cache/` 等业务目录名称。
- 所有 Manifest 中的产物路径均为相对于对应 Business 根目录的 POSIX 相对路径。必须通过安全路径解析函数访问。
- 禁止绝对路径、路径穿越、反斜杠路径和通过符号链接逃逸 Business 根目录。
- Business ID、Group ID、Run ID 是稳定标识；Label 是展示名称。不同 Business 可以有相同 Label。
- 同一个 Business 的元数据更新必须原子写入。多个进程同时写同一个 Business 时，必须明确并发策略，不得假设原子替换等于事务锁。

## Core / Report

- Core workflow 使用普通函数或有明确职责的类，不强制建立 Task / Step 层级。
- Core 输入输出使用 dataclass 或简单类型，不使用 Pydantic 作为领域模型。
- Report Dataset 是面向报告的数据契约，不等同于 Core 内部模型。
- 报告按 Business 特性实现；同一 Business 下 Group 是平级单元。
- 报告页面、数据文件和中间产物的物理布局由 Business 决定，并在 Group Manifest 中声明。
- 浏览器端只承担展示、筛选和轻量计算；复杂业务分析必须调用 Core。
- 不为了统一外观强制所有业务使用相同页面结构。

## HTTP

- HTTP 是可选的本地服务，默认只监听 127.0.0.1。
- HTTP 通过 Business / Group Manifest 发现业务、报告和 Dataset，不扫描或暴露任意业务文件。
- 不得把整个 workpath 或 Business 根目录直接挂载为静态文件目录。
- 对外文件只允许通过 Manifest 声明的相对路径访问，并再次进行路径安全校验。
- 路由负责请求校验和调用 Core；业务差异由明确的业务模块实现，不引入动态 Registry。
- 默认同步执行；没有实际需求时不引入 Celery、Kafka、Redis 或 Worker。

## 并发、日志与运行记录

- 多 Group 并发按需使用 ThreadPoolExecutor；不暴露 CPU、MEM 或 Worker 数量配置。
- 不默认创建 ProcessPoolExecutor。确有 CPU 密集需求时，在具体应用中引入共享进程池。
- Run ID 标识一次命令执行；Run Manifest 和日志位于 `.app/runs/{run_id}/`。
- 运行日志保留 JSONL；Console 显示摘要，不输出异常堆栈；文件日志保留异常堆栈。
- Manifest 保存有效配置、环境、Business / Group 结果和异常摘要；密码、Token、密钥等字段必须递归脱敏。
- 失败的 Run 不得覆盖 Group 的 latest successful Run。

## 配置与工具链

- 全局应用配置与业务配置分离：`app.cfg` 管理 workpath 和 HTTP；各业务配置管理业务参数与 Groups。
- 配置文件使用 TOML；外部输入由 CLI / HTTP 边界校验后映射到 Core 输入。
- 使用 uv 管理环境；保持 src 布局。
- 仅在确有必要时增加依赖，并说明原因。
- 修改 Python 版本时同步检查 Copier、pyproject、Dev Container 和 CI。

## 完成标准

- 修改前检查相关模块和测试，说明影响范围。
- 中大型任务按 Plan / Implement / Review 分阶段；小任务可合并。
- 新增行为必须有测试；至少运行 Ruff、格式检查和 pytest。
- Copier 生成项目必须能独立安装、运行 CLI、启动 HTTP 测试客户端。
- 不得将未实际执行的检查报告为通过。
