---
name: smaqit.infrastructure-provision-cyso
description: Use when provisioning cloud infrastructure for the HIM Corporate application on Cyso Cloud (OpenStack) using Terraform. Covers application credential sourcing, Object Storage backend initialization, SSH keypair variable configuration, `terraform init/plan/apply`, and fixed IP retrieval. Produces a running Cyso VM accessible via SSH, with Cinder data volume attached and security group configured on ports 22/80/443. Also use when re-running Terraform after infrastructure changes or when an operator invokes `/provision.cyso`.
metadata:
  version: "1.1.0"
---

# Provision Target: Cyso Cloud

## Pre-conditions (one-time manual steps — complete before first run)

- Cyso Cloud account with access to region `ams2`
- Application credential created in Cyso Cloud Portal → Access → Credentials; credential ID and secret loaded into Vault at `secret/<project-slug>/cyso`
- Object Storage state bucket created in Cyso dashboard (private); S3 access key + secret key loaded into Vault at `secret/<project-slug>/tfstate`
- SSH keypair generated (passphrase-free) and loaded into Vault at `secret/<project-slug>/ssh` (both private and public key fields)
- Fine-grained GitHub PAT with `variables:write` loaded into Vault at `secret/<project-slug>/github`
- Terraform 1.14+ installed locally (`sudo apt install -y terraform` via HashiCorp repo; already configured if Vault was installed from same repo)
- Local Vault running and unsealed (`smaqit.infrastructure-vault-loader` complete)

<!-- amendment: 2026-05-25 — credential sourcing moved from OpenRC file + manual exports to local Vault (smaqit.infrastructure-vault-loader). SSH key no longer stored at ~/.ssh/him_deploy_key. OpenRC file no longer required. -->

## Steps

1. **Unlock Vault:**
   Invoke `smaqit.infrastructure-vault-loader` and confirm Vault is running, unsealed, and all
   `secret/<project-slug>/*` paths are populated. Do not proceed until this is confirmed.

2. **Fetch credentials from Vault into shell environment:**
   ```bash
   export VAULT_ADDR=http://127.0.0.1:8200
   export PROJECT_SLUG=<project-slug>   # from copilot-instructions.md

   export TF_VAR_app_credential_id=$(vault kv get -field=app_credential_id secret/${PROJECT_SLUG}/cyso)
   export TF_VAR_app_credential_secret=$(vault kv get -field=app_credential_secret secret/${PROJECT_SLUG}/cyso)
   export TF_VAR_github_token=$(vault kv get -field=token secret/${PROJECT_SLUG}/github)
   export TF_VAR_ssh_public_key=$(vault kv get -field=public_key secret/${PROJECT_SLUG}/ssh)
   export AWS_ACCESS_KEY_ID=$(vault kv get -field=access_key secret/${PROJECT_SLUG}/tfstate)
   export AWS_SECRET_ACCESS_KEY=$(vault kv get -field=secret_key secret/${PROJECT_SLUG}/tfstate)
   ```
   Use `TF_VAR_github_token` — not `GITHUB_TOKEN` (reserved by Actions) and not `TF_VAR_GITHUB_TOKEN`
   (case-sensitive; Terraform maps `TF_VAR_github_token` → `var.github_token`). The public key is
   already newline-stripped at Vault load time — do not `tr -d '\n'` again.

3. **Confirm Ubuntu image ID and flavor** (catalog values change occasionally):
   ```bash
   openstack image list | grep "Ubuntu 24.04"
   openstack flavor list
   ```
   If IDs differ from the defaults in `deployment/terraform/main.tf`, update that file before continuing.
   Verified reference values (2026-04-05):
   - Image: `fd91e198-f162-4b6b-a23e-123304fb408a` (Ubuntu 24.04 LTS)
   - Flavor: `s5.small` (2 vCPU, 8 GB RAM, 50 GB disk, €17.50/mo)
   Note: `openstack` CLI must be available; auth is implicit via `TF_VAR_app_credential_*` env vars
   set above — no OpenRC file is required.

4. **Navigate to Terraform directory:**
   ```bash
   cd deployment/terraform
   ```

5. **Initialize Terraform:**
   ```bash
   terraform init
   ```
   Confirms remote state backend connectivity. `.terraform.lock.hcl` is committed; if regeneration is
   needed: `terraform providers lock -platform=linux_amd64 -platform=darwin_arm64`.

6. **Review plan.** Expected resources on first apply: 1 Nova VM, 1 boot volume (20 GB),
   1 security group (ports 22/80/443), 1 keypair, 1 GitHub Actions variable.
   ```bash
   terraform plan
   ```

7. **Apply:**
   ```bash
   terraform apply
   ```
   After apply, note the `fixed_ip` output — this is the public address. The floating IP is provisioned
   but does not route on Cyso's flat network; ignore it.

8. **Verify SSH access** (should succeed within 60 seconds of apply):
   ```bash
   # Fetch SSH private key to a temporary file
   TMPKEY=$(mktemp) && trap "rm -f $TMPKEY" EXIT
   vault kv get -field=private_key secret/${PROJECT_SLUG}/ssh > "$TMPKEY"
   chmod 600 "$TMPKEY"
   ssh -i "$TMPKEY" ubuntu@<fixed_ip>
   ```

