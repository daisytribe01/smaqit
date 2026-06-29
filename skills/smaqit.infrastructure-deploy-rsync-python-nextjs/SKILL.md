---
name: smaqit.infrastructure-deploy-rsync-python-nextjs
description: Use when deploying a Python/FastAPI backend + Next.js frontend application to a remote VM via rsync. Validated 2026-06-29 on Fashion App — AI Stylist deployed to Cyso Cloud (s5.small, Ubuntu 24.04). Covers Python source rsync, pnpm Next.js dev mode, Docker build gotchas, and database migrations. For Node.js + Vite/React deployments, use `smaqit.infrastructure-deploy-rsync`.
metadata:
  version: "1.0.0"
  validated: "2026-06-29"
  validated-stack: "Python 3.12, FastAPI 0.115, Next.js 15, pnpm 9, PostgreSQL 16, Docker Compose"
---

# Deploy Python/FastAPI + Next.js via rsync

Validated path for deploying a Python backend with a Next.js frontend to a remote VM via rsync.
Based on the Fashion App — AI Stylist deployment to Cyso Cloud (2026-06-29).

## Pre-conditions

- VM bootstrapped (`smaqit.infrastructure-vm-bootstrap` complete)
- Local Vault running and unsealed; SSH private key at `secret/<project-slug>/ssh`
- Docker running on VM and `ubuntu` in docker group
- `deployment/docker-compose.yml` and `deployment/nginx/{project}.conf` present locally
- Production docker-compose MUST include `127.0.0.1` port mappings for nginx (see Docker section)

## Steps

1. **Fetch SSH key from Vault:**
   ```bash
   export VAULT_ADDR=http://127.0.0.1:8200
   export PROJECT_SLUG=<project-slug>
   TMPKEY=$(mktemp) && trap "rm -f $TMPKEY" EXIT
   vault kv get -field=private_key secret/${PROJECT_SLUG}/ssh > "$TMPKEY"
   chmod 600 "$TMPKEY"
   ```
   If the key fails with "error in libcrypto", the Vault-stored key is missing a trailing newline.
   Store as base64 in Vault and decode on fetch: `base64 -d > "$TMPKEY"`.
   See `smaqit.infrastructure-vault-loader` gotchas.

2. **Build frontend** (Next.js):
   ```bash
   cd frontend && pnpm build
   ```
   Produces `frontend/.next/`. If `.next/` has permission issues from a previous Docker build:
   `sudo rm -rf .next && pnpm build`.

3. **Transfer backend to VM** (rsync entire source tree):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> "mkdir -p {deploy_path}/backend"
   rsync -avz --delete backend/ ubuntu@<host>:{deploy_path}/backend/
   ```
   CRITICAL: Rsync `backend/` recursively — this must include top-level files
   (`app/__init__.py`, `app/main.py`, `app/db.py`, `pyproject.toml`, `Dockerfile`).
   Do NOT rsync only subdirectories.

4. **Install Python deps on VM** (throwaway container):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> \
     "cd {deploy_path}/backend && docker run --rm -v \$(pwd):/app -w /app python:3.12-slim bash -c 'pip install uv && uv pip install --system .[dev]'"
   ```

5. **Transfer frontend build + source** (Next.js dev mode):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> "mkdir -p {deploy_path}/frontend"
   rsync -avz --delete frontend/.next/ ubuntu@<host>:{deploy_path}/frontend/.next/
   rsync -avz frontend/package.json frontend/pnpm-lock.yaml frontend/next.config.js ubuntu@<host>:{deploy_path}/frontend/
   rsync -avz frontend/src/ ubuntu@<host>:{deploy_path}/frontend/src/
   rsync -avz frontend/tsconfig.json frontend/postcss.config.mjs frontend/tailwind.config.js ubuntu@<host>:{deploy_path}/frontend/
   ```
   CRITICAL: Rsync config files (`tsconfig.json`, `postcss.config.mjs`, `tailwind.config.js`)
   as individual files. If they arrive as empty directories, Next.js fails with
   `Can't resolve '@/lib/auth'` or `TS5083: Cannot read file`.
   Verify with: `ssh ubuntu@<host> "file {deploy_path}/frontend/tsconfig.json"`

