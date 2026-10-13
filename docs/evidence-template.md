# Week 7 Evidence

Student Name: Jake Van Slyke

Repository URL: https://github.com/jakeryderv/devops-week7

Issue: [Complete Week 7 Compose stack and evidence #1](https://github.com/jakeryderv/devops-week7/issues/1)

Branch: `feat/week7-compose`

Evidence collected: October 13, 2026. Command transcripts include UTC timestamps,
the actual commands, their output, and exit codes. Browser screenshots come from
the running local application, Adminer, or GitHub. Evidence 2 and 6 use the
requested terminal fallback: saved command/output text, also displayed in a
browser and captured as images. They are not screenshots of a native terminal.

## Validation

The uv lockfile is current. The two supplied SQLite tests passed, as did Ruff
lint, Ruff formatting, and ty. The Docker image built successfully. The runtime
checks below separately verify PostgreSQL records, health, persistence, and
volume deletion.

- [Local test output](evidence/00-local-tests.txt)
- [Ruff lint](evidence/00-lint.txt)
- [Ruff formatting](evidence/00-formatting.txt)
- [ty check](evidence/00-types.txt)
- [Compose build and initial startup](evidence/02-compose-build.txt)

## Screenshot 1 – Local app running

![Local Streamlit application with a SQLite request](evidence/01-local-app.png)

The local Streamlit application ran from the uv-managed Python 3.13 environment
using SQLite. The visible request, "Local SQLite - Projector not working", proves
that the starter browser form can create and list a request before Docker is used.

The local launch used `uv run --locked --no-sync streamlit run app/main.py` with
loopback binding, port 8501, headless mode, and usage statistics disabled. The
local process was stopped before Compose started on port 8501.

## Screenshot 2 – Docker Compose services running

![Browser view of the captured docker compose ps output](evidence/02-compose-services.png)

The recorded `docker compose ps` output shows `web`, `db`, and `adminer` running,
with PostgreSQL marked healthy. The web and Adminer ports are available on the
local loopback interface; PostgreSQL is reachable internally through `db:5432`.

This image is a browser view of the terminal transcript, used as the requested
terminal screenshot fallback. The original command/output files are retained:

- [docker compose ps](evidence/02-compose-services.txt)
- [pg_isready reports accepting connections](evidence/02-database-health.txt)

## Screenshot 3 – Browser dashboard with at least two requests

![Compose dashboard with two requests and an updated status](evidence/03-compose-dashboard.png)

Two distinct requests were created through the Compose web application's browser
form: "Projector has no image" and "Library Wi-Fi disconnects". Request 2 was
updated through the dashboard to "In Progress"; the table and status counts show
one Open request and one In Progress request.

[PostgreSQL query confirming both records and the updated status](evidence/03-postgresql-records.txt)

## Screenshot 4 – Adminer connected to PostgreSQL

![Adminer showing both PostgreSQL requests](evidence/04-adminer-records.png)

Adminer connected to PostgreSQL using server `db`, user `student`, and database
`helpdesk`. The `public.requests` table displays the same two requests and the
same status update as the web dashboard, confirming that the app stores its
Compose records in PostgreSQL. The database password is not displayed.

## Screenshot 5 – Data persists after docker compose down / up

![Restarted dashboard with the original requests preserved](evidence/05-persistence.png)

After `docker compose down` and `docker compose up -d`, the same two requests
remained with their original IDs, statuses, and creation timestamps. All three
container IDs changed, and the before/after PostgreSQL query output matched
exactly, proving that the data survived container replacement in the named volume.

- [Container removal](evidence/05-compose-down.txt)
- [Container recreation](evidence/05-compose-up.txt)
- [Container IDs before](evidence/05-containers-before.txt)
- [Container IDs after](evidence/05-containers-after.txt)
- [PostgreSQL records after recreation](evidence/05-postgresql-records-after.txt)
- [Persistence comparison result](evidence/05-persistence-verification.txt)

## Screenshot 6 – Web service logs

![Browser view of the captured web-service logs](evidence/06-web-logs.png)

The `docker compose logs web` transcript shows Streamlit starting and listening
on port 8501. It also contains the supplied app's `use_container_width`
deprecation warning; the warning did not prevent the demonstrated request
creation, status update, or persistence behavior.

This image is a browser view of actual terminal output, used as the requested
terminal screenshot fallback. Both requested service log commands were run:

- [Web-service logs](evidence/06-web-logs.txt)
- [Database logs](evidence/06-database-logs.txt)

The database log records an existing data directory and readiness to accept
connections after the persistence restart.

## Screenshot 7 – Effect of deleting the database volume

![Empty dashboard after deleting and recreating the database volume](evidence/07-volume-deleted.png)

After screenshots 3–6 were saved, `docker compose down -v` removed the containers
and `week7_postgres_data` volume. Following `docker compose up -d` and a browser
reload, the dashboard contained no requests and all status counts were zero.
Deleting the volume removed the previous PostgreSQL data; an independent query
confirmed that the recreated `requests` table contained zero rows.

- [Volume removal output](evidence/07-volume-deletion.txt)
- [Startup with a new volume](evidence/07-compose-up.txt)
- [PostgreSQL request count is zero](evidence/07-postgresql-empty.txt)
- [Healthy services after reset](evidence/07-final-services.txt)

## Screenshot 8 – Final GitHub pull request with passing CI

The PR screenshot and passing CI details will be recorded here after the
pull request checks complete.

## Reflection

These six answers are reserved for the student to complete.

1. Why must the web service use `db` rather than `localhost` to reach PostgreSQL?

   **Your answer:**

2. What problem does the named Docker volume solve?

   **Your answer:**

3. What is the difference between `docker compose down` and `docker compose down -v`?

   **Your answer:**

4. What role does the database health check play?

   **Your answer:**

5. How is Docker Compose similar to Infrastructure as Code?

   **Your answer:**

6. What was the most difficult problem you encountered and how did you solve it?

   **Your answer:**
