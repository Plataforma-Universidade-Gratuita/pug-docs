# PUG Web Admin CI/CD

Back to [README.md](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-web-admin/README.md).

## 🚦 Pipeline overview

`pug-web-admin` uses GitHub Actions to verify the application, validate container builds, publish images to GHCR, and deploy revisions to Azure Container Apps.

Current workflow set:

| Workflow | File | Trigger | Purpose |
| --- | --- | --- | --- |
| Verify | [`verify.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.github/workflows/verify.yml) | reusable `workflow_call` | Run the repository quality gate |
| Build Image | [`build-image.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.github/workflows/build-image.yml) | pull request, push to `main` | Validate that the Docker image builds |
| Publish Image | [`publish-image.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.github/workflows/publish-image.yml) | `workflow_dispatch`, tag push `v*` | Verify, build, and publish a container image to GHCR |
| Deploy QA | [`deploy-qa.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.github/workflows/deploy-qa.yml) | `workflow_dispatch` | Deploy a selected image to the QA Container App |
| Deploy PRD | [`deploy-prd.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.github/workflows/deploy-prd.yml) | `workflow_dispatch` | Deploy `latest` to PRD from a protected version tag |
| Deploy Container App | [`deploy-container-app.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.github/workflows/deploy-container-app.yml) | reusable `workflow_call` | Shared Azure Container Apps deployment implementation |

```mermaid
flowchart TD
    PR[Pull request] --> BuildImage[build-image.yml]
    MainPush[Push to main] --> BuildImage
    ManualPublish[Manual publish from main] --> Publish[publish-image.yml]
    TagPush[Push tag v*] --> Publish

    Publish --> GuardPublish[guard publish ref]
    GuardPublish --> Verify[verify.yml]
    Verify --> BuildValidation[Docker build validation]
    BuildValidation --> ResolveApiUrl[resolve NEXT_PUBLIC_API_URL]
    ResolveApiUrl --> GHCR[push image to GHCR]

    ManualQA[Manual QA deploy] --> DeployQA[deploy-qa.yml]
    DeployQA --> DeployShared[deploy-container-app.yml]
    DeployShared --> AzureQA[Azure Container App QA]

    ManualPRD[Manual PRD deploy from v* tag] --> DeployPRD[deploy-prd.yml]
    DeployPRD --> GuardPRD[guard PRD ref]
    GuardPRD --> DeployShared
    DeployShared --> AzurePRD[Azure Container App PRD]
```

## 🔔 Workflow triggers

### Verify

`verify.yml` is reusable and is called by other workflows.

It does not run directly from pull requests or pushes by itself. It is currently invoked by `publish-image.yml`.

### Build Image

`build-image.yml` validates that the Docker image can be built.

It runs on:

- pull requests
- pushes to `main`

It keeps `push: false`, so it does not publish an image.

### Publish Image

`publish-image.yml` runs on:

- manual `workflow_dispatch`
- pushed tags matching `v*`

Manual publish is only allowed from `main`.

Tag publish is only allowed when the tagged commit is contained in `main`.

### Deploy QA

`deploy-qa.yml` is manual.

It requires a full image reference, for example:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:sha-abc123
```

### Deploy PRD

`deploy-prd.yml` is manual, but guarded.

It only runs when the workflow is started from a version tag matching:

```text
v*
```

The tagged commit must also be contained in `main`.

## 🛠️ Verification

The reusable verify job:

1. checks out the repository
2. sets up Node.js `22`
3. restores npm cache through `actions/setup-node`
4. runs `npm ci`
5. runs `npm run verify`

`npm run verify` expands to:

```bash
npm run format:check && tsc --noEmit && npm run trans && npm run build
```

That means the current quality gate covers:

- Prettier formatting
- ESLint
- TypeScript type-checking
- translation ordering and consistency checks
- production Next.js build success

Relevant files:

- [`package.json`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/package.json)
- [`scripts/reorderTranslations.js`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/scripts/reorderTranslations.js)
- [`scripts/checkMissingTranslations.js`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/scripts/checkMissingTranslations.js)
- [`scripts/checkUnusedTranslations.js`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/scripts/checkUnusedTranslations.js)

## 🐳 Build Image workflow

The image build workflow:

1. checks out the repository
2. sets up Docker Buildx
3. builds the image from [`Dockerfile`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/Dockerfile)
4. keeps `push: false`

This validates container buildability, but it does not publish an image.

## 📦 Publish Image workflow

The publish workflow is the main image release workflow.

It:

1. validates the ref
2. runs the reusable verify workflow
3. validates the Docker image build
4. resolves the public backend API URL for the selected target environment
5. computes image tags
6. logs in to GHCR
7. builds and pushes the image

### Ref guards

Manual publish is only allowed from:

```text
refs/heads/main
```

Tag publish is only allowed when:

```text
refs/tags/v*
```

and the tagged commit is contained in:

```text
origin/main
```

### Image name

The image name is fixed by the workflow:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin
```

### Manual publish tags

Manual publish from `main` publishes:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:sha-<shortsha>
```

Example:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:sha-abc123def456
```

This is the expected image format for QA deployments.

### Version tag publish tags

When a tag like this is pushed:

```text
v1.0.0
```