## Output

- Cyso VM running and SSH-accessible at `fixed_ip`
- Cinder data volume attached (appears as `/dev/sdb` inside VM)
- Security group open on ports 22, 80, 443
- Terraform state stored remotely in `him-corporate-tfstate` Object Storage bucket

## Scope

- Does NOT bootstrap the VM post-provision — use `smaqit.infrastructure-vm-bootstrap` for that.
- Does NOT deploy the application — use `smaqit.infrastructure-deploy-rsync` for that.
- Does NOT configure nginx or Docker inside the VM (handled by cloud-init user-data in `main.tf`).
- Floating IP is non-functional on Cyso's flat network — use `outputs.fixed_ip` exclusively.

## Examples

**Input:** Cyso account set up, OpenRC file ready, state bucket created. Operator invokes `/provision.cyso`.

**Output:** VM at `fixed_ip` (e.g. `81.24.10.203`), Cinder data volume attached as `/dev/sdb`, SSH
accessible with `him_deploy_key`, Terraform state in `him-corporate-tfstate`. Ready for
`smaqit.infrastructure-vm-bootstrap`.

## Gotchas

- **Floating IP does not route** — Cyso assigns publicly-routable IPs directly to VM interfaces on the
  flat network. `openstack_networking_floatingip_associate_v2` does not make it publicly accessible.
  Always use `outputs.fixed_ip`; never the floating IP value.

- **Data volume is `/dev/sdb`, not `/dev/vdb`** — Cyso presents Cinder volumes as SCSI devices, not
  virtio-blk. `lsblk` shows it as `sdb` (or `sdc` if multiple volumes are attached).

- **Keypair trailing-newline drift** — Terraform will show the keypair needing replacement on a
  non-first apply if the stored key string differs by a trailing newline. Strip the newline when
  loading into Vault:
  ```bash
  vault kv put secret/<slug>/ssh public_key="$(cat ~/.ssh/<key>.pub | tr -d '\n')" ...
  ```
  The Vault fetch (`vault kv get -field=public_key`) does not add a newline, so subsequent
  applies are stable. The variable name is `TF_VAR_ssh_public_key` (all lowercase) — matches
  `var.ssh_public_key` in `variables.tf`.

- Use `endpoints.s3` (plural), NOT `endpoint` (deprecated in Terraform 1.15+).
  `skip_requesting_account_id = true` is REQUIRED for non-AWS S3. `use_path_style` replaces
  deprecated `force_path_style`. The state bucket must be pre-created in the Cyso dashboard.

- **Provider credential passing:** The OpenStack provider does NOT auto-detect `TF_VAR_*` env vars.
  Pass credentials explicitly: `application_credential_id = var.app_credential_id` and
  `application_credential_secret = var.app_credential_secret` in the provider block.
  GitHub provider also needs explicit `token = var.github_token` and `owner = "<username>"`.

- **Terraform install:** May not be pre-installed. If `command not found`:
  `sudo apt update && sudo apt install -y terraform`. The HashiCorp apt repo is already
  configured if Vault was installed from it.

- **Provider lock file platform mismatch** — `.terraform.lock.hcl` is committed with hashes for
  `linux_amd64` and `darwin_arm64`. Regenerate if running on a different platform:
  `terraform providers lock -platform=<platform>`.

- **Application credential must be project-scoped** — a user-level credential without project scope
  will fail on resource creation.

- **Do not run `terraform destroy`** — it tears down live infrastructure. Not part of any deployment
  or acceptance step.

## Completion

- [ ] OpenRC sourced; `openstack token issue` succeeded
- [ ] Ubuntu image ID and flavor confirmed; `variables.tf` updated if needed
- [ ] Backend variables set (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
- [ ] Terraform provider variables set (correct casing; `TF_VAR_github_token`; ssh key newline stripped)
- [ ] `terraform init` succeeded with remote state backend
- [ ] `terraform plan` reviewed; expected resource count confirmed
- [ ] `terraform apply` succeeded; `fixed_ip` noted from outputs
- [ ] SSH access to VM confirmed via `him_deploy_key`
- [ ] `.terraform.lock.hcl` committed if regenerated

## Failure Handling

| Situation | Action |
|-----------|--------|
| Required input not provided | Request the missing information before proceeding |
| Gathered input is ambiguous | Flag the ambiguity and ask for clarification |
| Subagent invocation fails | Report the failure with context; do not silently retry |
| Output artifact already exists | Confirm with user before overwriting |
| `openstack token issue` fails | Re-source OpenRC; check if application credential has expired |
| `terraform init` backend connectivity failure | Verify `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`; check `endpoints.s3` and `skip_requesting_account_id` in backend.tf |
| Image ID not found | Run `openstack image list`; update `variables.tf` with current ID |
| Keypair replacement shown in plan | Strip trailing newline; re-export `TF_VAR_ssh_public_key` using `tr -d '\n'` |
| SSH access fails after apply | Verify security group allows port 22; confirm key fingerprint matches uploaded keypair |
