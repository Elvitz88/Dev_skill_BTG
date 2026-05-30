# CI/CD — Azure DevOps Pipelines → Argo CD (GitOps)
<!-- ไปป์ไลน์ CI/CD บน Azure DevOps ต่อกับ Argo CD แบบ GitOps -->

Read this when setting up or reviewing CI/CD. The Betagro standard is build/deploy **as code**
(Infrastructure as Code, Technical Req 27) with **separate environments** (Infra Req 4).

## Pipeline stages (mandatory order)
<!-- ลำดับ stage ของ pipeline ที่ต้องมี -->

```
CodeQuality (knip) ──┐
                     ├──→ Build (Docker) ──→ Update (GitOps manifest)
                     │
                     └── (parallel — block Build if knip fails)
```

- **CodeQuality** — `knip` dead-code scan; publishes `knip-results` artifact. Fail on unused files/deps.
- **Build** — short commit hash extraction → Docker build → push to GCP Artifact Registry (2 tags: `<env>-latest` + `<env>-<commit>`).
- **Update** — fix CRLF on script → run `updateK8smanifest.sh` on self-hosted pool → GitOps manifest repo updated → **Argo CD** syncs to cluster.

Additional CI checks run in other pipelines:
- **lint** — ESLint incl. `eslint-plugin-jsx-a11y`; fail on warnings where possible (Technical Req 18, 31).
- **test** — unit + E2E; enforce coverage ≥ 70% (Technical Req 9).
- **audit** — `npm audit --omit=dev` + Checkmarx gate (Technical Req 11).

## GitOps handoff to Argo CD
<!-- ส่งต่อให้ Argo CD แบบ GitOps -->

The pipeline does NOT `kubectl apply` directly. Instead:

1. Pipeline builds + pushes the image with an immutable tag (`<env>-<gitSHA>`).
2. Pipeline runs `updateK8smanifest.sh` to patch the image tag in the **GitOps manifest repo**.
3. **Argo CD** watches the manifest repo and syncs the desired state to the cluster.
4. Promotion between environments = a commit/PR moving a tag from Dev → QAS → Prod.

This keeps the cluster state declarative and auditable, and keeps secrets out of the pipeline.

## Environment separation (Infra Req 4)
<!-- แยก environment ให้ชัดเจน -->

Three independent environments, each with isolated config, secrets, DB, and domain:

| Environment | Branch / trigger        | Domain                | Access                          |
|-------------|-------------------------|-----------------------|---------------------------------|
| Development | feature → `develop`     | dev subdomain         | BTG network / BTG-only / login  |
| QAS         | `release/*` or `qas`    | qas/uat subdomain     | BTG network / BTG-only / login  |
| Production  | `main` (approval gate)  | prod subdomain (WAF)  | public per scope, behind WAF    |

- Secrets live in a secrets store (Azure Key Vault / sealed secrets) — never in code or YAML.
- Dev/QA/UAT must NOT be publicly accessible (Technical Req 8).
- Prod subdomain must work with the Betagro WAF (Security Req 9).

Separate pipeline YAML files per environment:
```
pipelines/
├── azure-pipeline-dev.yaml
├── azure-pipeline-scanning.yaml
├── azure-pipelines-qa.yml
├── azure-pipelines-uat.yml
└── azure-pipelines-prod.yml
```

## Repo split (Technical Req 12 / history note)
<!-- แยก repo ตามมาตรฐาน -->

Separate `frontend`, `backend`, and `device` into different repos. Each repo has:
- its own README (incl. automated local-setup instructions — Technical Req 15),
- its own `azure-pipelines.yml`,
- stale branches deleted (keep env-deploy + main — Technical Req 7).

## Starter template

An Azure Pipelines starter is provided at `assets/azure-pipelines-template.yml`. Copy it into the
repo root as `azure-pipelines.yml` and adapt the variables section to the project.
Verify every tool/library it references is on the GT&D-approved list before committing.

**Required variables to set per project:**

| Variable | Description | Example |
|----------|-------------|---------|
| `containerRegistryConection` | Azure DevOps service connection name | `app-nonprod-registry` |
| `imageName` | Docker image name | `btg-labware-auth-service` |
| `GCP_PROJECT_ID` | GCP project ID | `app-nonprod-project` |
| `GCP_REPO` | GCP Artifact Registry repo name | `app-nonprod-ar` |
| `pool` | Self-hosted agent pool (for Update stage) | `GCP-NONPRD-POOL` |
| `k8sManifestRepoUrl` | SSH URL of the GitOps manifest repo | `git@ssh.dev.azure.com:v3/betagro-dev/...` |
| `deploymentFileName` | Name of the deployment YAML to patch | `authservicedev-deployment.yaml` |
| `environment` | Environment name | `dev` / `qa` / `prod` |

## Real-project gotchas
<!-- ปัญหาที่เจอในโปรเจกต์จริง — อ้างอิงจาก btg-labware-auth-service -->

### 1. CRLF line endings in `updateK8smanifest.sh`
**Problem:** If `updateK8smanifest.sh` is edited on Windows, it gets `\r\n` line endings. The script
will fail on Linux with a cryptic `bad interpreter` or `command not found` error.

**Fix:** Always add a "Fix Script Line Endings" step before running the script:
```yaml
- task: Bash@3
  displayName: 'Fix Script Line Endings (CRLF → LF)'
  inputs:
    targetType: 'inline'
    script: |
      sed -i 's/\r$//' $(System.DefaultWorkingDirectory)/$(updateK8sScriptFile)
      chmod +x $(System.DefaultWorkingDirectory)/$(updateK8sScriptFile)
```
This step is included in `assets/azure-pipelines-template.yml`.

### 2. `deploymentFileName` — 5th argument to `updateK8smanifest.sh`
**Problem:** Boilerplate calls the script with 4 args. In multi-service repos (or when the manifest
repo has multiple deployment files), the script needs to know **which** deployment YAML to patch.

**Fix:** Pass `deploymentFileName` as the 5th argument:
```yaml
arguments: '$(environment) $(environment)-$(Build.SourceVersion) $(containerImageRepository) $(k8sManifestRepoUrl) $(deploymentFileName)'
```
Also expose it as `DEPLOYMENT_FILE_NAME` env var for scripts that read env instead of positional args.

### 3. `containerRegistryConection` spelling
The boilerplate uses `app-nonprd-registry` but real projects may use `app-nonprod-registry`
(without the abbreviation). Double-check the Azure DevOps service connection name — it must
match exactly, including spelling.

### 4. CodeQuality stage — parallel vs. blocking
The template runs CodeQuality in parallel with Build (`dependsOn: []`). To **block** Build on
knip failures, change Build stage to `dependsOn: [CodeQuality]`. Current real-project pipelines
(e.g. btg-labware-auth-service) skip CodeQuality and go straight to Build — add it back to
comply with the full standard.

## DataDog logging in the deployed app (Technical Req 21)

- Use **Winston** with log levels + rotation; ship to Betagro DataDog.
- No `console.log` in shipped code.
- Instrument for monitoring + condition-based alerting (Infra Req 11) so failures are caught before
  users complain.
