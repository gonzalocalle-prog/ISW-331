# Dockerized GitHub Actions — Patterns with Real Examples

How real projects combine Docker with GitHub Actions. Each section covers one pattern, gives a minimal working example, and links to a repository where you can see the same pattern used in production.

---

## Pattern 1 — Run a Job Inside a Container (`container:`)

**What it is:** Tell GitHub to run every step of a job inside a Docker image you specify, instead of on the bare runner. Checkout, install, build, test — everything happens inside that container.

**When to use it:** You need a controlled toolchain (specific compiler, weird system libraries, exact OS version) and you don't want to maintain a long `apt install` script at the top of every workflow.

### Simple example

```yaml
# .github/workflows/test.yml
name: Test in Container

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: node:20-alpine    # entire job runs inside this image
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

No `setup-node` step needed — Node is already in the image.

**Real-life example:**
- **Repo:** [microsoft/playwright](https://github.com/microsoft/playwright/tree/main/.github/workflows)
- **Why:** Playwright needs exact browser binaries and system fonts. They run jobs in their own `mcr.microsoft.com/playwright:*` images so CI matches what users get when they install Playwright locally.

---

## Pattern 2 — Service Containers (Databases & Queues for Tests)

**What it is:** Sidecar containers that GitHub spins up next to your job — Postgres, Redis, MySQL, RabbitMQ — so integration tests have something real to talk to.

**When to use it:** Almost any time you have integration tests that touch a database, cache, or message queue. This is the closest thing to a "free win" in CI design.

### Simple example

```yaml
# .github/workflows/integration.yml
name: Integration Tests

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: myapp_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/myapp_test
          REDIS_URL: redis://localhost:6379
```

The `--health-cmd` options make GitHub wait until Postgres is actually ready before running steps — no more flaky "connection refused" on the first request.

**Real-life example:**
- **Repo:** [strapi/strapi](https://github.com/strapi/strapi/tree/main/.github/workflows)
- **Why:** Strapi tests against multiple databases (Postgres, MySQL, SQLite). Their workflows spin up service containers in a matrix so the same test suite runs against every supported DB.

---

## Pattern 3 — Build & Push Your Application's Docker Image

**What it is:** Your application's deployable artifact *is* a Docker image. CI builds it, tags it (typically with the commit SHA), and pushes it to a registry. Later jobs deploy by pulling that exact tag.

**When to use it:** You deploy to anything that consumes containers — Kubernetes, ECS, Cloud Run, Azure Container Apps, Fly.io, etc. This is the standard pattern for modern containerized deployments.

### Simple example

```yaml
# .github/workflows/build-and-push.yml
name: Build & Push Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write     # needed to push to ghcr.io
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.sha }}
            ghcr.io/${{ github.repository }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

The `cache-from: type=gha` lines are critical — they hook BuildKit into the GitHub Actions cache, turning a 5-minute cold build into a 30-second incremental build.

**Real-life example:**
- **Repo:** [docker/build-push-action](https://github.com/docker/build-push-action)
- **Why:** Official action from Docker. The README and `.github/workflows/` directory contain end-to-end examples for multi-architecture builds, registry caching, and attestations — the source of truth for this pattern.

---

## Pattern 4 — Build Once, Promote the Same Image Through Environments

**What it is:** Combine Pattern 3 with the GitHub Environments feature. Build the image once, then promote the *same SHA-tagged image* through dev → qa → staging → production. Each environment has its own secrets and (for prod) manual approval.

**When to use it:** Anytime you need confidence that what you tested in staging is bit-for-bit what runs in production. This is the gold standard for safe deployments.

### Simple example

```yaml
# .github/workflows/cicd.yml
name: CI/CD

on:
  push:
    branches: [develop, main]

jobs:
  build-image:
    runs-on: ubuntu-latest
    permissions: { contents: read, packages: write }
    outputs:
      image: ghcr.io/${{ github.repository }}:${{ github.sha }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-dev:
    needs: build-image
    if: github.ref == 'refs/heads/develop'
    environment:
      name: dev
      url: https://dev.myapp.com
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh ${{ needs.build-image.outputs.image }} dev
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}    # dev-scoped secret

  deploy-staging:
    needs: build-image
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.myapp.com
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh ${{ needs.build-image.outputs.image }} staging
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}    # staging-scoped secret

  deploy-prod:
    needs: deploy-staging
    environment:
      name: production       # required reviewers configured in repo settings
      url: https://myapp.com
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh ${{ needs.build-image.outputs.image }} production
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}    # prod-scoped secret
```

Notice three things:
1. The image is built **once** — every deploy job pulls the same `ghcr.io/...@<sha>` tag.
2. `${{ secrets.DEPLOY_TOKEN }}` resolves to a **different value per environment** — same syntax, different secret store.
3. `deploy-prod` waits for `deploy-staging` to succeed *and* for a human reviewer to approve.

**Real-life example:**
- **Repo:** [actions/starter-workflows](https://github.com/actions/starter-workflows/tree/main/deployments)
- **Why:** Official templates from GitHub for deploying containerized apps to AWS, Azure, and GCP using environments + approval gates. The closest thing to a canonical reference.

---

## Pattern 5 — Composite / Custom Docker Action

**What it is:** Package a piece of CI logic as a reusable action defined by a `Dockerfile` and an `action.yml`. Anyone (including other workflows in your repo) can then use it with `uses: ./path/to/action`.

**When to use it:** You have a CI step that's complex enough to deserve its own image — e.g., a security scanner, a custom linter, a deploy tool — and you want to reuse it cleanly.

### Simple example

```yaml
# .github/actions/my-scanner/action.yml
name: 'My Scanner'
description: 'Runs our custom security scan'
inputs:
  severity:
    description: 'Minimum severity to fail on'
    default: 'high'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.severity }}
```

```dockerfile
# .github/actions/my-scanner/Dockerfile
FROM alpine:3.19
RUN apk add --no-cache curl jq
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

```yaml
# .github/workflows/scan.yml
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/my-scanner
        with:
          severity: critical
```

> Note: Docker container actions only work on Linux runners — not Windows or macOS.

**Real-life example:**
- **Repo:** [aquasecurity/trivy-action](https://github.com/aquasecurity/trivy-action)
- **Why:** A widely-used Docker container action for vulnerability scanning. Open the `action.yml` and `Dockerfile` to see exactly how a production-grade custom Docker action is structured.

---

## Quick Decision Guide

| Your situation | Pattern to use |
|---|---|
| Tests need a real database | **Pattern 2** (service containers) |
| Toolchain drift is causing "works on my machine" bugs | **Pattern 1** (job in container) |
| Deploying to Kubernetes / ECS / Cloud Run / Container Apps | **Pattern 3** (build & push) |
| You want safe, gated multi-stage deploys with audit trail | **Pattern 4** (build once, promote) |
| You're packaging reusable CI tooling | **Pattern 5** (custom Docker action) |
| Small static site to Vercel / Netlify | **None** — `setup-node` and ship |

---
