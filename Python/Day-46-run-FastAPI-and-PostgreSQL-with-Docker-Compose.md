Regular `Day 46`: Docker Compose for FastAPI + PostgreSQL.

Day 45 ran only the FastAPI container and connected to PostgreSQL on your laptop. Today Docker Compose runs both services together: `api` and `db`.

Sources checked: Docker Compose supports `env_file`, named volumes for persistent database storage, and service definitions with ports/environment. Playwright-style CI is later; today is local container orchestration. See [Docker Compose env files](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/), [Compose services](https://docs.docker.com/reference/compose-file/services/), and [Compose volumes](https://docs.docker.com/reference/compose-file/volumes/).

**Day 46 Goal**

- Understand Docker Compose
- Run FastAPI and PostgreSQL together
- Use Docker network service names
- Persist PostgreSQL data with a volume
- Run Alembic migrations inside container
- Stop depending on local PostgreSQL

Start from Day 45:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-45-docker-fastapi .\day-46-docker-compose
cd day-46-docker-compose
code .
```

Create `.env.compose`:

```env
DB_NAME=expense_app
DB_USER=expense_user
DB_PASSWORD=expense_password
DB_HOST=db
DB_PORT=5432
JWT_SECRET_KEY=change_this_to_a_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Create `compose.yaml`:

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

  api:
    build: .
    env_file:
      - .env.compose
    ports:
      - "8000:8000"
    depends_on:
      - db

volumes:
  postgres_data:
```

Important details:

- Inside Docker, API connects to PostgreSQL using `DB_HOST=db`.
- From your laptop, database is exposed on port `5433`.
- We use `5433` to avoid conflict with local PostgreSQL on `5432`.

Start both containers:

```powershell
docker compose up --build
```

Open:

```text
http://127.0.0.1:8000/docs
```

If API starts but database tables are missing, run migrations in another terminal:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-46-docker-compose"
docker compose exec api alembic upgrade head
```

Then test:

- `GET /health`
- `POST /auth/register`
- `POST /auth/login`
- Authorize
- `POST /categories`
- `POST /expenses`
- `GET /expenses`

Useful commands:

```powershell
docker compose ps
docker compose logs api
docker compose logs db
docker compose exec api alembic upgrade head
docker compose down
docker compose down -v
```

Important: `docker compose down -v` deletes the PostgreSQL volume data. Use only when you want to reset the database.

Add to `.gitignore`:

```gitignore
.env.compose
```

Commit:

```powershell
git add .
git commit -m "Day 46 run FastAPI and PostgreSQL with Docker Compose"
```

Day 46 is complete when you can explain: Docker Compose creates a private network where the API can reach PostgreSQL using the service name `db`.
