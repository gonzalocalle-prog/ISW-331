# Week 12 - CI/CD with GitHub Actions

> **Block 4: DevOps and CI/CD**
> This week covers why teams automate integration and delivery, how CI/CD reduces risk, and how GitHub Actions models that automation as workflows, jobs, and steps. The goal is not only to run a pipeline, but to understand what problem each stage solves and what engineering habits a good pipeline reinforces.

---

## Part 1 - Why CI/CD Exists

### CI/CD is a response to a scaling problem

When a project is small and only one person works on it, it is possible to survive with informal habits:

- Code is merged when "it seems ready"
- Testing happens manually and inconsistently
- Deployments depend on memory or private notes
- Bugs are discovered late, often by users

This breaks down quickly when a team grows, deadlines tighten, or the application becomes important to real users.

The root problem is not just "we need more automation." The real problem is this:

> **Without a repeatable feedback system, software quality depends too much on human memory, luck, and heroics.**

CI/CD exists to replace fragile, manual release habits with a system that is:

- Predictable
- Observable
- Repeatable
- Fast enough to support frequent change

### The hidden costs of manual integration

Imagine three developers working on the same product. Each one finishes a feature locally. None of them runs exactly the same checks. They merge late. Deployments happen by hand on Friday night.

What usually follows?

- Merge conflicts become larger because changes were integrated too late
- Bugs are discovered after the code is already on the shared branch
- Nobody trusts deployments, so releases become rare and stressful
- The team starts treating deployment as a special event instead of a normal engineering activity

CI/CD attacks those costs directly.

### CI/CD as a feedback loop

At its core, CI/CD is not a product feature. It is a feedback architecture.

```text
Code change
   -> Automated validation
   -> Build/package
   -> Release decision
   -> Deployment
   -> Production feedback
```

The shorter and more reliable this loop becomes, the easier it is to improve software safely.

> **Key insight:** CI/CD is not "automation for automation's sake." It is a way to shorten the time between a code change and trustworthy feedback.

---

## Part 2 - CI, Continuous Delivery, and Continuous Deployment

People often use "CI/CD" as a single phrase, but it actually combines related ideas.

| Term | Core Question | What It Means |
|------|---------------|---------------|
| **Continuous Integration (CI)** | "Can this change join the shared codebase safely?" | Developers merge frequently and automated checks validate each change |
| **Continuous Delivery** | "Can this version be released at any time?" | The team keeps the software in a deployable state, even if production release still needs approval |
| **Continuous Deployment** | "Should every validated change go directly to production?" | Every successful change is deployed automatically with no manual release gate |

### Why the distinction matters

These are not maturity labels where every team must reach the final stage immediately.

- CI is the foundation
- Continuous Delivery is often the best default for early-stage teams
- Continuous Deployment only makes sense when tests, rollback, monitoring, and operational confidence are strong enough

In other words, a team should not automate deployment to production just because the platform allows it.

> **Good engineering principle:** Automate only the decisions that your system is reliable enough to make.

### The promises of a healthy pipeline

A good CI/CD pipeline should give the team four things:

- **Fast feedback** so mistakes are discovered close to the change that caused them
- **Consistency** so the same checks happen every time, not only when someone remembers
- **Visibility** so the repository shows whether the software is healthy
- **Confidence** so releases can happen in small, safe increments

---

## Part 3 - A Pipeline Is a Set of Quality Gates

Students often think of a pipeline as "a script that runs commands in the cloud." That is too shallow.

A better mental model is this:

> **A pipeline is a sequence of quality gates that answers whether a change is trustworthy enough to move forward.**

Here is a common flow:

```text
Pull Request or Push
        |
        v
   Install dependencies
        |
        v
      Lint
        |
        v
   Run tests
        |
        v
      Build
        |
        v
  Package artifact/image
        |
        v
 Deploy to environment
```

Each stage answers a different question:

| Stage | Question It Answers |
|-------|---------------------|
| **Lint / format / typecheck** | Does the code respect agreed quality and consistency rules? |
| **Unit or integration tests** | Does the behavior still work? |
| **Build** | Can the project produce a valid deployable output? |
| **Package** | Can we create a reproducible artifact from this version? |
| **Deploy** | Can that artifact run correctly in the target environment? |

### Why this matters conceptually

If every stage exists for a clear reason, the pipeline becomes easier to maintain.

If stages are added without a clear purpose, the pipeline becomes noisy, slow, and hard to trust.

That is why a strong pipeline is not defined by how many tools it runs. It is defined by whether every stage protects the team from a real class of failure.

---

## Part 4 - GitHub Actions: The Core Mental Model

GitHub Actions is GitHub's automation platform. It lets a repository react to events and execute workflows.

The most useful mental model looks like this:

```text
GitHub event
   -> Workflow
      -> Job(s)
         -> Step(s)
            -> Action(s) or shell commands
```

### The building blocks

