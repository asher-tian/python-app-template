# Python App Template

A reusable [Copier](https://copier.readthedocs.io/) template for building maintainable Python applications.

The generated project uses a CLI interface by default and keeps the application core independent from its delivery interfaces. It can be extended with Web APIs, SDKs, and additional infrastructure adapters as needed.

## Quick start

Install Copier and run:

```bash
copier copy gh:asher-tian/python-app-template ./my-app
```

Pin a template release for reproducible project generation:

```bash
copier copy --vcs-ref v1.0.0 gh:asher-tian/python-app-template ./my-app
```

Copier downloads this template directly from GitHub. You do not need to clone the `dev-templates` catalog or this repository manually.

## Initialize the generated project

```bash
cd my-app
uv sync --all-groups
uv run pytest
uv run ruff check .
uv run ruff format --check .
```

The template asks for the project display name, distribution name, Python package name, and Python version.

## Architecture

The generated project is organized into these layers:

- `core/`: application API, use cases, Tasks, Steps, business models, and ports. It does not depend on CLI frameworks or concrete infrastructure.
- `interfaces/`: delivery adapters. The initial template includes a CLI interface.
- `infrastructure/`: concrete implementations for compute, database, storage, and logging.
- `runtime/`: configuration, composition root, scheduling, and resource lifecycle.
- `tests/`: application, infrastructure, CLI, runtime, and process-pool tests.

The example runtime uses a thread-based Task scheduler and one shared spawn-based ProcessPool for CPU-intensive work. SQLite persistence is optional and disabled by default.

## Configuration

The generated project includes a sample `config.json` with:

- Work directory and log directory.
- CPU budget and reserved CPU count.
- Maximum concurrent Tasks.
- Per-Task in-flight compute limit and chunk size.
- Optional SQLite persistence.
- Multiple example Task groups.

See the generated project's README and AGENTS.md for implementation and extension guidance.

## Update an existing project

Copier records the source template and revision in the generated project. To check for and apply template updates later, run:

```bash
copier update
```

Review the resulting diff and run the project's checks before accepting an update.

## Development and validation

The repository CI generates a fresh project from this template, installs its dependencies, and runs Ruff and pytest.

To validate locally:

```bash
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
```

## Related repository

- [dev-templates](https://github.com/asher-tian/dev-templates): the catalog and entry point for all project templates.