6. **Install Node deps on VM** (throwaway container):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> \
     "cd {deploy_path}/frontend && docker run --rm -v \$(pwd):/app -w /app node:22 bash -c 'npm install -g pnpm@9 && pnpm install --frozen-lockfile'"
   ```
   Use `npm install -g pnpm@9` — `corepack prepare pnpm@9` produces broken installs on `node:22`
   where every pnpm command fails with "packages field missing or empty".

7. **Transfer config files:**
   ```bash
   scp -i "$TMPKEY" deployment/docker-compose.yml ubuntu@<host>:{deploy_path}/docker-compose.yml
   scp -i "$TMPKEY" deployment/nginx/{project}.conf ubuntu@<host>:/etc/nginx/sites-available/{project}
   ssh -i "$TMPKEY" ubuntu@<host> "sudo ln -sf /etc/nginx/sites-available/{project} /etc/nginx/sites-enabled/{project}"
   ```

8. **Run database migrations** (PostgreSQL + Alembic):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> \
     "cd {deploy_path}/backend && docker run --rm -v \$(pwd):/app -w /app --network host python:3.12-slim bash -c 'pip install uv && uv pip install --system .[dev] && alembic upgrade head'"
   ```
   REQUIRED. The `--network host` flag allows the throwaway container to reach the db container on localhost.
   Skipping this causes 500 errors: `relation "users" does not exist`.

9. **Restart containers:**
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path} && docker compose up -d --force-recreate"
   ```
   CRITICAL: Use `up -d --force-recreate`, NOT `restart`. `restart` reuses stale container config
   including bind mounts.

10. **Reload nginx:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> "sudo nginx -t && sudo systemctl reload nginx"
    ```

11. **Write deploy stamps:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> \
      "printf '%s' '$(git rev-parse HEAD)' > {deploy_path}/backend/DEPLOY_SHA && \
       printf '%s' '$(date -u +%Y-%m-%dT%H:%M:%SZ)' > {deploy_path}/backend/DEPLOY_TIME"
    ```

12. **Verify:** Invoke `smaqit.infrastructure-deploy-verify` with the VM URL.

## Docker-Related Deployments

### Production docker-compose.yml requirements

1. **Port mappings for nginx:** Add `127.0.0.1` bindings so nginx on the host can reach containers:
   ```yaml
   api:
     ports: ["127.0.0.1:8000:8000"]
   frontend:
     ports: ["127.0.0.1:3000:3000"]
   ```
   Without these, nginx returns 502 because it cannot reach the Docker network.

2. Remove dev bind-mount volumes (`./backend:/app`). Files are baked into images at build time.

3. Remove `--reload` flags from uvicorn — production uses the base command.

### Python/FastAPI Dockerfile — validated pattern

```dockerfile
FROM python:3.12-slim
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends curl libpq-dev
RUN pip install --no-cache-dir uv
COPY . .
RUN uv pip install --system ".[dev]"
ENV PYTHONPATH=/app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

CRITICAL: `COPY . .` must come BEFORE `RUN uv pip install`. Do NOT use `-e` (editable) in Docker —
it requires the source tree at link time, which the layered COPY doesn't provide.
`ENV PYTHONPATH=/app` is required when the setuptools config doesn't register top-level imports.

### Next.js frontend — dev mode container

Next.js production mode (`next start`) was not achieved during validation. The validated approach
runs the frontend in dev mode with source mounted as bind volumes:

```bash
docker run -d --name {project}-frontend --network {project}_default --restart unless-stopped \
  -e NEXT_PUBLIC_API_URL= \
  -v {deploy_path}/frontend/src:/app/src:ro \
  -v {deploy_path}/frontend/public:/app/public:ro \
  -v {deploy_path}/frontend/package.json:/app/package.json:ro \
  -v {deploy_path}/frontend/node_modules:/app/node_modules \
  -v {deploy_path}/frontend/next.config.js:/app/next.config.js:ro \
  -v {deploy_path}/frontend/tsconfig.json:/app/tsconfig.json:ro \
  -v {deploy_path}/frontend/postcss.config.mjs:/app/postcss.config.mjs:ro \
  -v {deploy_path}/frontend/tailwind.config.js:/app/tailwind.config.js:ro \
  -p 127.0.0.1:3000:3000 -w /app --entrypoint npx node:22 next dev -p 3000
```

