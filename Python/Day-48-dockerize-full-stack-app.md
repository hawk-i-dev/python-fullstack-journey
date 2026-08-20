Regular `Day 48`: Dockerize the full stack.

Today you run everything with Docker Compose:

```text
React frontend -> Nginx -> /api proxy -> FastAPI -> PostgreSQL
```

This removes the CORS problem because the browser calls the same frontend origin, and Nginx forwards `/api` requests to FastAPI.

Sources checked: Vite builds static files into `dist`, Vite `VITE_*` variables are baked at build time, and Docker’s React guide uses a Node build stage plus Nginx serve stage. See [Vite production build](https://vite.dev/guide/build.html), [Vite env variables](https://vite.dev/guide/env-and-mode), and [Docker React guide](https://docs.docker.com/guides/reactjs/).

**Start**

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
mkdir day-48-fullstack-docker
Copy-Item -Recurse .\day-47-docker-compose-startup .\day-48-fullstack-docker\backend
Copy-Item -Recurse .\day-44-github-actions-ci .\day-48-fullstack-docker\frontend
cd day-48-fullstack-docker
code .
```

**Frontend Dockerfile**

Create `frontend/Dockerfile`:

```dockerfile
FROM node:24-alpine AS builder

WORKDIR /app

ARG VITE_API_URL=/api
ENV VITE_API_URL=$VITE_API_URL

COPY package.json package-lock.json* ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginxinc/nginx-unprivileged:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
```

Create `frontend/nginx.conf`:

```nginx
events {}

http {
    server {
        listen 8080;

        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api/ {
            proxy_pass http://api:8000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

**Root Compose File**

Create `compose.yaml` in `day-48-fullstack-docker`:

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
    build: ./backend
    env_file:
      - ./backend/.env.compose
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy

  frontend:
    build:
      context: ./frontend
      args:
        VITE_API_URL: /api
    ports:
      - "3000:8080"
    depends_on:
      - api

volumes:
  postgres_data:
```

Make sure `backend/.env.compose` has:

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

Run full stack:

```powershell
docker compose up --build
```

Open frontend:

```text
http://127.0.0.1:3000
```

Backend docs still available:

```text
http://127.0.0.1:8000/docs
```

Test:

- Register user from React
- Login from React
- Create categories
- Create expenses
- Open reports
- Refresh browser
- Data should remain

Useful commands:

```powershell
docker compose ps
docker compose logs frontend
docker compose logs api
docker compose logs db
docker compose down
```

Commit:

```powershell
git add .
git commit -m "Day 48 dockerize full stack app"
```

Day 48 is complete when you can explain: React is built into static files, Nginx serves those files, `/api` requests are proxied to FastAPI, and PostgreSQL stores the data in a Docker volume.
