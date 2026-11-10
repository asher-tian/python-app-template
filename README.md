# Python App Template

面向本地数据分析应用的 Copier 模板。以 CLI 为主要入口，Core 实现业务，Report 负责业务报告和交互，Store 负责 Workspace 路径访问与 Run 记录；需要浏览器动态访问时，通过可选的本地 HTTP Server 提供页面和 API。

## 快速开始

需要 Copier 9+、uv，以及 Python 3.12、3.13 或 3.14。

    uv tool install copier
    copier copy gh:asher-tian/python-app-template ./my-app
    cd ./my-app
    uv sync --all-groups

    uv run my_app analysis --config analysis.cfg
    uv run my_app comparison --config comparison.cfg
    uv run my_app http --config app.cfg

默认导入包名为 my_app。如果生成项目时修改了 package_name，运行命令时使用对应的包名。

运行检查：

    uv run ruff check .
    uv run ruff format --check .
    uv run pytest

## 架构

- CLI：参数解析、配置加载、命令执行和运行编排。
- Core：纯业务模型和业务流程；不依赖 CLI、HTTP、Report 或文件系统布局。
- Report：业务专属报告生成、报告数据契约和共享前端资源。
- Store：Workspace 路径访问、Run 记录和通用文件操作。
- HTTP：可选 FastAPI 本地接口，提供报告页面、Dataset API 和业务二次分析 API；不承载业务算法。

不预建 Runtime、Bootstrap、Ports、Adapters、Task Registry、DAG、Worker 或消息队列。多 Group 的 I/O 型执行由命令层按需使用标准库线程池；CPU 进程池不是模板默认能力。

## Workspace / workpath

workpath 是应用工作目录。Workspace 不保存自身 Manifest，不创建 workspace.json，也不强制创建 shared、businesses、data、reports 或 cache 等目录。

全局配置 app.cfg 中的 workspace.run_records_dir 指定唯一的通用 Run 记录目录，默认是 .app/runs。每次 CLI 或 HTTP 任务在该目录下创建独立 Run 目录，包含 run.json 和日志。

run.json 保存当次命令、Business 信息、完整有效配置、配置来源、Python / OS 环境、Group 执行结果、开始结束时间和异常摘要。敏感配置字段递归脱敏。每个 Run 只写自己的目录，不需要共享 Business / Group 元数据，也不需要 latest 指针文件。

## 业务路径与报告发现

每个业务配置通过 paths 声明自己的数据、报告和中间产物目录。路径相对于 workpath，内部结构完全由业务决定。通用 Workspace / Store 不理解业务目录含义。

Report 从 Run 记录聚合 Business -> Group -> Run 导航，latest successful Run 由历史 Run 计算得出。报告页面和 Dataset 的实际路径记录在对应 Run 的 Group 结果中；HTTP 只服务成功 Run 明确声明的产物，并进行路径安全校验。

报告按业务特性实现。新增业务时增加对应 Core workflow、CLI command、业务专属 Report builder，并把业务路径写入该业务配置；不需要增加 Business / Group Manifest。

报告前端可以执行筛选、排序和轻量统计；复杂或需要 Python 的二次分析通过 HTTP API 调用 Core。HTTP 默认只监听 loopback 地址，不引入分布式服务。
