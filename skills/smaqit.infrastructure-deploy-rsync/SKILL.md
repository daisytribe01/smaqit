---
name: smaqit.infrastructure-deploy-rsync
description: Use when deploying a backend + frontend application to a remote VM via rsync. Supports Node.js and Python backends, Vite/React and Next.js frontends. Used in Phase 5 dev sweep of smaqit.new-greenfield-project. Also use as a manual fallback for direct VM deployment outside CI/CD.
metadata:
  version: "2.0.0"
---

# Deploy Application via rsync

## Project Type Detection

Before executing steps, determine the project type from the stack spec or codebase:

| Aspect | Variant A: Node.js | Variant B: Python/FastAPI |
|--------|-------------------|--------------------------|
| Backend language | Node.js / TypeScript | Python 3.12+ |
| Backend build | `npm run build` → `dist/` | None (interpreted, rsync source) |
| Backend deps | `npm install --production` | `uv pip install --system ".[dev]"` |
| Frontend framework | Vite / React | Next.js |
| Frontend build | `npm run build` → `dist/` | `pnpm build` → `.next/` |
| Frontend deps | npm | pnpm |
| Frontend serve | `npm start` (static/express) | `npx next dev` or `npx next start` |

Use the appropriate variant in the steps below. Variant references are marked **[Node]** or **[Python]**.

## Pre-conditions

- VM bootstrapped; Docker running on VM and `ubuntu` in docker group
- Local Vault running and unsealed; SSH private key at `secret/<project-slug>/ssh`
- `deployment/docker-compose.yml` and `deployment/nginx/{project}.conf` present locally
- Production docker-compose MUST include `127.0.0.1` port mappings for nginx (see Docker section)

## Steps

1. **Fetch SSH key:**
   ```bash
   export VAULT_ADDR=http://127.0.0.1:8200
   export PROJECT_SLUG=<project-slug>
   TMPKEY=$(mktemp) && trap "rm -f $TMPKEY" EXIT
   vault kv get -field=private_key secret/${PROJECT_SLUG}/ssh > "$TMPKEY"
   chmod 600 "$TMPKEY"
   ```
   If the key fails with "error in libcrypto", the Vault-stored key is missing a trailing newline.
   Fix: store as base64 (`base64 -w0`) and decode on fetch (`base64 -d > "$TMPKEY"`).

2. **Build backend:**
   - **[Python]** No build step — backend runs from source.
   - **[Node]** `cd backend && npm run build` → produces `backend/dist/`.

3. **Build frontend:**
   - **[Next.js]** `cd frontend && pnpm build` → `frontend/.next/`. If `.next/` has Docker-owned permissions: `sudo rm -rf .next` before rebuild.
   - **[Vite]** `cd frontend && npm run build` → `frontend/dist/`.

4. **Transfer backend to VM:**
   - **[Python]** Rsync entire source recursively:
     ```bash
     ssh -i "$TMPKEY" ubuntu@<host> "mkdir -p {deploy_path}/backend"
     rsync -avz --delete backend/ ubuntu@<host>:{deploy_path}/backend/
     ```
     CRITICAL: Verify top-level files (`app/__init__.py`, `app/main.py`, `app/db.py`, `pyproject.toml`, `Dockerfile`) all transferred. Do NOT just rsync subdirectories.
   - **[Node]** Rsync dist + package files:
     ```bash
     ssh -i "$TMPKEY" ubuntu@<host> "mkdir -p {deploy_path}/backend/dist"
     rsync -avz --delete backend/dist/ ubuntu@<host>:{deploy_path}/backend/dist/
     rsync -avz backend/package.json backend/package-lock.json ubuntu@<host>:{deploy_path}/backend/
     ```

