# Week 3 — Professional Setup: Branching Strategies & Code Quality Tooling

## Overview

This week is all about the **invisible infrastructure** that separates hobby projects from professional codebases. We cover two foundational topics: **branching strategies** (how your team collaborates on code without stepping on each other) and **linters & formatters** (how you enforce code quality automatically, before bugs ever reach production). Both topics follow the same structure: **What → Why → How**.

---

## Part 1 — Branching Strategies (How Teams Collaborate on Code)

### The Bigger Picture: Version Control & Team Collaboration

Before diving into specific branching models, let's understand the **conceptual foundation** they're built on. Branching strategies aren't just "how to use Git" — they're part of a broader discipline of how teams coordinate work on shared codebases.

#### The Discipline: Configuration Management

**Configuration management** (CM) is the practice of tracking and controlling changes to software artifacts. Branching strategies are one component of CM, specifically focused on **managing parallel development**.

| Aspect | What It Covers | Tools/Practices |
|--------|---------------|-----------------|
| **Source Control** | Tracking changes to code over time | Git, Mercurial, Subversion |
| **Branching & Merging** | Managing parallel development streams | Git Flow, GitHub Flow, Trunk-Based Development |
| **Release Management** | Coordinating what goes into each release | Semantic versioning, changelog automation, release branches |
| **Build Management** | Ensuring reproducible builds | Docker, dependency lock files, build scripts |
| **Deployment Configuration** | Managing environment-specific settings | Environment variables, feature flags, config as code |

> **Key insight:** A branching strategy is just one piece of configuration management. It answers: "How do we coordinate changes from multiple people working at the same time?"

#### The Evolution of Source Control Paradigms

Version control systems evolved over decades, and each generation influenced how teams think about branching:

```
1980s-1990s          2000s               2005+               2010s+
──────────────────────────────────────────────────────────────────
Centralized         Centralized         Distributed         Cloud-Native
Lock-based          Merge-based         Merge-based         Trunk-based

RCS, CVS      →     Subversion    →     Git, Mercurial →    + CI/CD
                    Perforce                                + Feature Flags
                                                            + Cloud Platforms

Problem:            Problem:            Problem:            Solution:
- One person at     - Long-lived        - Integration       - Merge to trunk
  a time              branches            hell                constantly
- No parallelism    - Merge conflicts   - Complex history   - Deploy often
                    - Slow integration                      - Hide features
                                                             behind flags
```

**Further exploration:**
- [A Visual Guide to Version Control](https://betterexplained.com/articles/a-visual-guide-to-version-control/) — History and evolution of VCS
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) — Understanding Git's data model

#### Related Concepts & Topics

Branching strategies intersect with several broader software engineering disciplines:

##### 1. **Team Coordination & Conway's Law**

> "Any organization that designs a system will produce a design whose structure is a copy of the organization's communication structure." — Melvin Conway, 1967

Your branching strategy should **match your team structure**:

| Team Structure | Branching Strategy That Fits | Why |
|----------------|------------------------------|-----|
| **Solo developer** | Simple main branch, optional feature branches | No coordination overhead needed |
| **Small co-located team (2-5)** | GitHub Flow or Trunk-Based with short branches | High trust, fast communication, easy to coordinate |
| **Distributed team (5-20)** | GitHub Flow with PR reviews | Asynchronous collaboration, documented decisions |
| **Multiple teams on one codebase** | Trunk-Based with feature flags or release branches | Avoid blocking each other, independent release cycles |
| **Multiple teams, separate services** | Each team picks their own (likely Trunk-Based) | Teams own their deployment pipeline |

