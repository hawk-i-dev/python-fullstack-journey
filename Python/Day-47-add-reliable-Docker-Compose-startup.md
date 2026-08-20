Regular `Day 47`: reliable Docker Compose startup.

Today you improve Day 46 so Compose becomes one-command reliable: database starts, API waits for database, Alembic migrations run automatically, then FastAPI starts.

Sources checked: Docker Compose supports `depends_on` with `condition: service_healthy`, `healthcheck`, `env_file`, and Compose Watch for development. FastAPI recommends building your own Docker image instead of using deprecated old FastAPI base images. Sources: [Compose startup order](https://docs.docker.com/compose/how-tos/startup-order/), [Compose Watch](https://docs.docker.com/compose/how-tos/file-watch/), [FastAPI Docker](https://fastapi.tiangolo.com/deployment/docker/).

**Day 47 Goal**

- Add database healthcheck
- Add API startup script
- Wait for PostgreSQL before API starts
- Run Alembic migrations automatically
- Start FastAPI after migration success
- Add Compose Watch for development

Start from Day 46:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-46-docker-compose .\day-47-docker-compose-startup
cd day-47-docker-compose-startup
code .
```

Create folder:

```text
scripts/
```

Create `scripts/wait_for_db.py`:

```python
import os
import time

import psycopg


MAX_ATTEMPTS = 30
WAIT_SECONDS = 2


def wait_for_db():
    for attempt in range(1, MAX_ATTEMPTS + 1):
        try:
            connection = psycopg.connect(
                dbname=os.environ["DB_NAME"],
                user=os.environ["DB_USER"],
                password=os.environ["DB_PASSWORD"],
                host=os.environ["DB_HOST"],
                port=int(os.environ["DB_PORT"]),
                connect_timeout=3,
            )
            connection.close()
            print("Database is ready.")
            return
        except psycopg.OperationalError:
            print(f"Database not ready. Attempt {attempt}/{MAX_ATTEMPTS}.")
            time.sleep(WAIT_SECONDS)

    raise RuntimeError("Database did not become ready in time.")


if __name__ == "__main__":
    wait_for_db()
```

Create `entrypoint.sh`:

```sh
#!/bin/sh
set -e

python scripts/wait_for_db.py
alembic upgrade head

exec uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Create `.gitattributes` to prevent Windows line-ending issues:

```gitattributes
*.sh text eol=lf
```

Update `Dockerfile`:

```dockerfile
FROM python:3.14-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

COPY app ./app
COPY alembic ./alembic
COPY scripts ./scripts
COPY alembic.ini .
COPY entrypoint.sh .

RUN sed -i 's/\r$//' /app/entrypoint.sh \
    && chmod +x /app/entrypoint.sh

EXPOSE 8000

CMD ["/app/entrypoint.sh"]
```

Update `compose.yaml`:

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: expense_app
      POSTGRES_USER: expense_user
      POSTGRES_PASSWORD: expense_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5433:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U expense_user -d expense_app"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  api:
    build: .
    env_file:
      - .env.compose
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy
    develop:
      watch:
        - action: sync+restart
          path: ./app
          target: /app/app
        - action: sync+restart
          path: ./scripts
          target: /app/scripts
        - action: rebuild
          path: requirements.txt
        - action: rebuild
          path: Dockerfile

volumes:
  postgres_data:
```

Run:

```powershell
docker compose up --build
```

You should see logs like:

```text
Database is ready.
Running upgrade ...
Uvicorn running on http://0.0.0.0:8000
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test:

- `GET /health`
- `POST /auth/register`
- `POST /auth/login`
- Authorize
- `POST /categories`
- `POST /expenses`
- `GET /expenses`

For development watch mode:

```powershell
docker compose up --watch
```

Useful troubleshooting:

```powershell
docker compose ps
docker compose logs api
docker compose logs db
docker compose restart api
docker compose down
```

Do not use this unless you want to delete database data:

```powershell
docker compose down -v
```

Commit:

```powershell
git add .
git commit -m "Day 47 add reliable Docker Compose startup"
```

Day 47 is complete when you can explain: `depends_on` waits for PostgreSQL health, `entrypoint.sh` waits again from the API side, Alembic updates the schema, and only then Uvicorn starts.
