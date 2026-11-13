# Python CLI Data Analysis Template

面向本地 Python 数据分析应用的 Copier 模板。生成项目只提供两个 CLI 命令：`demo` 和 `http`。

## 快速开始

需要 Copier 9+、uv，以及 Python 3.12–3.14。

```bash
uv tool install copier
copier copy gh:asher-tian/python-cli-data-analysis-template ./my-project
cd ./my-project
uv sync

# 运行完整示例：并行 Group、共享计算、日志、Run 记录、结果持久化和报告
uv run my_project demo

# 查看报告
uv run my_project http
```

配置案例统一放在生成项目的 `examples/` 目录中，包括 `demo.cfg` 和 `app.cfg`。可通过 `--config`、`--app-config` 和 `--workpath` 覆盖默认路径。

`project_name` 默认取目标目录名。例如目标目录为 `./my-project` 时，默认生成 `project_name=my-project`、`project_slug=my-project` 和 `package_name=my_project`。这些值都可以在 Copier 提示中修改。

HTTP 服务按需启动，默认仅监听本机地址 `127.0.0.1:8000`。

## 命令

| 命令 | 用途 |
| --- | --- |
| `demo` | 运行完整示例，验证并行 Group、共享进程池、日志、Run 记录、Dataset 保存和报告生成 |
| `http` | 启动本地报告服务，浏览 Business → Group → Run、Dataset 和二次分析 |

CLI 使用显式命令映射分发 Handler。新增命令时增加独立 Handler 和 Parser 定义，并更新命令映射；主入口不维护 `if/elif` 分支。

## Demo 执行模型

- 多个 Group 由一个 `ThreadPoolExecutor` 并行调度。
- 所有 Group 共用同一个 `SharedCompute`，底层是有界 `ProcessPoolExecutor`；不会为每个 Group 单独创建进程池。
- Core 只提供可序列化的计算输入、纯计算函数和结果聚合；进程池生命周期由 CLI 管理。
- 每次执行创建独立 Run 记录；Run 与 Group 分别写文本日志和 JSONL 结构化日志。
- 结果按 Demo 配置的路径保存为 JSON / JSONL；报告记录 Dataset 引用并由本地 HTTP 服务分页读取。

## 项目结构

```text
src/<package_name>/
├── cli/       # CLI 入口、命令 Handler、并行编排和共享计算资源
├── core/      # 纯业务计算与 Workflow
├── report/    # Dataset 契约、报告生成与前端资源
├── store/     # Workspace、文件操作、日志与 Run 记录
└── http/      # 本地报告服务
```

## 设计约定

- **CLI** 负责命令解析、配置加载、Run 生命周期、Group 并发和计算资源生命周期。
- **Core** 聚焦纯业务计算，不依赖 CLI、HTTP、Report、Store 或具体文件布局。
- **Store** 提供通用路径管理、文件操作、日志和 Run 记录，不规定业务目录结构。
- **Report** 通过 HTTP API 提供数据；大型 Dataset 使用 JSONL 和服务端分页。
- **HTTP** 独立启动，只读取 Run 记录中声明的成功产物，不暴露整个 workpath。

前端使用 Bootstrap、原生 JavaScript ES Modules 和 ECharts。第三方资源随项目本地提供，不依赖 CDN；运行项目不需要 Node.js 或前端构建步骤。

## 开发检查

```bash
uv run ruff check .
uv run ruff format --check .
uv run pytest
```