| Concept | What It Is | Why It Matters |
|---------|------------|----------------|
| **Workflow** | A YAML file in `.github/workflows/` | Defines the overall automation process |
| **Event / trigger** | The thing that starts the workflow, such as `push` or `pull_request` | Connects repository activity to automation |
| **Job** | A group of steps that runs on a runner | Lets you separate responsibilities and control dependencies |
| **Step** | One task inside a job | The smallest visible unit of work in the job log |
| **Action** | A reusable unit of automation, often shared by others | Prevents rewriting common setup logic |
| **Runner** | The machine that executes the job | Defines the execution environment |
| **Artifact** | A file produced and stored by the workflow | Lets later jobs or humans inspect outputs |
| **Secret** | Encrypted value injected at runtime | Protects credentials and tokens |
| **Environment** | A named deployment target with optional approvals and scoped secrets | Adds release control and safety |
| **Matrix** | A strategy for repeating one job across several combinations | Useful for testing multiple versions or platforms |

### Jobs versus steps: a very important distinction

This is one of the most common sources of confusion.

- Steps inside the **same job** share the same runner and filesystem state
- Different **jobs** do not automatically share state
- Jobs can run in parallel unless you connect them with `needs`

That means:

- Install dependencies in one step, then test in the next step of the same job if they share the same setup
- Use separate jobs when you want stronger isolation, parallelism, or a clear stage boundary
- Use artifacts when one job must pass generated files to another

> **Practical insight:** A job is a unit of environment and responsibility. A step is a unit of procedure.

---

## Part 5 - Anatomy of a GitHub Actions Workflow

Here is a minimal CI workflow for a JavaScript project:

```yaml
name: ci

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

### How to read this file conceptually

- `name` gives the workflow a recognizable label in GitHub
- `on` defines the events that trigger it
- `jobs` is the top-level map of work units
- `validate` is one job that runs on a Linux runner
- Each `step` either uses a reusable action or runs a command

This file is small, but it already expresses several important ideas:

- Every pull request gets checked before merge
- The shared branch gets checked again after merge
- Validation is automated and visible to the team
- The same environment and commands are used repeatedly

### Why YAML structure matters

The YAML syntax is not the most important part. The structure is.

When students learn GitHub Actions, they often memorize snippets without understanding the hierarchy. That creates confusion later when a workflow grows.

Always identify these four levels first:

1. What event starts this workflow?
2. Which jobs should exist?
3. Which steps belong together inside each job?
4. Which dependencies between jobs must be explicit?

If you can answer those four questions, writing the YAML becomes much easier.

---

## Part 6 - Designing Good CI with GitHub Actions

### Start with the repository event, not the command list

The first design question is not "what commands should I run?"

The first design question is:

> **At what repository moments do I need trustworthy feedback?**

For most teams, the answer is:

- On every pull request, before merging
- On pushes to the protected main branch
- Sometimes on manual demand with `workflow_dispatch`

### What a healthy CI workflow usually includes

| Check | Why It Exists |
|-------|---------------|
| **Install with lockfile-aware command** | Ensures reproducible dependency resolution |
| **Lint / static analysis** | Catches style and some correctness issues early |
| **Tests** | Protects behavior and reduces regression risk |
| **Build** | Verifies the project can actually produce output from source |
| **Optional security scan** | Surfaces dependency or configuration risk |

### Principles worth teaching explicitly

- **Fail early:** cheap checks should run before expensive ones when possible
- **Keep feedback fast:** if CI takes too long, people stop trusting or respecting it
- **Be deterministic:** the same commit should behave the same way every time
- **Keep validation separate from deployment:** not every successful test run should imply production release
- **Protect the main branch:** GitHub branch protection and required status checks turn CI from "advice" into policy

### Branch protection changes the meaning of CI

Without branch protection, CI is informative.

With branch protection, CI becomes a governance mechanism.

That difference matters. A green check only improves team behavior when merge rules actually depend on it.

---

## Part 7 - Continuous Delivery and Deployment in GitHub Actions

Once CI proves that a change is healthy, the next question is whether that change can move toward release.

### Delivery versus deployment in practice

| Model | What Happens After Validation |
|-------|-------------------------------|
| **Continuous Delivery** | The build is ready to release, but a human still decides when production deployment happens |
| **Continuous Deployment** | The validated build is released automatically to production |

For most student projects, continuous delivery is the safer conceptual target. It teaches discipline without pretending that production risk is trivial.

### Good delivery design principles

- Build once, deploy the same artifact forward
- Separate environments such as staging and production
- Use environment-specific secrets instead of hardcoding values
- Add approvals for sensitive targets
- Design rollback before the first failed deployment happens

### GitHub Actions concepts that support safer delivery

| GitHub Actions Feature | Conceptual Use |
|------------------------|----------------|
| **`needs`** | Enforces that deployment waits for validation |
| **`environment`** | Adds target-specific control, approvals, and secrets |
| **`secrets`** | Keeps credentials out of source control |
| **`permissions`** | Limits what the workflow token is allowed to do |
| **`concurrency`** | Prevents overlapping deployments to the same target |
| **Artifacts** | Lets jobs reuse a single built output rather than rebuilding differently per stage |

### Example deployment workflow shape

```yaml
name: deploy

