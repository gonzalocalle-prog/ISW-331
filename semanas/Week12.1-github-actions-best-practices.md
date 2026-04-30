# GitHub Actions — Best Practices with Reference Repositories

A curated guide to the most-used GitHub Actions best practices. Each section explains *what* the practice is, *why* it matters, and points to a real open-source repository where you can see it in production.

---

## 1. Reusable Workflows (`workflow_call`)

**What it is:** Define a workflow once and call it from other workflows, the same way you'd call a function. Avoids copy-pasting the same build/test/deploy logic across multiple files.

**Why it matters:** Large repos accumulate dozens of workflows that share 80% of the same steps. Reusable workflows let you fix a bug or upgrade a runner version in one place instead of fifteen.

**How it looks (minimal):**
```yaml
# .github/workflows/reusable-build.yml
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        required: true
```
```yaml
# .github/workflows/ci.yml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
```

**Real-life example:**
- **Repo:** [vercel/next.js](https://github.com/vercel/next.js/tree/canary/.github/workflows)
- **Look at:** `build_reusable.yml` — called from `build_and_test.yml`. This is how Vercel keeps dozens of CI variations DRY.

---

## 2. Pin Actions to a Commit SHA (Supply-Chain Security)

**What it is:** Instead of `uses: actions/checkout@v4`, pin to a full commit hash like `uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11`.

**Why it matters:** Tags are mutable. If an action's maintainer is compromised, an attacker can push a malicious commit and re-tag `v4` to point at it — your workflow would silently pull the bad code on the next run. A SHA is immutable.

**How it looks:**
```yaml
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
- uses: docker/build-push-action@2cdde995de11925a030ce8070c3d77a52ffcf1c0 # v5.3.0
```
The trailing comment preserves human readability.

**Real-life example:**
- **Repo:** [github/docs](https://github.com/github/docs/tree/main/.github/workflows)
- **Look at:** Almost any workflow file — GitHub's own docs team SHA-pins every third-party action. Dependabot keeps them updated automatically.

---

## 3. Cache Dependencies to Speed Up Runs

**What it is:** Use `actions/cache` (or the cache built into `setup-node`, `setup-python`, etc.) to persist `node_modules`, `~/.nuget/packages`, `~/.m2`, etc. between runs.

**Why it matters:** A cold `npm ci` can take 60–120 seconds on every run. With a warm cache, it's 5–10 seconds. Across thousands of CI runs per month, this is the difference between fast feedback and developers waiting around.

**How it looks:**
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'        # automatic caching keyed on package-lock.json
    cache-dependency-path: '**/package-lock.json'
```

**Real-life example:**
- **Repo:** [vitejs/vite](https://github.com/vitejs/vite/tree/main/.github/workflows)
- **Look at:** `ci.yml` — pnpm + Node caching, plus a Playwright browser cache. A clean reference for a fast, modern JS pipeline.

---

## 4. Matrix Builds for Parallel Testing

**What it is:** A single job definition that fans out into many parallel jobs across combinations of OS, runtime version, or shard index.

**Why it matters:** Cuts wall-clock time dramatically. A 20-minute test suite split into 5 shards finishes in ~4 minutes. Also catches "works on my machine" issues across Node 18/20/22 or Ubuntu/macOS/Windows in a single workflow.

**How it looks:**
```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20, 22]
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-node@v4
    with: { node-version: ${{ matrix.node }} }
  - run: npm test
```

**Real-life example:**
- **Repo:** [microsoft/TypeScript](https://github.com/microsoft/TypeScript/tree/main/.github/workflows)
- **Look at:** `ci.yml` — matrix across Node versions and OS, plus test sharding. A textbook example of the pattern at scale.

---

## 5. Cancel Stale Runs with `concurrency`

**What it is:** A workflow-level setting that cancels in-progress runs when a new commit lands on the same branch or PR.

**Why it matters:** Without it, every push to a PR queues another full CI run on top of the previous one. Concurrency control saves runner minutes (= money) and gives faster feedback because the latest commit jumps the queue.

**How it looks:**
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

**Real-life example:**
- **Repo:** [shadcn-ui/ui](https://github.com/shadcn-ui/ui/tree/main/.github/workflows)
- **Look at:** Top of most workflow files — concurrency block scoped per-PR. Standard modern practice.

---

## 6. Path Filters in Monorepos

**What it is:** `paths:` and `paths-ignore:` filters that prevent workflows from running when irrelevant files change.

**Why it matters:** In a monorepo with 10 packages, you don't want the iOS workflow to run because someone edited the docs. Path filters keep CI focused and fast.

**How it looks:**
```yaml
on:
  pull_request:
    paths:
      - 'apps/web/**'
      - 'packages/ui/**'
      - '.github/workflows/web-ci.yml'
```

**Real-life example:**
- **Repo:** [belgattitude/nextjs-monorepo-example](https://github.com/belgattitude/nextjs-monorepo-example/tree/main/.github/workflows)
- **Look at:** Per-app workflow files with `paths:` filters. A clean reference for Yarn workspaces + path-scoped CI.

---

## 7. Least-Privilege Permissions on `GITHUB_TOKEN`

**What it is:** Explicitly declaring the minimum permissions a job needs, instead of relying on the (overly permissive) defaults.

**Why it matters:** A compromised dependency in a job with `contents: write` can push to your repo. With `contents: read` it cannot. Defense in depth — assume something will eventually go wrong.

**How it looks:**
```yaml
permissions:
  contents: read       # default for everything
  pull-requests: write # only the job that needs to comment on PRs

jobs:
  build:
    permissions:
      contents: read   # job-level overrides further restrict
```

**Real-life example:**
- **Repo:** [step-security/harden-runner](https://github.com/step-security/harden-runner/tree/main/.github/workflows)
- **Look at:** Every workflow declares minimal `permissions:` blocks at both workflow and job level — security tooling team eating their own dog food.

---

## 8. Environments + Manual Approval Gates for Deployments

**What it is:** GitHub's built-in `environment:` feature — declare `dev`, `qa`, `staging`, `production` with per-environment secrets, branch restrictions, and required reviewers.

**Why it matters:** Production deploys should not happen automatically on every merge. Environments give you a paved-road way to add manual approval, restrict who can deploy, and scope secrets per stage — all without writing custom approval logic.

**How it looks:**
```yaml
deploy-prod:
  needs: deploy-staging
  environment:
    name: production
    url: https://myapp.com
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh production
      env:
        API_KEY: ${{ secrets.API_KEY }}   # resolves to the prod secret
```
Configure required reviewers in `Settings → Environments → production`.

**Real-life example:**
- **Repo:** [actions/starter-workflows](https://github.com/actions/starter-workflows/tree/main/deployments)
- **Look at:** Official GitHub-maintained templates for AWS / Azure / GCP deploys with environments and approval gates baked in.

---

## Summary Table

| # | Practice | Primary Benefit |
|---|---|---|
| 1 | Reusable workflows | DRY, easier maintenance |
| 2 | SHA-pinned actions | Supply-chain security |
| 3 | Dependency caching | Faster CI runs |
| 4 | Matrix builds | Parallelism + cross-platform coverage |
| 5 | Concurrency control | Save minutes, faster feedback |
| 6 | Path filters | Skip irrelevant work in monorepos |
| 7 | Least-privilege tokens | Reduce blast radius |
| 8 | Environments + approvals | Safe, gated deployments |

---

## How to Study These

1. Pick **two repos** from the list — ideally one small (like `shadcn-ui/ui`) and one large (like `vercel/next.js`).
2. Open `.github/workflows/` in both side by side.
3. For each practice above, find where (or *whether*) the repo applies it.
4. Note what they do differently — that's where the real lessons are.
