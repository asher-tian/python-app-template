# Python CLI Data Analysis Template

面向本地 Python CLI 数据分析应用的 Copier 模板。CLI 负责执行分析任务，HTTP 提供统一的本地报告访问入口；Core 实现业务流程，Report 负责业务报告，Store 负责 Workspace 路径访问和 Run 记录。

HTTP 是模板的必备能力。执行分析任务时不会自动启动服务；需要查看报告、浏览历史 Run、访问 Dataset 或执行二次分析时，再启动本地 HTTP 服务。

## 快速开始

需要 Copier 9+、uv，以及 Python 3.12、3.13 或 3.14。

    uv tool install copier
    copier copy gh:asher-tian/python-cli-data-analysis-template ./my-app
    cd ./my-app
    uv sync --all-groups

    uv run my_app analysis --config analysis.cfg
    uv run my_app comparison --config comparison.cfg
    uv run my_app http --config app.cfg

默认命令名为 my_app。修改 Copier 参数 package_name 后，使用对应的命令名。

浏览器默认访问 http://127.0.0.1:8000。HTTP 服务默认只监听本机地址。

运行检查：

    uv run ruff check .
    uv run ruff format --check .
    uv run pytest

## 模板结构与架构

    src/<package_name>/
    ├── cli/       # CLI 入口、配置、命令编排和日志
    ├── core/      # 纯业务模型与 workflow
    ├── report/    # 业务报告、Dataset 契约和前端资源
    ├── store/     # Workspace 路径、安全文件操作和 Run 记录
    └── http/      # 必备的 FastAPI 本地报告服务

- **CLI** 负责参数解析、配置加载、Run 生命周期和 Group 并发；多 Group 按需使用 ThreadPoolExecutor。
- **Core** 只包含业务规则和流程，不依赖 CLI、HTTP、Report、Store 或具体文件布局。
- **Report** 按 Business 实现报告和数据输出，从 Run 记录聚合 Business → Group → Run 导航；latest successful Run 动态计算，不写共享索引。
- **Store** 只负责 workpath、相对路径安全解析、通用文件操作和 Run 记录，不创建固定业务目录。
- **HTTP** 是必备的本地报告访问入口，默认只监听 127.0.0.1；提供报告导航、历史 Run、Dataset 访问和业务二次分析，不直接暴露整个 workpath。HTTP 服务按需启动，不需要随每次 CLI 分析任务自动运行。

workpath 是应用工作目录。全局配置 app.cfg 的 workspace.run_records_dir 指定唯一通用 Run 记录目录，默认是 .app/runs。每次执行创建独立目录，保存 run.json，以及 logs/run.log、logs/run.jsonl 和各 Group 对应的日志文件。run.json 保存有效配置、配置来源、运行环境、Group 结果、开始结束时间和异常摘要；敏感配置字段会递归脱敏。

每个 Business 在自己的 TOML 配置中通过 paths 声明数据、报告和中间产物目录。业务内部目录结构由 Business 自行决定；不同 Business 可以使用相同 Group ID 或 Label。模板不建立 workspace.json、business.json、Group Manifest、Runtime、Bootstrap、通用 DAG、Task Registry 或分布式 Worker。