the workflow publishes:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:1.0.0
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:latest
```

The PRD deploy workflow uses `latest`.

### Provenance and SBOM

The publish step disables Docker provenance and SBOM generation:

```yaml
provenance: false
sbom: false
```

This keeps GHCR package versions simpler and avoids extra untagged package entries.

## 🌐 Build-time API URL resolution

`NEXT_PUBLIC_API_URL` is a browser-facing value.

Because this is a Next.js frontend variable, it is baked into the JavaScript bundle during build time. Changing it only at Azure Container App runtime does not reliably change the already-built client bundle.

The publish workflow accepts:

```text
target_environment: dev | qa | prd
```

Then it resolves one of these repository variables:

| Target environment | Repository variable |
| --- | --- |
| `dev` | `NEXT_PUBLIC_API_URL_DEV` |
| `qa` | `NEXT_PUBLIC_API_URL_QA` |
| `prd` | `NEXT_PUBLIC_API_URL_PRD` |

The resolved value is passed into Docker as:

```text
NEXT_PUBLIC_API_URL=<resolved API URL>
```

### Important note for tag publishes

Tag publishes are normally production releases. If the workflow runs from a tag and no `target_environment` input exists, the workflow should treat the target environment as `prd`.

Recommended behavior:

```yaml
TARGET_ENVIRONMENT: ${{ inputs.target_environment || 'prd' }}
```

## ☁️ Deploy Container App workflow

`deploy-container-app.yml` is a reusable workflow used by both QA and PRD deployment workflows.

It receives:

| Input | Purpose |
| --- | --- |
| `environment_name` | GitHub Environment name, such as `qa` or `prd` |
| `azure_resource_group` | Azure resource group containing the Container App |
| `azure_container_app` | Azure Container App name |
| `image` | Full container image reference to deploy |

It performs these steps:

1. logs in to Azure using `AZURE_CREDENTIALS`
2. ensures the Azure Container Apps extension exists
3. configures GHCR as the Container App registry
4. updates the Container App image
5. sets runtime environment variables
6. prints the deployed URL

Runtime environment values set by the workflow:

```text
NODE_ENV=production
NEXT_TELEMETRY_DISABLED=1
PORT=3000
HOSTNAME=0.0.0.0
NEXT_PUBLIC_API_URL=<environment variable value>
```

## 🚚 QA deployment

`deploy-qa.yml` is manual.

It asks for:

```text
image
```

Example:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:sha-abc123def456
```

Deployment target:

| Setting | Value |
| --- | --- |
| GitHub Environment | `qa` |
| Azure resource group | `rg-pug-qa` |
| Azure Container App | `pug-web-admin-qa` |

Typical QA flow:

```mermaid
flowchart LR
    Main[main branch] --> Publish[Manual Publish Image]
    Publish --> ShaImage[sha image in GHCR]
    ShaImage --> DeployQA[Manual Deploy QA]
    DeployQA --> QA[pug-web-admin-qa]
```

## 🚀 PRD deployment

`deploy-prd.yml` is manual but protected.

It validates:

1. the workflow is running from a `v*` tag
2. the tagged commit is contained in `main`

It deploys:

```text
ghcr.io/plataforma-universidade-gratuita/pug-web-admin:latest
```

Deployment target:

| Setting | Value |
| --- | --- |
| GitHub Environment | `prd` |
| Azure resource group | `rg-pug-prd` |
| Azure Container App | `pug-web-admin-prd` |

Typical PRD flow:

```mermaid
flowchart LR
    Main[main commit] --> Tag[v1.0.0 tag]
    Tag --> Publish[Publish Image]
    Publish --> Latest[latest image]
    Latest --> DeployPRD[Manual Deploy PRD]
    DeployPRD --> PRD[pug-web-admin-prd]
```

## 🔐 Required secrets and variables

### GitHub secrets

| Secret | Used by | Purpose |
| --- | --- | --- |
| `GITHUB_TOKEN` | `publish-image.yml` | automatic GitHub token used to push to GHCR |
| `AZURE_CREDENTIALS` | `deploy-container-app.yml` | Azure service principal JSON |
| `GHCR_USERNAME` | `deploy-container-app.yml` | username used by Azure to pull from GHCR |
| `GHCR_TOKEN` | `deploy-container-app.yml` | token used by Azure to pull from GHCR |

`GITHUB_TOKEN` is provided automatically by GitHub Actions.

`GHCR_TOKEN` should have permission to read packages.

### Repository variables

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_API_URL_DEV` | backend API URL baked into dev-target images |
| `NEXT_PUBLIC_API_URL_QA` | backend API URL baked into QA-target images |
| `NEXT_PUBLIC_API_URL_PRD` | backend API URL baked into PRD-target images |

These values are public by design because `NEXT_PUBLIC_*` values are visible in the browser bundle.

### Environment variables

Each GitHub deployment environment should define:

| Environment | Variable |
| --- | --- |
| `qa` | `NEXT_PUBLIC_API_URL` |
| `prd` | `NEXT_PUBLIC_API_URL` |

The deploy workflow sets this runtime value on the Azure Container App revision.

## 🧪 Tests, lint, and coverage status

What the pipeline enforces today:

- formatting
- linting
- type-checking
- translation checks
- production build success
- Docker image build success
- image publishing
- manual QA deployment
- guarded manual PRD deployment

Repository scope currently excludes:

- a dedicated automated test script in [`package.json`](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/package.json)
- coverage reporting
- a coverage threshold gate
- preview environment provisioning
- post-deploy smoke tests

This repository treats `npm run verify` as its quality gate instead of a dedicated test suite.

## Common pitfalls

- `NEXT_PUBLIC_API_URL` must be correct at image build time, not only at deployment runtime.
- Manual publish is only allowed from `main`.
- Manual QA deploy requires the full image tag, usually `sha-<shortsha>`.
- PRD deploy uses `latest`, so the version tag publish must complete successfully before deploying PRD.
- PRD deploy must be started from a `v*` tag whose commit is contained in `main`.
- A green Docker build only proves the container builds; it does not mean a deployment occurred.

## Links

- [Back to README](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-web-admin/README.md)
- [`pug-web-admin` repository](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin)
- [`pug-web-admin` workflows](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/.github/workflows)