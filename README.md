# SDI 4213 Week 7 Service Request Tracker

A Streamlit help desk application with a PostgreSQL database and Adminer browser
interface, configured with Docker Compose. Requests support creation, listing,
and status updates. Running the app outside Compose uses SQLite by default.

The assignment is in [the Week 7 directions](docs/Week7_Individual_Exercise.docx).
The report is in [the evidence document](docs/evidence-template.md), with
screenshots and command transcripts under [docs/evidence/](docs/evidence/).
The student completes the six reflection answers in that report.

## Requirements

- Git, uv, Python 3.13 (uv can install it), and a browser
- Docker Engine or Docker Desktop with Docker Compose
- Ports 8501 and 8080 available locally

## Local development with uv

From the repository root:

```sh
uv sync --locked
uv run --locked python -m pytest -v
uv run --locked streamlit run app/main.py
```

Open `http://localhost:8501` and create a request. With no `DATABASE_HOST` or
`DATABASE_URL` in the shell environment, the app stores requests in the ignored
local `helpdesk.db` SQLite file. The `.env` file is read by Compose, not loaded
automatically by the local Streamlit command. Stop the local app with Ctrl+C
before starting Compose on the same port.

## Run the Compose application

```sh
cp .env.example .env
# Edit .env and replace the example password before starting the stack.
docker compose config --quiet
docker compose up --build -d
docker compose ps
```

Use a URL-safe development password: the supplied database helper interpolates
the credentials into a connection URL. `.env` is ignored by Git and excluded
from Docker's build context. Do not put its contents into evidence or commits.

| Service | Access | Configuration |
|---|---|---|
| Streamlit `web` | `http://localhost:8501` | Built from the Dockerfile; connects to `db:5432` |
| PostgreSQL `db` | Compose network | `postgres:17`; data in the `postgres_data` named volume |
| Adminer `adminer` | `http://localhost:8080` | `adminer:5`; default server `db` |

Both browser ports are bound to the local loopback interface. PostgreSQL does
not publish a host port. Compose's default network resolves the service name
`db`; `localhost` inside the web container would refer to that container.

The database health check uses `pg_isready`. Both web and Adminer wait for a
healthy database before starting. PostgreSQL 17's data volume is mounted at
`/var/lib/postgresql/data`.

In Adminer, choose **PostgreSQL**, use **db** as the server, and enter the user,
password, and database from `.env`. Open the `requests` table to inspect rows.

## Persistence, logs, and reset

Create two distinct requests in the dashboard and change one status. Then:

```sh
docker compose down
docker compose up -d
docker compose ps
```

After healthy startup, reload the dashboard. The named volume retains the same
records even though the containers were replaced.

```sh
docker compose logs web
docker compose logs db
```

Capture the persistence and log evidence before intentionally deleting data:

```sh
docker compose down -v
docker compose up -d
```

`down -v` deletes this Compose project's named volume. After startup and a
browser reload, the previous requests are gone. Use ordinary `docker compose
down` when stopping the app while retaining its data.

## Validation and dependency updates

`pyproject.toml` declares dependencies, `uv.lock` records resolved versions, and
`requirements.txt` is the generated runtime export used by the Dockerfile.
Pytest, Ruff, and ty are development dependencies installed by uv.

```sh
uv lock --check
uv run --locked ruff check app tests
uv run --locked ruff format --check app tests
uv run --locked ty check
uv run --locked python -m pytest -v
docker compose --env-file .env.example config --quiet
```

To apply lint fixes and formatting, run `uv run ruff check --fix app tests` and
`uv run ruff format app tests`, then repeat validation. After changing
dependencies with `uv add`, `uv remove`, or an intentional upgrade, regenerate:

```sh
uv export --quiet --locked --no-dev --no-emit-project --no-header --output-file requirements.txt
```

Commit `pyproject.toml`, `uv.lock`, `.python-version`, and the generated export.
CI runs lint, formatting, types, the two SQLite tests, export consistency, and
Compose configuration validation. The browser/Compose evidence separately
demonstrates PostgreSQL behavior, database readiness, and volume persistence.

## GitHub workflow

Use Issue → Branch → Change → Commit → Push → Pull Request → CI → Review →
Merge. The PR links to the assignment issue and contains the configuration,
documentation, and evidence. Capture the PR with passing CI, commit that
evidence, and verify CI also passes for the final commit before merging.