5. **Install backend deps on VM** (throwaway container):
   - **[Python]**
     ```bash
     ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path}/backend && docker run --rm -v \$(pwd):/app -w /app python:3.12-slim bash -c 'pip install uv && uv pip install --system .[dev]'"
     ```
   - **[Node]**
     ```bash
     ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path}/backend && docker run --rm -v \$(pwd):/app -w /app node:22-alpine npm install --production"
     ```

6. **Transfer frontend build:**
   - **[Next.js]** Rsync `.next/` + config files + `src/`:
     ```bash
     ssh -i "$TMPKEY" ubuntu@<host> "mkdir -p {deploy_path}/frontend"
     rsync -avz --delete frontend/.next/ ubuntu@<host>:{deploy_path}/frontend/.next/
     rsync -avz frontend/package.json frontend/pnpm-lock.yaml frontend/next.config.js ubuntu@<host>:{deploy_path}/frontend/
     rsync -avz frontend/src/ ubuntu@<host>:{deploy_path}/frontend/src/
     rsync -avz frontend/tsconfig.json frontend/postcss.config.mjs frontend/tailwind.config.js ubuntu@<host>:{deploy_path}/frontend/
     ```
     CRITICAL: Rsync config files individually. If they arrive as empty directories, Next.js fails with `Can't resolve '@/lib/auth'` or `TS5083: Cannot read file`.
   - **[Vite]** `rsync -avz --delete frontend/dist/ ubuntu@<host>:{deploy_path}/frontend/`

7. **Install frontend deps on VM:**
   - **[Next.js / pnpm]**
     ```bash
     ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path}/frontend && docker run --rm -v \$(pwd):/app -w /app node:22 bash -c 'npm install -g pnpm@9 && pnpm install --frozen-lockfile'"
     ```
     Use `npm install -g pnpm@9` — `corepack prepare pnpm@9` produces broken installs on `node:22`.

8. **Transfer config files:**
   ```bash
   scp -i "$TMPKEY" deployment/docker-compose.yml ubuntu@<host>:{deploy_path}/docker-compose.yml
   scp -i "$TMPKEY" deployment/nginx/{project}.conf ubuntu@<host>:/etc/nginx/sites-available/{project}
   ssh -i "$TMPKEY" ubuntu@<host> "sudo ln -sf /etc/nginx/sites-available/{project} /etc/nginx/sites-enabled/{project}"
   ```

9. **Run database migrations** (REQUIRED for PostgreSQL + Alembic projects):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path}/backend && docker run --rm -v \$(pwd):/app -w /app --network host python:3.12-slim bash -c 'pip install uv && uv pip install --system .[dev] && alembic upgrade head'"
   ```
   The `--network host` flag allows the throwaway container to reach the db container on localhost.

10. **Restart containers:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path} && docker compose up -d --force-recreate"
    ```
    CRITICAL: Use `up -d --force-recreate`, NOT `restart`. `restart` reuses stale bind mounts.

11. **Reload nginx:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> "sudo nginx -t && sudo systemctl reload nginx"
    ```

12. **Write deploy stamps:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> "printf '%s' '$(git rev-parse HEAD)' > {deploy_path}/backend/DEPLOY_SHA && printf '%s' '$(date -u +%Y-%m-%dT%H:%M:%SZ)' > {deploy_path}/backend/DEPLOY_TIME"
    ```

13. **Verify:** Invoke `smaqit.infrastructure-deploy-verify` with the VM URL.

## Docker-Related Deployments

### Production docker-compose.yml requirements

1. **Port mappings for nginx** — add `127.0.0.1` bindings so nginx on the host can reach containers:
   ```yaml
   api:
     ports: ["127.0.0.1:8000:8000"]
   frontend:
     ports: ["127.0.0.1:3000:3000"]
   ```
   Without these, nginx returns 502 because it cannot reach the Docker network.

2. Remove dev bind-mounts (`./backend:/app`). Files are baked into images at build time.

3. Remove `--reload` flags from uvicorn. Production uses the base command.

### Python/FastAPI Dockerfile — correct pattern

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

