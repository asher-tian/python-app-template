# Python App 模板

基于 Copier 的 Python 应用模板。默认提供 CLI，采用 Workflow 组织业务流程，并将业务逻辑与接口、运行时及基础设施实现分离。

## 快速开始

需要 Copier 9+、uv，以及 Python 3.12、3.13 或 3.14。

~~~bash
uv tool install copier
copier copy gh:asher-tian/python-app-template ./my-app
cd ./my-app

uv sync --all-groups
uv run my_app analysis --config config.json
~~~

默认导入包名为 \`my_app\`。如果生成项目时修改了 \`package_name\`，运行命令时使用对应的包名。

运行检查：

~~~bash
uv run ruff check .
uv run ruff format --check .
uv run pytest
~~~

## 模板结构与架构

~~~text
src/<package_name>/
├── core/
│   ├── workflows/       # 完整业务流程及 Workflow 输入模型
│   ├── tasks/            # Workflow 组合的独立业务单元
│   ├── components/       # 可复用的纯功能模块
│   └── ports/            # Core 所需的外部能力契约
├── interfaces/
│   └── cli/
│       ├── commands/     # CLI 命令适配器
│       ├── configuration/ # 外部配置 Schema、Loader 和 Mapper
│       └── logging_setup.py
├── runtime/
│   ├── workflow_runner.py
│   ├── application_runtime.py
│   └── bootstrap.py      # 运行资源装配
└── infrastructure/
    ├── execution/        # 共享进程池
    ├── logging/          # 日志实现
    ├── storage/          # 文件存储实现
    └── database/         # 可选数据库适配器
~~~

- **Core**：包含 Workflow、Task、业务模型和 Ports，不依赖 CLI、配置格式或具体基础设施。
- **Interfaces**：负责接收外部输入、校验配置，并将输入映射为 Core 的 Workflow Input。
- **Runtime**：创建并管理 WorkflowRunner、共享执行资源及其生命周期。
- **Infrastructure**：实现进程池、日志、存储和数据库等技术能力。

一个 Command 选择一种 Workflow。配置可以提供多个执行输入；每个输入创建独立的 Workflow 实例。Workflow 负责组合自己的 Task，不共享跨执行实例的业务状态。

WorkflowRunner 使用 ThreadPoolExecutor 并发运行多个 Workflow；CPU 密集型计算由所有 Workflow 共享的 ProcessPoolExecutor 执行。Core 通过 Ports 使用外部能力，Infrastructure 提供实现，Runtime/Bootstrap 负责注入和装配。
