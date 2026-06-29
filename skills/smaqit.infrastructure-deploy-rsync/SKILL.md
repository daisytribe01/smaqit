---
name: smaqit.infrastructure-deploy-rsync
description: Use when deploying a Node.js backend + React frontend application to a remote VM via rsync. Used in the Phase 5 dev environment sweep of `smaqit.new-greenfield-project` to validate the deployment approach locally before CI/CD. Also use as a manual fallback for direct VM deployment outside the CI/CD pipeline. For Python/FastAPI + Next.js deployments, see `smaqit.infrastructure-deploy-rsync-python-nextjs`.
metadata:
  version: "1.1.0"
---

# Deploy Application via rsync (Node.js)

## Pre-conditions

- VM bootstrapped (`smaqit.infrastructure-vm-bootstrap` complete)
- Local Vault running and unsealed (`smaqit.infrastructure-vault-loader` complete); SSH private key at `secret/<project-slug>/ssh`
- Docker running on VM and `ubuntu` in docker group
- `deployment/docker-compose.yml` and `deployment/nginx/{project}.conf` present locally

## Steps

1. **Fetch SSH key from Vault into a secure temp file:**
   ```bash
   export VAULT_ADDR=http://127.0.0.1:8200
   export PROJECT_SLUG=<project-slug>   # from copilot-instructions.md

   TMPKEY=$(mktemp)
   trap "rm -f $TMPKEY" EXIT
   vault kv get -field=private_key secret/${PROJECT_SLUG}/ssh > "$TMPKEY"
   chmod 600 "$TMPKEY"
   ```
   All subsequent `ssh`, `rsync`, and `scp` commands use `-i "$TMPKEY"`. The file is wiped
   automatically when the shell exits or the script completes.

2. **Build backend:**
   ```bash
   cd backend && npm run build
   ```
   Produces `backend/dist/`.

3. **Build frontend:**
   ```bash
   cd frontend && npm run build
   ```
   Produces `frontend/dist/`. If `VITE_DEMO_MODE` must be set, export it before building — Vite bakes
   this value in at build time and it cannot be changed without a rebuild:
   ```bash
   VITE_DEMO_MODE=true npm run build
   ```

4. **Transfer backend artifacts to VM:**
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> "mkdir -p {deploy_path}/backend/dist"
   rsync -avz --delete backend/dist/ ubuntu@<host>:{deploy_path}/backend/dist/
   rsync -avz backend/package.json backend/package-lock.json ubuntu@<host>:{deploy_path}/backend/
   ```
   CRITICAL: Always `mkdir -p {deploy_path}/backend/dist` before rsyncing. The trailing slash on
   `backend/dist/` copies the directory's *contents* — if the target directory does not exist, rsync
   creates it one level too shallow and the container fails with
   `Cannot find module '/app/dist/index.js'`.

5. **Install `node_modules` on VM** (production only, inside a throwaway container):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> \
     "cd {deploy_path}/backend && docker run --rm -v \$(pwd):/app -w /app node:22-alpine npm install --production"
   ```

6. **Transfer frontend build:**
   ```bash
   rsync -avz --delete frontend/dist/ ubuntu@<host>:{deploy_path}/frontend/
   ```

7. **Transfer config files:**
   ```bash
   scp -i "$TMPKEY" deployment/docker-compose.yml ubuntu@<host>:{deploy_path}/docker-compose.yml
   scp -i "$TMPKEY" deployment/nginx/{project}.conf ubuntu@<host>:/etc/nginx/sites-available/{project}
   ssh -i "$TMPKEY" ubuntu@<host> "sudo ln -sf /etc/nginx/sites-available/{project} /etc/nginx/sites-enabled/{project}"
   ```

8. **Run database migrations** (if project uses a relational DB with migrations):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path} && docker compose run --rm api <migration-command>"
   ```
   Adapt `<migration-command>` to the project's migration tool (e.g., `alembic upgrade head`,
   `npx prisma migrate deploy`, `npm run migrate`). Skipping this causes 500 errors on endpoints
   that query unmigrated tables.

9. **Write deploy stamp files** (enables SHA verification in the health endpoint):
   ```bash
   ssh -i "$TMPKEY" ubuntu@<host> \
     "printf '%s' '$(git rev-parse HEAD)' > {deploy_path}/backend/DEPLOY_SHA && \
      printf '%s' '$(date -u +%Y-%m-%dT%H:%M:%SZ)' > {deploy_path}/backend/DEPLOY_TIME"
   ```
   Write to `{deploy_path}/backend/`, not `{deploy_path}/` — the container mounts `{deploy_path}/backend/` as `/app`;
   files one level up are invisible to the container.

10. **Restart containers:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> "cd {deploy_path} && docker compose up -d --force-recreate"
    ```
    Use `docker compose` (v2, no hyphen). `--force-recreate` is required because the app is deployed
    as files, not as a new image. Use `up -d --force-recreate`, NOT `restart` — `restart` reuses
    stale container config including bind mounts.

