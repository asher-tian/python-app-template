# Python App Template

面向本地数据分析应用的 Copier 模板。以 CLI 为主要入口，Core 实现业务，Report 负责业务报告和交互，Store 管理工作空间；需要浏览器动态访问时，通过可选的本地 HTTP Server 提供页面和 API。

## 快速开始

需要 Copier 9+、uv，以及 Python 3.12、3.13 或 3.14。

```bash
uv tool install copier
copier copy gh:asher-tian/python-app-template ./my-app
cd ./my-app
uv sync --all-groups

# 执行业务；两个示例业务都包含 alpha Group，用于验证业务隔离
uv run my_app analysis --config analysis.cfg
uv run my_app comparison --config comparison.cfg

# 启动本地报告中心
uv run my_app http --config app.cfg
```

默认导入包名为 `my_app`。如果生成项目时修改了 `package_name`，运行命令时使用对应的包名。

运行检查：

```bash
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

## 架构

- **CLI**：参数解析、配置加载、命令执行和运行编排。
- **Core**：纯业务模型和业务流程；不依赖 CLI、HTTP、Report 或文件系统布局。
- **Report**：业务专属报告生成、报告数据契约和共享前端资源。
- **Store**：Workspace、路径安全、JSON/文件读写和运行 Manifest。
- **HTTP**：可选的 FastAPI 本地接口，提供报告页面、数据 API 和业务二次分析 API；不承载业务逻辑。

不预建 Runtime、Bootstrap、Ports、Adapters、Task Registry、DAG、Worker 或消息队列。多 Group 的 I/O 型执行由命令层按需使用标准库线程池；CPU 进程池不是模板默认能力。

## Workspace / workpath

`workpath` 是应用工作空间，不是固定的业务目录模板。模板只约定以下公共入口：

- `.app/`：Workspace 元数据、Run Manifest 和日志。
- `shared/`：跨业务共享数据的入口；其内部目录由使用方决定。
- `businesses/`：业务命名空间。每个业务根目录必须提供 `business.json`，其内部目录完全由业务决定。

Business Manifest 声明 Group 及其 Group Manifest 相对路径；Group Manifest 声明运行历史、报告入口和 Dataset 路径。HTTP 通过元数据发现产物，不通过猜测 `reports/`、`data/` 等目录名推断业务含义。

Business ID、Group ID 用于稳定标识和路径隔离；Label 是展示名称。不同业务可以拥有相同 Group Label。Run ID 标识一次执行，不是业务导航层级。

## 扩展原则

新增业务时，增加对应 Core workflow、CLI command 和业务专属 Report builder，并写出公共 Business / Group 元数据。业务可自由决定其内部数据、缓存、报告和中间产物目录，只需在 Group Manifest 中声明对外可访问的报告与 Dataset。

报告前端可执行筛选、排序和轻量统计；复杂或需要 Python 的二次分析通过 HTTP API 调用 Core。HTTP 只负责请求校验、调用和结果返回。默认只监听 loopback 地址，不引入分布式服务。
