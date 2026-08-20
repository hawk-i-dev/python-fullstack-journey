Regular `Day 49`: production-ready Docker cleanup.

Today you make the Day 48 full-stack Docker setup cleaner and safer: one root `.env`, health checks, restart policies, better Nginx config, and a repeatable run checklist.

Sources checked: Docker recommends separate env files, understanding env precedence, health checks with `depends_on: condition: service_healthy`, and using secrets for real sensitive production data. See Docker docs on [environment best practices](https://docs.docker.com/compose/how-tos/environment-variables/best-practices/), [Compose startup order](https://docs.docker.com/compose/how-tos/startup-order/), and [Compose secrets](https://docs.docker.com/reference/compose-file/secrets/).

**Start**

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-48-fullstack-docker .\day-49-production-docker-cleanup
cd day-49-production-docker-cleanup
code .
```

Create root `.env`:

```env
POSTGRES_DB=expense_app
POSTGRES_USER=expense_user
POSTGRES_PASSWORD=change_this_password
JWT_SECRET_KEY=change_this_to_a_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
FRONTEND_PORT=3000
API_PORT=8000
DB_PORT=5433
```

Create root `.env.example`:

```env
POSTGRES_DB=expense_app
POSTGRES_USER=expense_user
POSTGRES_PASSWORD=change_me
JWT_SECRET_KEY=change_me_to_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
FRONTEND_PORT=3000
API_PORT=8000
DB_PORT=5433
```

Update root `.gitignore`:

```gitignore
.env
backend/.env.compose
frontend/.env.local
```

Replace root `compose.yaml`:

```yaml
services:
  db:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "${DB_PORT}:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  api:
    build: ./backend
    restart: unless-stopped
    environment:
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
      DB_HOST: db
      DB_PORT: 5432
      JWT_SECRET_KEY: ${JWT_SECRET_KEY}
      JWT_ALGORITHM: ${JWT_ALGORITHM}
      ACCESS_TOKEN_EXPIRE_MINUTES: ${ACCESS_TOKEN_EXPIRE_MINUTES}
    ports:
      - "${API_PORT}:8000"
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD-SHELL", "python -c \"import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health')\""]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  frontend:
    build:
      context: ./frontend
      args:
        VITE_API_URL: /api
    restart: unless-stopped
    ports:
      - "${FRONTEND_PORT}:8080"
    depends_on:
      api:
        condition: service_healthy

volumes:
  postgres_data:
```

Update `frontend/nginx.conf`:

```nginx
events {}

http {
    gzip on;
    gzip_types text/plain text/css application/json application/javascript image/svg+xml;

    server {
        listen 8080;

        root /usr/share/nginx/html;
        index index.html;

        location /assets/ {
            expires 1y;
            add_header Cache-Control "public, immutable";
            try_files $uri =404;
        }

        location /api/ {
            proxy_pass http://api:8000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

Validate Compose config:

```powershell
docker compose config
```

Run full stack:

```powershell
docker compose up --build
```

Open:

```text
http://127.0.0.1:3000
```

Check containers:

```powershell
docker compose ps
docker compose logs api
docker compose logs frontend
docker compose logs db
```

Test:

- Register user
- Login
- Create category
- Add expense
- Open reports
- Refresh browser
- Data should remain

Commit:

```powershell
git add .
git commit -m "Day 49 clean up production Docker configuration"
```

Day 49 is complete when you can explain: `.env` provides runtime config, Compose wires services together, health checks confirm readiness, and Nginx serves React while proxying `/api` to FastAPI.