on:
  push:
    branches:
      - main

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Run lint, tests, and build here"

  deploy:
    runs-on: ubuntu-latest
    needs: validate
    environment: production

    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploy only after validation succeeds"
```

The important concept is not the placeholder command. It is the relationship:

- `deploy` does not run independently
- The `production` environment can enforce approvals or scoped secrets
- The repository event is tied to a controlled release path

---

## Part 8 - GitHub Actions Features That Matter Most Conceptually

When students first open GitHub Actions documentation, it is easy to get lost in syntax. Focus on the features that change system behavior.

| Feature | Conceptual Meaning | Typical Use |
|---------|--------------------|-------------|
| **`workflow_dispatch`** | Manual invocation | Run a pipeline on demand for demos, backfills, or controlled releases |
| **Matrix strategy** | One logical job, many runtime combinations | Test multiple language versions or OS targets |
| **Reusable workflows** | Standardized automation across repositories | Apply the same CI policy to several projects |
| **Composite actions** | Reusable step bundles | Reduce duplication inside a repository or organization |
| **Cache** | Performance optimization, not correctness | Speed up dependency installation |
| **Artifacts** | Durable workflow outputs | Share build files, test reports, or logs |
| **Environments** | Deployment governance | Add protection rules and target-specific secrets |
| **`permissions`** | Least privilege for automation | Reduce the blast radius of compromised workflows |

### Two subtleties worth teaching

#### Caching does not equal reproducibility

Caching makes workflows faster, but it does not make them more correct. A bad pipeline can still be bad and fast.

Use caching to optimize a stable process, not to hide an unreliable one.

#### Reusable workflows are a scaling tool

Once multiple repositories need the same CI rules, copy-pasting YAML becomes a maintenance problem. Reusable workflows let a team define shared policy once and consume it in many repos.

That is not just a convenience feature. It is a governance feature.

---

## Part 9 - Common CI/CD Mistakes

### Mistake 1: Treating the pipeline as a random script bucket

If commands are added without a clear stage purpose, the workflow becomes hard to reason about.

### Mistake 2: Mixing validation and deployment carelessly

Running tests and deploying production in the same uncontrolled flow increases risk, especially on every branch.

Validation and release can be connected, but they should not be conceptually blurred.

### Mistake 3: Rebuilding differently for each environment

If staging and production use different builds, then staging stops being a trustworthy rehearsal.

The safer model is: build once, promote the same artifact.

### Mistake 4: Storing secrets in the repository

Secrets belong in GitHub Secrets or environment-scoped secrets, never in source code or committed config files.

### Mistake 5: Designing a pipeline nobody wants to run

If CI is extremely slow, noisy, or flaky, developers learn to work around it instead of trusting it.

Pipeline quality is not only about what it checks. It is also about whether the team can live with it every day.

### Mistake 6: Believing a green pipeline means the system is perfect

A green workflow only means the checks that were written passed successfully. It does not prove the absence of bugs.

CI/CD raises confidence. It does not create certainty.

---

## Part 10 - Suggested Exercise and Deliverable

### Suggested in-class exercise

Build one GitHub Actions workflow that does all of the following:

1. Runs on `pull_request`
2. Installs dependencies in a reproducible way
3. Executes lint, tests, and build
4. Produces at least one artifact or visible output
5. Makes the repository clearly show whether a change is safe to merge

Then extend the design with a second stage or second workflow that represents delivery to an environment.

### Deliverable expectations

A strong Week 12 deliverable should include:

- A working GitHub Actions workflow in `.github/workflows/`
- At least one validation job that proves code quality or correctness
- A clear separation between validation and deployment logic
- Use of secrets or environments where deployment credentials are involved
- A short explanation in the project README of what the pipeline guarantees

### Reflection questions

Ask students to explain these in plain English:

1. What problem does your CI workflow solve for your team?
2. Which repository event starts your pipeline, and why that event?
3. Why did you separate your jobs the way you did?
4. What does your pipeline guarantee before code reaches `main`?
5. If deployment fails, what is your rollback or recovery idea?

---

## Final Takeaway

CI/CD is not primarily about writing YAML. It is about building a trustworthy path from code change to released software.

GitHub Actions gives you the mechanics:

- Events
- Workflows
- Jobs
- Steps
- Runners
- Secrets
- Environments

But the engineering value comes from the concepts behind those mechanics:

- Integrate early
- Validate automatically
- Release in small increments
- Separate concerns clearly
- Keep production changes controlled and observable

If a team understands those ideas, the exact platform syntax becomes a solvable detail.