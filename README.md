# Python CLI Data Analysis Template

面向本地 Python 数据分析应用的 Copier 模板。提供 CLI 分析命令和本地 HTTP 报告服务。

## 快速开始

需要 Copier 9+、uv，以及 Python 3.12–3.14。

```bash
uv tool install copier
copier copy gh:asher-tian/python-cli-data-analysis-template ./my-project
cd ./my-project
uv sync

# 执行分析
uv run my_project analysis --config examples/analysis.cfg
uv run my_project comparison --config examples/comparison.cfg

# 查看报告
uv run my_project http --config examples/app.cfg
```

配置案例统一放在生成项目的 `examples/` 目录中，包含 `analysis.cfg`、`comparison.cfg` 和 `app.cfg`。

`project_name` 默认取目标目录名。例如目标目录为 `./my-project` 时，默认生成 `project_name=my-project`、`project_slug=my-project` 和 `package_name=my_project`。这些值都可以在 Copier 提示中修改。

浏览器打开 http://127.0.0.1:8000。HTTP 服务按需启动，默认仅监听本机地址。

## 项目结构

```text
src/<package_name>/
├── cli/       # 命令入口与运行编排
├── core/      # 业务模型与分析流程
├── report/    # 报告生成与前端资源
├── store/     # Workspace 与 Run 记录
└── http/      # 本地报告服务
```

## 设计约定

- **CLI** 执行分析任务；HTTP 独立启动，负责浏览报告、历史 Run、Dataset 和二次分析。
- **Core** 聚焦业务逻辑，不依赖具体存储、CLI 或 HTTP 实现。
- **Business** 自主管理分析流程、报告和业务目录；不同 Business 之间保持隔离。
- **Store** 提供通用路径管理、文件操作和 Run 记录，不规定业务目录结构。
- **Report** 通过 HTTP API 提供数据；大型 Dataset 使用服务端分页。

前端使用 Bootstrap、原生 JavaScript ES Modules 和 ECharts。第三方资源随项目本地提供，不依赖 CDN；运行项目不需要 Node.js 或前端构建步骤。

## 开发检查

```bash
uv run ruff check .
uv run pytest
```