11. **Reload nginx:**
    ```bash
    ssh -i "$TMPKEY" ubuntu@<host> "sudo nginx -t && sudo systemctl reload nginx"
    ```

12. **Verify:** Invoke `smaqit.infrastructure-deploy-verify` with the VM URL.

## Output

Application artifacts deployed to `{deploy_path}/` on the VM, container running, nginx serving.

## Scope

- Does NOT provision the VM — use `smaqit.infrastructure-provision-cyso` for that.
- Does NOT handle database migrations — current project uses SQLite with append-only schema.
- Does NOT pull from a container registry — deploys local build artifacts directly.
- For Python/FastAPI + Next.js deployments, use `smaqit.infrastructure-deploy-rsync-python-nextjs`.

## Examples

**Input:** Feature branch merged to main; CI/CD workflow runs or operator invokes `/app.deploy`.  
**Output:** Backend and frontend artifacts on VM, container running, nginx serving, health endpoint
returning correct SHA and `deployedAt` timestamp.

## Gotchas

- **Hardcoded source paths** — this skill assumes `backend/` and `frontend/` as local source directories. If the project uses different paths, update steps accordingly and ensure the stack spec declares the same paths.
- **`Cannot find module '/app/dist/index.js'`** — rsync trailing slash puts files at the wrong depth.
  Always `mkdir -p {deploy_path}/backend/dist` before rsyncing `backend/dist/`.
- **`VITE_DEMO_MODE` is build-time only** — changing the GitHub Actions variable and re-running the
  workflow triggers a rebuild. Changing it in the running environment has no effect.
- **Deploy stamp path** — write to `{deploy_path}/backend/DEPLOY_SHA` and `{deploy_path}/backend/DEPLOY_TIME`,
  not `{deploy_path}/`. Files in `{deploy_path}/` are invisible to the container.
- **`docker compose` vs `docker-compose`** — use `docker compose` (v2, no hyphen) on Ubuntu 24.04
  with Docker 24+.
- **`--force-recreate`** — required even when no image changed; use `up -d --force-recreate` NOT `restart`.
- **Production port mappings** — production docker-compose.yml must expose ports to `127.0.0.1` for
  nginx on the host to reach containers (e.g., `127.0.0.1:8000:8000`, `127.0.0.1:3000:3000`).
- **Database migrations** — if the project uses Alembic or similar, run migrations after deploy.
  Skipping causes 500 errors on unmigrated tables.

## Completion

- [ ] Backend TypeScript build succeeded
- [ ] Frontend Vite build succeeded
- [ ] Backend artifacts rsynced to `{deploy_path}/backend/dist/`
- [ ] `node_modules` installed via Docker on VM
- [ ] Frontend dist rsynced to `{deploy_path}/frontend/`
- [ ] `docker-compose.yml` and nginx config transferred
- [ ] Deploy stamp files written to `{deploy_path}/backend/`
- [ ] Container restarted with `--force-recreate`
- [ ] nginx reloaded without errors
- [ ] `smaqit.infrastructure-deploy-verify` invoked and passed

## Failure Handling

| Situation | Action |
|-----------|--------|
| Required input not provided | Request the missing information before proceeding |
| Gathered input is ambiguous | Flag the ambiguity and ask for clarification |
| Subagent invocation fails | Report the failure with context; do not silently retry |
| Output artifact already exists | Confirm with user before overwriting |
| `npm run build` fails | Show the build error; stop — do not deploy broken artifacts |
| rsync permission denied | Check SSH key and `ubuntu` user write permissions on `{deploy_path}/` |
| `Cannot find module` in container logs | Check rsync target depth; re-run step 4 with explicit `mkdir -p` |
| nginx config test fails | Show the nginx error; do not reload — current config remains active |
| deploy-verify reports SHA mismatch | Check `docker ps`; old container may still be running; retry step 10 |
