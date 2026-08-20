Regular `Day 50`: publish Docker images to GitHub Container Registry.

Today you make your Dockerized full-stack app ready for deployment. Instead of building images only on your laptop, GitHub Actions will build and push:

```text
backend image  -> ghcr.io/<your-username>/expense-api
frontend image -> ghcr.io/<your-username>/expense-web
```

Sources checked: GitHub recommends `GITHUB_TOKEN` for GHCR publishing, and Docker recommends `login-action`, `metadata-action`, and `build-push-action`. See [GitHub Docker publishing](https://docs.github.com/en/actions/tutorials/publish-packages/publish-docker-images), [GHCR docs](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry), and [Docker login-action](https://github.com/docker/login-action).

**Start**

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-49-production-docker-cleanup .\day-50-ghcr-docker-images
cd day-50-ghcr-docker-images
code .
```

Create workflow at repo root, not inside the day folder:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
mkdir .github\workflows
```

Create `.github/workflows/day-50-docker-images.yml`:

```yaml
name: Day 50 Docker Images

on:
  push:
    branches: [main, master]
  workflow_dispatch:

permissions:
  contents: read
  packages: write

env:
  REGISTRY: ghcr.io
  BACKEND_IMAGE: ${{ github.repository_owner }}/expense-api
  FRONTEND_IMAGE: ${{ github.repository_owner }}/expense-web

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v4
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Backend metadata
        id: backend_meta
        uses: docker/metadata-action@v6
        with:
          images: ${{ env.REGISTRY }}/${{ env.BACKEND_IMAGE }}
          tags: |
            type=ref,event=branch
            type=sha
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push backend
        uses: docker/build-push-action@v7
        with:
          context: day-50-ghcr-docker-images/backend
          push: true
          tags: ${{ steps.backend_meta.outputs.tags }}
          labels: ${{ steps.backend_meta.outputs.labels }}

      - name: Frontend metadata
        id: frontend_meta
        uses: docker/metadata-action@v6
        with:
          images: ${{ env.REGISTRY }}/${{ env.FRONTEND_IMAGE }}
          tags: |
            type=ref,event=branch
            type=sha
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push frontend
        uses: docker/build-push-action@v7
        with:
          context: day-50-ghcr-docker-images/frontend
          push: true
          build-args: |
            VITE_API_URL=/api
          tags: ${{ steps.frontend_meta.outputs.tags }}
          labels: ${{ steps.frontend_meta.outputs.labels }}
```

Add `compose.prod.yaml` inside `day-50-ghcr-docker-images`:

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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    image: ghcr.io/${GHCR_OWNER}/expense-api:${IMAGE_TAG}
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
    depends_on:
      db:
        condition: service_healthy

  frontend:
    image: ghcr.io/${GHCR_OWNER}/expense-web:${IMAGE_TAG}
    restart: unless-stopped
    ports:
      - "${FRONTEND_PORT}:8080"
    depends_on:
      - api

volumes:
  postgres_data:
```

Update `.env.example`:

```env
GHCR_OWNER=your_github_username
IMAGE_TAG=latest
POSTGRES_DB=expense_app
POSTGRES_USER=expense_user
POSTGRES_PASSWORD=change_me
JWT_SECRET_KEY=change_me_to_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
FRONTEND_PORT=3000
```

Before pushing, run locally:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-50-ghcr-docker-images"
docker compose up --build
```

Then commit and push:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
git add .
git commit -m "Day 50 publish Docker images with GitHub Actions"
git push
```

After GitHub Actions finishes, check:

```text
GitHub repo -> Actions
GitHub repo -> Packages
```

If push fails with package permission issue, go to:

```text
GitHub repo -> Settings -> Actions -> General -> Workflow permissions
```

Set:

```text
Read and write permissions
```

Day 50 is complete when you can explain: CI builds Docker images from source, tags them, pushes them to GHCR, and deployment servers can later pull those images instead of rebuilding from code.
