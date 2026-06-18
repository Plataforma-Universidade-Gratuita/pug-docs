# CI/CD Guide

This document covers the GitHub Actions workflows currently present in [`pug-service/.github/workflows`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/.github/workflows). It reflects what is actually in the repository today: verification, image build, and image publish. A deployment workflow is not part of the repository.

## Pipeline overview

The repository currently defines three workflows:

- [`verify.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/verify.yml)
- [`build-image.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/build-image.yml)
- [`publish-image.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/publish-image.yml)

```mermaid
flowchart LR
    PR[Pull request]
    MainPush[Push to main]
    TagPush[Push tag v*]
    Manual[workflow_dispatch]

    PR --> Verify[verify.yml]
    PR --> BuildImage[build-image.yml]
    MainPush --> Verify
    TagPush --> Publish[publish-image.yml]
    Manual --> BuildImage
    Manual --> Publish

    Verify --> MavenVerify[./mvnw -B clean verify]
    BuildImage --> DockerBuild[Docker build only]
    Publish --> GhcrPush[Build and push to GHCR]
```

## Workflow triggers

| Workflow | Triggers |
| --- | --- |
| `verify.yml` | every `pull_request`, every `push` to `main` |
| `build-image.yml` | every `pull_request`, `workflow_dispatch` |
| `publish-image.yml` | `push` tags matching `v*`, `workflow_dispatch` |

All three workflows define `concurrency` groups so newer runs cancel older runs for the same ref.

## Verify pipeline

### What it does

[`verify.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/verify.yml) is the main quality gate.

Steps:

1. checkout code
2. install Temurin Java 21
3. make `mvnw` executable
4. run `./mvnw -B clean verify`

### Service containers

The workflow provisions explicit databases:

- PostgreSQL 16 on host port `5434`
- MongoDB 7 on host port `27019`

The workflow disables Quarkus Dev Services and points the app at those service containers through environment variables.

### Build, test, lint, and coverage behavior

Because it runs `clean verify`, this workflow executes everything currently bound in `pom.xml`:

- compile/build through Quarkus Maven plugin
- unit/integration-style tests through Surefire
- Spotless formatting via `spotless:apply` during `validate`
- Checkstyle during `verify`
- SpotBugs during `verify`
- JaCoCo merge, report, and coverage check during `verify`

Important detail: the current build uses `spotless:apply`, not `spotless:check`. In CI that does not help the branch because the workflow does not commit changes; it only means `verify` can fail after reformatting the working tree in the runner.

## Build-image pipeline

### What it does

[`build-image.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/build-image.yml) only verifies that the Docker image can be built.

Steps:

1. checkout code
2. set up Docker Buildx
3. run `docker/build-push-action@v6` with `push: false`

### What it does not do

- it does **not** run Maven tests
- it does **not** run `verify`
- it does **not** publish an image

This workflow is a packaging smoke test, not a quality gate by itself.

## Publish-image pipeline

### What it does

[`publish-image.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/publish-image.yml) builds and publishes a container image to GitHub Container Registry.

Steps:

1. checkout code with full history
2. if the run is a tag build, verify the tagged commit is reachable from `main`
3. install Temurin Java 21
4. make `mvnw` executable
5. read `project.version` and `project.artifactId` from Maven
6. if the trigger is a tag, require `refs/tags/v{project.version}`
7. set up Docker Buildx
8. log in to `ghcr.io` using `secrets.GITHUB_TOKEN`
9. compute Docker tags and labels
10. build and push the image

### Image naming and tags

The workflow publishes to:

```text
ghcr.io/{github.repository_owner}/{project.artifactId}
```

Tag behavior from the current metadata config:

- `latest` and `sha-*` are enabled only when the ref is `refs/heads/main`
- `{artifactId}-{version}` is enabled for tag builds

Practical implication:

- on `v*` tag pushes, the publish workflow produces an artifact/version tag
- it does **not** automatically publish `latest` from a tag ref

## Required secrets and environment variables

### GitHub Actions secrets and permissions

Directly discoverable from the workflows:

- `secrets.GITHUB_TOKEN` is used for GHCR login in `publish-image.yml`
- `publish-image.yml` requests `packages: write` permission

No other GitHub Actions secrets were found in the workflow files.

### Runtime application environment variables

The publish workflow only pushes an image. Runtime environment values are defined by the application profiles, not by a deployment workflow in this repo.

From [`application-prod.properties`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/application-prod.properties) and [`application-qa.properties`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/application-qa.properties), the runtime environment needs:

- `DB_URL`
- `DB_USER`
- `DB_PASS`
- `MONGODB_URI`
- `JWT_SECRET_KEY`
- `PASSWORD_PEPPER`
- `QR_PEPPER`
- `CORS_ORIGINS`

Those variables are required by the application, but this repository does not contain a deployment workflow or manifest that wires them into a runtime platform.

### Verify workflow environment

`verify.yml` sets these explicit CI-only variables:

- `QUARKUS_DATASOURCE_DEVSERVICES_ENABLED=false`
- `QUARKUS_MONGODB_DEVSERVICES_ENABLED=false`
- `QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://localhost:5434/pug_test`
- `QUARKUS_DATASOURCE_USERNAME=pug_test`
- `QUARKUS_DATASOURCE_PASSWORD=pug_test`
- `QUARKUS_MONGODB_CONNECTION_STRING=mongodb://pug_test:pug_test@localhost:27019`

## Docker build path

The current [`Dockerfile`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/Dockerfile) uses a two-stage build:

1. `maven:3.9.9-eclipse-temurin-21` builds the Quarkus application with `./mvnw -B -DskipTests package`
2. `eclipse-temurin:21-jre` runs the produced `target/quarkus-app/`

Operational details:

- the runtime container runs as user `quarkus` with UID `1001`
- it exposes port `8080`
- it starts with `java -jar /app/quarkus-run.jar`

## Test, lint, and coverage responsibilities

| Responsibility | Where it happens today |
| --- | --- |
| compile/build | `verify.yml`, Docker build stage, local Maven |
| JVM test suite | `verify.yml`, local `./mvnw test` / `./mvnw verify` |
| formatting | `verify.yml` through Spotless apply in Maven `validate` |
| Checkstyle | `verify.yml` through Maven `verify` |
| SpotBugs | `verify.yml` through Maven `verify` |
| JaCoCo report and coverage check | `verify.yml` through Maven `verify` |
| image build smoke test | `build-image.yml` |
| image publish | `publish-image.yml` |
| deployment | no deployment workflow in the repository |

## Deployment or publishing steps if present

### Present

- container image build
- container image publish to GHCR

### Not found

The repository does **not** currently contain:

- Kubernetes manifests
- Helm charts
- Docker Compose production deployment workflow
- cloud deployment workflow
- release promotion workflow beyond GHCR image publishing

What was checked:

- `.github/workflows`
- repository root for deploy manifests

## Common pitfalls

- A green `build-image` run does not mean tests passed; that workflow never runs Maven verify.
- Tag publishing is version-sensitive: `publish-image.yml` fails if the pushed tag does not match `v${project.version}` from `pom.xml`.
- `verify.yml` disables Dev Services explicitly, so CI issues can differ from local `./mvnw test`.
- `verify.yml` uses service containers on non-default local ports (`5434`, `27019`), while local dev uses `5433` and `27018`.
- The current publish workflow pushes images, but runtime deployment remains an external concern outside this repository.

## Links

- [Back to README](./README.md)