CRITICAL: `COPY . .` must come BEFORE `RUN uv pip install`. Do NOT use `-e` (editable) in Docker.
`ENV PYTHONPATH=/app` is required when setuptools doesn't register top-level imports.

### Next.js in Docker

- `pnpm` via `corepack prepare` is broken on `node:22`. Use `npm install -g pnpm@9`.
- For PoC/dev: run `npx next dev` with source mounted as bind volumes.
- `NEXT_PUBLIC_*` vars must be at build time (Dockerfile) or passed via `-e` in dev mode.

### Container lifecycle

- `docker restart` does NOT recreate bind mounts — use `docker compose up -d --force-recreate`.
- After `docker compose build`, the image must be re-created with `up`, not `restart`.
- `sed` on YAML is fragile — prefer rsyncing a fresh file from local deployment directory.

## Output

Application artifacts deployed to `{deploy_path}/` on the VM, container running, nginx serving.

## Scope

- Does NOT provision the VM — use `smaqit.infrastructure-provision-cyso` for that.
- Does NOT pull from a container registry — deploys local build artifacts directly.

## Gotchas

### rsync
- **Top-level Python files missing** — rsyncing `backend/app/*/` only copies subdirectories, misses `app/__init__.py`, `app/main.py`. Always rsync `backend/` recursively.
- **Config files as directories** — `tsconfig.json`/`postcss.config.mjs`/`tailwind.config.js` may arrive as empty dirs. Rsync as individual files; verify with `file <path>`.
- **Trailing slash depth** — `rsync backend/dist/` copies contents; missing target dir causes wrong depth. Always `mkdir -p` first.

### Docker
- **pnpm broken on node:22** — `corepack prepare pnpm@9` → "packages field missing or empty" on all commands. Fix: `npm install -g pnpm@9`.
- **Python editable install in Docker** — `-e` flag requires source at link time; layered COPY breaks it. Use non-editable `uv pip install --system ".[dev]"` after `COPY . .`.
- **PYTHONPATH required** — `ENV PYTHONPATH=/app` when package isn't auto-registered.
- **docker restart vs --force-recreate** — `restart` keeps stale bind mounts. Always `up -d --force-recreate`.
- **Port exposure** — production compose needs `127.0.0.1` port mappings for nginx.

### Next.js
- **`NEXT_PUBLIC_API_URL=""` truthiness** — `||` treats empty string as falsy. Use `??` (nullish coalescing).
- **`.next/` Docker-owned permissions** — EACCES on `rm -rf`. Use `sudo rm -rf .next`.
- **`next start` needs production build** — missing `required-server-files.json`. Dev mode: `npx next dev`.

### Config
- **`sed` range delete on YAML** — `sed '/pat1/,/pat2/d'` corrupts structured YAML. Use Python `yaml` or rsync clean file.

### Database
- **Migrations not run** — `alembic upgrade head` required after deploy. Skipping → 500 errors.

## Failure Handling

| Situation | Action |
|-----------|--------|
| SSH key "error in libcrypto" | Store key as base64 in Vault; decode on fetch |
| `pnpm build` EACCES | `sudo rm -rf .next && pnpm build` |
| `ModuleNotFoundError: app.main` | Dockerfile: COPY before RUN; add ENV PYTHONPATH=/app |
| pnpm "packages field missing" | `npm install -g pnpm@9` instead of corepack |
| `Can't resolve '@/lib/auth'` | Verify tsconfig.json is a file, not a directory |
| nginx 502 | `ss -tlnp` check ports; add `127.0.0.1` mappings to compose |
| `next start` "no production build" | Rebuild `.next/` or use `npx next dev` |
| "relation does not exist" (DB) | Run `alembic upgrade head` |
| compose validation error | YAML corrupted; rsync clean file |
| SHA mismatch in verify | Use `up -d --force-recreate`, not `restart` |
