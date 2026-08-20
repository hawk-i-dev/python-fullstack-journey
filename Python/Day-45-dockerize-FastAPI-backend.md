Regular `Day 45`: Dockerize the FastAPI backend.

Today you package your FastAPI backend into a Docker container. This is the first serious deployment foundation. Docker lets your app run with the same Python version, dependencies, and startup command on any machine or server.

Sources checked: Docker’s Python guide recommends containerizing Python apps with a `Dockerfile` and Compose; FastAPI’s Docker guide shows copying `requirements.txt`, installing dependencies, copying `app/`, and running the app inside the container. See [Docker Python guide](https://docs.docker.com/guides/python/) and [FastAPI Docker deployment](https://fastapi.tiangolo.com/deployment/docker/).

**Day 45 Goal**

- Understand Docker image vs container
- Create `Dockerfile`
- Create `.dockerignore`
- Build FastAPI backend image
- Run backend container
- Use environment variables in Docker
- Keep database outside container for today

Start from Day 21 backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-21-user-owned-expenses .\day-45-docker-fastapi
cd day-45-docker-fastapi
code .
```

Install Docker Desktop if not installed. Then verify:

```powershell
docker --version
docker compose version
```

Make sure `requirements.txt` contains:

```text
fastapi
uvicorn
sqlalchemy
psycopg[binary]
pydantic-settings
alembic
pyjwt
pwdlib[argon2]
python-multipart
```

Create `.dockerignore`:

```dockerignore
.venv/
__pycache__/
.pytest_cache/
.mypy_cache/
.git/
.env
*.pyc
alembic/versions/__pycache__/
```

Create `Dockerfile`:

```dockerfile
FROM python:3.14-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

COPY app ./app
COPY alembic ./alembic
COPY alembic.ini .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Create `.env.docker`:

```env
DB_NAME=python_fullstack_day21
DB_USER=postgres
DB_PASSWORD=your_postgres_password
DB_HOST=host.docker.internal
DB_PORT=5432
JWT_SECRET_KEY=your_generated_secret_key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Important: `host.docker.internal` lets the container connect to PostgreSQL running on your laptop.

Build image:

```powershell
docker build -t expense-api-day45 .
```

Run container:

```powershell
docker run --env-file .env.docker -p 8000:8000 expense-api-day45
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
- `GET /expenses`

If port `8000` is busy:

```powershell
docker run --env-file .env.docker -p 8001:8000 expense-api-day45
```

Then open:

```text
http://127.0.0.1:8001/docs
```

Useful Docker commands:

```powershell
docker ps
docker ps -a
docker images
docker logs <container_id>
docker stop <container_id>
```

Do not commit real secrets. Add this to `.gitignore`:

```gitignore
.env
.env.docker
```

Commit:

```powershell
git add .
git commit -m "Day 45 dockerize FastAPI backend"
```

Day 45 is complete when you can explain: a Docker image is the packaged app, and a container is a running instance of that image.