Set `NEXT_PUBLIC_API_URL=` (empty string) so the frontend uses relative API URLs.
The frontend code must use `??` (nullish coalescing) not `||` for the fallback:
```typescript
const API_BASE = process.env.NEXT_PUBLIC_API_URL ?? "http://localhost:8080";
```

### Container lifecycle

- `docker restart` does NOT recreate bind mounts — use `docker compose up -d --force-recreate`
- After `docker compose build`, the image must be re-created with `up`, not `restart`
- `sed` on YAML files is fragile — prefer rsyncing a fresh file from local

## Output

Application artifacts deployed to `{deploy_path}/` on the VM, containers running, nginx serving.

## Scope

- Does NOT provision the VM — use `smaqit.infrastructure-provision-cyso` for that.
- Does NOT support Next.js production mode (`next start`) — only dev mode (`next dev`) validated.
- Does NOT pull from a container registry — deploys local build artifacts directly.
- For Node.js + Vite/React deployments, use `smaqit.infrastructure-deploy-rsync`.

## Gotchas

### Python backend
- **`ModuleNotFoundError: No module named 'app.main'`** — Dockerfile has `RUN uv pip install` before
  `COPY . .`. Fix: move `COPY . .` before the install step, add `ENV PYTHONPATH=/app`.
- **Top-level source files missing** — rsyncing `backend/app/*/` only copies subdirectories,
  misses `app/__init__.py`, `app/main.py`, `app/db.py`. Always rsync `backend/` recursively.

### Next.js frontend
- **pnpm broken on node:22** — `corepack prepare pnpm@9` produces installs where every command
  fails with "packages field missing or empty". Fix: `npm install -g pnpm@9`.
- **Config files as directories** — `tsconfig.json`, `postcss.config.mjs`, `tailwind.config.js`
  may arrive as empty directories instead of files. Rsync individually; verify with `file <path>`.
- **`.next/` Docker-owned permissions** — EACCES on `rm -rf` after a Docker build. Fix: `sudo rm -rf .next`.
- **`NEXT_PUBLIC_API_URL=""` truthiness** — `||` treats empty string as falsy, falling back to
  `localhost:8080`. Use `??` (nullish coalescing) instead.
- **Production mode not validated** — `next start` requires `required-server-files.json` which
  only exists in a complete production build. Dev mode (`npx next dev`) is the validated path.

### Docker
- **Port exposure** — production compose needs `127.0.0.1` port mappings for nginx on host.
  Without them: 502 Bad Gateway.
- **`docker restart` vs `--force-recreate`** — `restart` keeps stale bind mounts.
  Always use `up -d --force-recreate`.

### Database
- **Migrations not run** — `alembic upgrade head` required after deploy.
  Skipping → 500 errors: `relation "<table>" does not exist`.

## Failure Handling

| Situation | Action |
|-----------|--------|
| SSH key "error in libcrypto" | Store key as base64 in Vault; decode on fetch |
| `pnpm build` EACCES | `sudo rm -rf .next && pnpm build` |
| `ModuleNotFoundError: app.main` | Dockerfile: COPY before RUN; add ENV PYTHONPATH=/app |
| pnpm "packages field missing" | `npm install -g pnpm@9` instead of corepack |
| `Can't resolve '@/lib/auth'` | Verify tsconfig.json is a file, not a directory |
| nginx 502 Bad Gateway | Check `ss -tlnp` for ports; add `127.0.0.1` mappings to compose |
| `next start` "no production build" | Use `npx next dev` instead (production mode not validated) |
| "relation does not exist" in API logs | Run `alembic upgrade head` |
| compose validation error | YAML corrupted; rsync clean file from local |
| SHA mismatch in verify | Use `up -d --force-recreate`, not `restart` |
