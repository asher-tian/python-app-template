# Python App Template

A reusable Copier template for maintainable Python applications.

The generated project starts with a CLI interface and keeps Workflow business logic independent from the input mechanism. Future interfaces such as Web APIs can map their requests to the same Workflow input models without changing Core.

## Quick start

~~~bash
copier copy gh:asher-tian/python-app-template ./my-app
cd my-app
uv sync --all-groups
uv run pytest
uv run ruff check .
uv run ruff format --check .
~~~

Copier downloads this template directly from GitHub. Cloning the dev-templates catalog is not required.

## Architecture

- Core contains Workflows, Tasks, functional components, business input/result models, and Ports.
- Interfaces contains delivery adapters. The initial template provides CLI commands and CLI-specific configuration schemas.
- Runtime owns WorkflowRunner and shared execution resource lifecycles.
- Infrastructure provides concrete ProcessPool, logging, filesystem storage, and optional database adapters.

One Command selects one Workflow type. Its configuration may contain multiple inputs; each input creates a fresh, independent Workflow instance. WorkflowRunner uses ThreadPoolExecutor for Workflow-level concurrency. All Workflows share one ProcessPoolExecutor for CPU-intensive functions.

There is no Task Registry, Component Registry, Step framework, DAG engine, distributed scheduler, or CPU/MEM resource configuration.

## Configuration

Each exposed Workflow has an external CLI Config Schema, Loader, and Mapper. The Mapper converts validated input into a Core Workflow Input. Task-specific option models are added only when a Task genuinely needs them.

A future Web interface may define a separate Request Schema and Mapper. Both interfaces can map to the same Workflow Input. The template does not pre-create Web code.

## Logging

CLI runs create a Run Directory under the configured workpath:

~~~text
logs/
└── analysis/
    └── YYYY-MM-DD/
        └── HH-MM-SS-mmm_runid/
            ├── run.jsonl
            ├── manifest.json
            └── workflows/
                ├── analysis_0001.jsonl
                └── analysis_0002.jsonl
~~~

Run logs contain command-level events and Workflow summaries. Each Workflow Execution has one detailed JSONL file; its Task logs are written to that file. Console output is human-readable and omits tracebacks. JSONL files preserve structured exception details, including the full stack trace.

The manifest records the effective configuration with sensitive values redacted, application metadata, runtime environment, execution statuses, and log paths.

## Template development

The repository CI generates a fresh project with Copier, installs dependencies, and runs Ruff and pytest.

To validate locally:

~~~bash
copier copy --defaults \
  --data "project_name=Template Validation" \
  --data "project_slug=template-validation" \
  --data "package_name=template_validation" \
  --data "python_version=3.13" \
  . ./generated

cd generated
uv sync --all-groups
uv run ruff check .
uv run ruff format --check .
uv run pytest
uv run template_validation analysis --config config.json --dry-run
~~~

## Updating generated projects

Copier records the source template and revision. Use copier update to review template changes in an existing generated project. Review the diff and run all checks before accepting updates.

## Related repository

- [dev-templates](https://github.com/asher-tian/dev-templates): template catalog and entry point.