**Further exploration:**
- [Conway's Law](https://martinfowler.com/bliki/ConwaysLaw.html) — Martin Fowler's explanation
- [The Inverse Conway Maneuver](https://www.thoughtworks.com/radar/techniques/inverse-conway-maneuver) — Using team structure to shape architecture

##### 2. **Continuous Integration (CI)**

Continuous Integration is the practice of **integrating code changes frequently** (multiple times per day) to detect conflicts and bugs early.

**The relationship with branching:**

| Branching Approach | Integration Frequency | True CI? |
|--------------------|----------------------|---------|
| **Long-lived feature branches** (weeks) | Weekly or monthly | ❌ No — integration is batched, not continuous |
| **Short-lived feature branches** (days) | Daily | ⚠️ Partially — better than long branches, but still delayed |
| **Trunk-based with daily merges** | Multiple times per day | ✅ Yes — this is true continuous integration |

> **Misconception:** Many teams say "we do CI" because they use GitHub Actions or Jenkins. But CI is about **how often you merge**, not just running automated builds.

**The CI Test (from Jez Humble):**
- Can every developer merge to `main` at least once per day?
- Does every merge trigger automated tests?
- Can you fix a broken build in < 10 minutes?

If you answered "no" to any of these, you're doing **automated builds**, not Continuous Integration.

**Further exploration:**
- [Martin Fowler: Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) — The canonical reference
- [Continuous Delivery: Reliable Software Releases](https://continuousdelivery.com/) — Book by Jez Humble and David Farley

##### 3. **Release Management & Deployment Frequency**

Your branching strategy directly affects **how often you can release** and **how risky releases feel**.

| Metric | Long-Lived Branches | Short-Lived Branches | Trunk-Based |
|--------|---------------------|----------------------|-------------|
| **Deploy frequency** | Monthly or less | Weekly | Multiple times per day |
| **Change size** | Large (many features) | Medium | Small (incremental) |
| **Risk per deploy** | High | Medium | Low |
| **Rollback complexity** | Hard (many changes) | Medium | Easy (small changes) |

**DORA's Four Key Metrics** (from the book *Accelerate*):
1. **Deployment Frequency** — How often you deploy to production
2. **Lead Time for Changes** — Time from commit to production
3. **Mean Time to Restore** (MTTR) — How fast you recover from failure
4. **Change Failure Rate** — % of deployments that cause issues

**Elite performers** (top 10% of teams) achieve:
- Deploy frequency: **Multiple times per day**
- Lead time: **Less than one hour**
- MTTR: **Less than one hour**
- Change failure rate: **0-15%**

**Their secret?** Trunk-based development + comprehensive test automation + feature flags.

**Further exploration:**
- [DORA State of DevOps Report](https://dora.dev/) — Annual research on high-performing teams
- [Accelerate (book)](https://itrevolution.com/product/accelerate/) — The science behind DevOps performance
- [Feature Toggles (Feature Flags)](https://martinfowler.com/articles/feature-toggles.html) — Martin Fowler's comprehensive guide

##### 4. **Code Review & Pull Requests**

Code review is a quality gate **before** code merges. Your branching strategy determines when and how code review happens.

| Approach | Code Review Model | Trade-off |
|----------|-------------------|----------|
| **GitHub Flow** | Every change goes through a PR. Review is mandatory. | Slower to merge, but higher quality and knowledge sharing |
| **Trunk-Based (with PRs)** | Short-lived branches (< 1 day), fast reviews. | Quick feedback, but requires discipline for small changes |
| **Trunk-Based (pair/mob programming)** | No PRs — continuous real-time review. | Fastest integration, but requires co-located or synchronous work |

**Best practices (from Google's Engineering Practices):**
- Small PRs (< 200 lines) get reviewed faster
- Reviewers should respond within 24 hours
- Automate what can be automated (formatting, linting) — reviewers focus on logic

**Further exploration:**
- [Google: Code Review Developer Guide](https://google.github.io/eng-practices/review/) — How Google does code review
- [The Gentle Art of Patch Review](https://sage.thesharps.us/2014/09/01/the-gentle-art-of-patch-review/) — Philosophy of constructive review

##### 5. **Merge Strategies & Conflict Resolution**

How you merge branches affects code history, traceability, and ease of rollback.

| Merge Strategy | What It Does | When to Use |
|----------------|-------------|-------------|
| **Merge Commit** | Creates a merge commit with two parents. Preserves full history. | Default for GitHub Flow. Useful when you want to see feature branches in history. |
| **Squash & Merge** | Collapses all commits in a branch into one. Clean linear history. | When branch commits are messy ("fix typo", "wip", "oops"). |
| **Rebase & Merge** | Replays commits on top of `main`. Linear history, no merge commit. | When you want a clean history and can rewrite branch history. |
| **Fast-Forward** | Moves branch pointer forward. No merge commit. Only possible if no divergence. | Trunk-Based Development with very short branches. |

**GitHub's default:** Merge commit (preserves all history).  
**Common in open-source:** Squash & merge (keeps `main` clean).  
**Advanced teams:** Rebase & merge (linear history, easier to bisect bugs).

**Further exploration:**
- [Atlassian: Merging vs. Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing) — Visual explanation of merge strategies
- [Git: Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) — When and how to rewrite commits

##### 6. **Semantic Versioning & Changelog Automation**

Versioning is how you communicate **what changed** to users and other teams.

**Semantic Versioning (SemVer):** `MAJOR.MINOR.PATCH` (e.g., `2.4.1`)
- **MAJOR** — Breaking changes (API incompatible)
- **MINOR** — New features (backward compatible)
- **PATCH** — Bug fixes (backward compatible)

**How branching strategies support versioning:**

| Strategy | Versioning Approach |
|----------|-------------------|
| **Git Flow** | `release/*` branches map to MINOR versions. Hotfixes bump PATCH. |
| **GitHub Flow** | Every merge to `main` can be a PATCH or MINOR bump (you decide). |
| **Trunk-Based** | Continuous deployment → frequent PATCH bumps. MINOR/MAJOR are feature-flag reveals. |

**Changelog automation tools:**
- [Conventional Commits](https://www.conventionalcommits.org/) — Structured commit messages (`feat:`, `fix:`, `BREAKING CHANGE:`)
- [semantic-release](https://github.com/semantic-release/semantic-release) — Automates version bumps and changelog generation
- [Release Please](https://github.com/googleapis/release-please) — Google's automated release tool (used in GitHub Actions)

**Further exploration:**
- [Semantic Versioning](https://semver.org/) — The spec
- [Keep a Changelog](https://keepachangelog.com/) — How to write good changelogs
- [Conventional Commits](https://www.conventionalcommits.org/) — Commit message format that enables automation

---

### The Problem (Specific to Branching Strategies)

Now that we understand the broader context, let's zoom into the specific problem branching strategies solve.

When multiple developers work on the same codebase, chaos emerges fast:

- Two people edit the same file → **merge conflicts**.
- Someone pushes broken code to `main` → **everyone is blocked**.
- A feature takes 3 weeks → the branch drifts so far from `main` that merging becomes a nightmare (**integration hell**).
- No one knows what's in production vs. what's still in progress.

Branching strategies are **conventions** — agreed-upon rules for when to create branches, how to name them, and when/how to merge them back. There is no "best" strategy; the right choice depends on your team size, release cadence, and project complexity.

---

### 1. Git Flow

#### What

Git Flow, introduced by Vincent Driessen in 2010, is a **structured branching model** with clearly defined branch types:

| Branch | Purpose | Lifetime |
|--------|---------|----------|
| `main` | Production-ready code. Every commit here is a release. | Permanent |
| `develop` | Integration branch. All features merge here first. | Permanent |
| `feature/*` | New features. Branch off `develop`, merge back into `develop`. | Temporary |
| `release/*` | Prepare a release. Branch off `develop`, merge into both `main` and `develop`. | Temporary |
| `hotfix/*` | Emergency production fixes. Branch off `main`, merge into both `main` and `develop`. | Temporary |

```
main:       ●───────────────────●──────────●
             \                 / \        /
release:      \          ●───●   \      /
               \         /        \    /
develop:  ●─────●───●───●──────────●──●
               \   /
feature:        ●─●
```

#### Why

- **Clear separation** between what's in production, what's being prepared for release, and what's still in development.
- **Parallel releases** — you can stabilize a release branch while `develop` keeps moving forward.
- **Hotfix isolation** — emergency fixes don't require cherry-picking across branches.
- Works well for software with **scheduled release cycles** (e.g., "we ship every two weeks").

#### When NOT to use

- If you deploy continuously (multiple times per day), Git Flow adds unnecessary ceremony.
- Small teams (1-3 developers) will find the overhead painful.
- The long-lived `develop` branch can become a merge-conflict magnet.

#### Learn more

- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/) — Vincent Driessen's original blog post
- [Atlassian: Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) — Visual walkthrough with commands

---

### 2. GitHub Flow

#### What

GitHub Flow is a **simplified branching model** created by GitHub. It has exactly **one rule**: anything in `main` is deployable. The entire workflow is:

1. Create a branch off `main` (name it descriptively: `add-user-auth`, `fix-cart-total`).
2. Make commits. Push the branch.
3. Open a **Pull Request** (PR).
4. Team reviews the code, discusses, requests changes.
5. Once approved, **merge to `main`**.
6. **Deploy** immediately (or automatically via CI/CD).

```
main:    ●────●────●────●────●────●
          \       /      \       /
branch:    ●──●──●        ●──●──●
           add-auth       fix-cart
```

That's it. No `develop`, no `release/*`, no `hotfix/*`. Just `main` and short-lived feature branches.

#### Why

- **Dead simple** — one permanent branch, one workflow. New team members understand it in minutes.
- **Encourages small, frequent merges** — branches live hours or days, not weeks. Less integration hell.
- **Built around Pull Requests** — code review is a first-class citizen, not an afterthought.
- **Pairs perfectly with CI/CD** — every merge to `main` triggers automated tests and deployment.
- Used internally by **GitHub, Shopify, and thousands of startups**.

#### When NOT to use

- If you need to maintain **multiple versions** in production simultaneously (e.g., v2.1 and v3.0).
- If your deployment process is manual and slow — GitHub Flow assumes you can ship `main` at any time.

#### Learn more

- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow) — Official GitHub documentation
- [Understanding the GitHub Flow](https://guides.github.com/introduction/flow/) — Visual guide by GitHub
- [Scott Chacon: GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html) — The original blog post explaining the motivation

---

### 3. Trunk-Based Development (TBD)

#### What

Trunk-Based Development takes simplicity even further: **everyone commits directly to a single branch** (the "trunk" — usually `main`). If branches exist at all, they are extremely short-lived (less than a day).

Two variants exist:

| Variant | How it works |
|---------|-------------|
| **True TBD** | Developers commit directly to `main`. No branches at all. | 
| **Scaled TBD** | Short-lived feature branches (< 1 day), but no long-lived branches. PRs are optional and merged quickly. |

```
main:  ●──●──●──●──●──●──●──●──●──●
        ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑
       dev1  dev2  dev1  dev3  dev2
```

To make this work safely, teams rely on:

- **Feature flags** — New features are merged into `main` but hidden behind a toggle (`if (featureFlags.newCheckout) { ... }`). You deploy code that isn't "on" yet.
- **CI that runs on every commit** — If you break `main`, you know within minutes.
- **Small, incremental changes** — No 2,000-line PRs. Every commit should be shippable.

#### Why

- **Fastest integration** — No merge conflicts because branches don't diverge far.
- **Continuous Integration for real** — "integrate continuously" literally means merging to trunk multiple times per day.
- **Used by elite teams** — Google, Meta, Microsoft, Netflix, and most companies scoring "Elite" on the [DORA metrics](https://dora.dev/) use trunk-based development.
- The [Accelerate](https://itrevolution.com/product/accelerate/) research (by Dr. Nicole Forsgren, Jez Humble, Gene Kim) found that trunk-based development is a **statistically significant predictor** of high software delivery performance.

#### When NOT to use

- Teams without strong CI/CD — if your test suite takes hours or doesn't exist, merging to trunk constantly is dangerous.
- Teams without a feature flag system — you'd ship half-finished features to users.
- Very junior teams — requires discipline to make small, safe commits.

#### Learn more

- [trunkbaseddevelopment.com](https://trunkbaseddevelopment.com/) — The definitive reference site with diagrams, FAQs, and detailed guides
- [Google's Engineering Practices: Trunk-Based Development](https://cloud.google.com/architecture/devops/devops-tech-trunk-based-development) — How Google approaches it
- [Atlassian: Trunk-Based Development](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development) — Practical guide with comparisons

---

### Comparison: Which Strategy When?

| Factor | Git Flow | GitHub Flow | Trunk-Based |
|--------|----------|-------------|-------------|
| **Complexity** | High (5 branch types) | Low (2 branch types) | Minimal (1 branch) |
| **Best for** | Scheduled releases, multiple versions | Continuous delivery, small-medium teams | High-performing teams, continuous deployment |
| **Branch lifetime** | Days to weeks | Hours to days | Minutes to hours (or none) |
| **Release cadence** | Weekly / monthly | On every merge | Multiple times per day |
| **Code review** | Optional | Via PRs (required) | Optional (pair programming common) |
| **CI/CD dependency** | Low | Medium | Critical |
| **Feature flags needed?** | No | No | Yes |

> **Recommendation for this course:** Use **GitHub Flow**. It strikes the best balance between simplicity and professionalism. You get pull requests, code review, and CI/CD integration without the overhead of Git Flow or the discipline demands of Trunk-Based Development.

---

