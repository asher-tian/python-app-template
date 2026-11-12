# AI 编码助手协作规范

## 项目定位

- 本项目是本地数据分析应用模板，不是通用任务调度框架或 Web 平台框架。
- 顶层模块为 CLI、Core、Report、Store、HTTP；HTTP 是必备的本地报告访问入口，但只在需要查看报告时启动。
- 优先采用普通函数、数据类和标准库能力。只有存在真实替换需求时才引入抽象。
- 不得擅自扩大任务范围；发现额外问题时先报告并说明影响。

## 模块边界

- CLI 负责参数、配置、命令编排、运行生命周期和退出码。
- Core 负责业务模型、业务规则和分析流程；不得依赖 CLI、Pydantic 配置模型、HTTP、Report、Store 或具体路径。
- Report 负责业务专属报告生成、报告数据契约和前端交互；不得复制 Core 业务算法。
- Store 负责 Workspace 路径访问、Run 记录和通用文件操作；不解释业务目录内部含义。
- HTTP 负责页面服务、请求校验和调用业务入口；不得在路由中实现业务算法或直接拼接业务内部目录。
- CLI 和 HTTP 可以调用同一 Core workflow。Core 不反向依赖任何入口。
- 不预建 Runtime、Bootstrap、Ports、Adapters、Repository Framework、Task Registry、Component Registry、DAG Engine、插件系统或分布式 Worker。

## Workspace 与路径

- workpath 是应用工作目录，不是固定的业务目录模板。
- Workspace 只负责根路径、相对路径解析和路径安全；初始化时只确保 workpath 根目录存在。
- 禁止 workspace.json、business.json、Group Manifest 或任何跨 Run 的业务元数据文件。
- 不创建固定的 shared/、businesses/、data/、reports/、cache/ 等业务目录。
- 唯一通用持久化约定是 Run 记录目录；其相对路径由 app.cfg 的 workspace.run_records_dir 配置，默认值为 .app/runs。
- Business 自己在业务配置中声明所需路径；业务内部目录结构完全由业务决定。通用 Store 不解释这些路径的业务含义。
- 所有相对路径必须是 POSIX 相对路径，禁止绝对路径、路径穿越、反斜杠和通过符号链接逃逸 workpath。
- Business ID、Group ID 用于业务标识；Label 仅用于展示。不同 Business 可以有相同 Group ID 或 Label。

## Run 记录

- 每次 CLI 或 HTTP 任务执行创建独立 Run ID，并在配置的 Run 记录目录下写入 run.json。
- run.json 保存命令、Business 信息、开始/结束时间、状态、有效配置、配置来源、运行环境、Group 结果和异常摘要。
- 有效配置必须递归脱敏；密码、Token、密钥和凭据不得明文写入 Run 记录。
- Run 记录在任务开始时创建；Group 完成时增量更新；结束时写入最终状态。
- 每个 Run 只写自己的目录。禁止通过更新共享 Business 或 Group 文件维护 latest 状态。
- latest successful Run 由 Report 从 Run 记录中计算，不额外持久化。

## Core / Report

- Core workflow 使用普通函数或有明确职责的类，不强制建立 Task / Step 层级。
- Core 输入输出使用 dataclass 或简单类型，不使用 Pydantic 作为领域模型。
- Report Dataset 是面向报告的数据契约，不等同于 Core 内部模型。
- 报告按 Business 特性实现；同一 Business 下 Group 是平级单元。
- Business 配置声明的路径由对应业务的 Report builder 使用；不要把业务目录名写进通用 Workspace 或 Store。
- Report 可以扫描和聚合 Run 记录，构建 Business -> Group -> Run 导航；不生成或维护 Business / Group Manifest。
- GroupRunResult.report_kind 标识业务报告类型；公共 Shell 负责导航和通用 Run 信息，业务展示使用明确的业务分支，不建立动态 Registry。
- 每个 Business 的前端展示放在 report/frontend/businesses/<report_kind>.js；共享 report.js 只做显式分发、导航和 Dataset 分页，不集中堆叠业务页面逻辑。
- 大型表格 Dataset 使用 JSONL；DatasetReference 保存 format 和 row_count。HTTP 只返回当前分页，不把完整 Dataset 发送给浏览器。
- 小型统计和结构化结果使用 JSON。分页默认 50 条，最大 200 条。
- 浏览器端只承担展示、筛选和轻量计算；复杂业务分析必须调用 Core。
- 不为了统一外观强制所有业务使用相同页面结构。

## HTTP

- HTTP 是必备的本地报告访问入口，默认只监听 127.0.0.1；CLI 执行分析时不自动启动，用户通过 http 命令按需启动。
- FastAPI 和 Uvicorn 属于基础依赖，不得移动到 optional-dependencies。
- HTTP 通过 Report 从 Run 记录发现报告和 Dataset，不扫描或暴露任意 workpath 文件。
- 不得把整个 workpath 或业务目录直接挂载为静态文件目录。
- 对外文件必须来自成功 Run 的输出记录，并再次进行路径安全校验。
- 路由负责请求校验和调用 Core；业务差异由明确的业务模块实现，不引入动态 Registry。
- 默认同步执行；没有实际需求时不引入 Celery、Kafka、Redis 或 Worker。

## 并发、日志与运行记录

- 多 Group 并发按需使用 ThreadPoolExecutor；不暴露 CPU、MEM 或 Worker 数量配置。
- 不默认创建 ProcessPoolExecutor。确有 CPU 密集需求时，在具体应用中引入共享进程池。
- Run 日志与 run.json 位于同一个 Run 目录；run.log / groups/*.log 保留 DEBUG 文本、源码文件和行号及完整异常堆栈，run.jsonl / groups/*.jsonl 保存结构化事件；Console 仅显示 INFO 及异常摘要，不输出堆栈。
- 不同 Run 之间无共享写入；同一个 Run 内由主线程汇总 Group 结果并原子更新 run.json。

## 配置与工具链

- 全局 app.cfg 管理 workpath、Run 记录路径和 HTTP；各业务配置管理业务参数、Groups 和业务专属路径。
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
