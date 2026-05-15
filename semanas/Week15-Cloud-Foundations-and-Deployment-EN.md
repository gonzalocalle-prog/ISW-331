# Week 15 - Cloud Foundations and Deployment

> **Block 5: Cloud and Production**
> This week introduces the cloud concepts that transfer across providers: compute, storage, networking, identity, configuration, secrets, DNS, and deployment workflows. The goal is not to memorize one vendor console. The goal is to understand how a web application moves from a local repository to a public, production-facing environment.

---

## Part 1 - Why Cloud Matters for Web Applications

Cloud platforms matter because they solve a very practical problem:

> **Your application only creates value when other people can reach it reliably.**

Local development is necessary, but it is not the end state. A real product needs:

- A public URL
- A stable runtime environment
- Configuration that differs between local and production
- Storage for files and persistent data
- Logs and basic monitoring
- A deployment path the team can repeat safely

Cloud platforms package these concerns into reusable services. That does not remove engineering decisions. It changes them.

Instead of asking only, "How do I run this on my laptop?" teams start asking:

- Where should this application run?
- How will it store data?
- How will secrets be managed?
- How will users reach it?
- How will we know when it fails?

### What this week is really about

This week is not mainly about Azure, AWS, or any single provider. It is about learning the shared architecture behind them.

If students understand the common cloud model, they can move between platforms much more easily later.

---

## Part 2 - The Common Cloud Vocabulary

Different providers use different product names, but the underlying ideas are usually the same.

| Concept | What it means | Why it matters |
|---------|---------------|----------------|
| **Region** | A geographic area where services run | Affects latency, compliance, and availability |
| **Compute** | The place where your code executes | Hosts your frontend, backend, jobs, or APIs |
| **Storage** | Persistent storage for files or objects | Stores uploads, images, backups, and static assets |
| **Database** | Structured or document data persistence | Stores application state and business data |
| **Networking** | The path between users and services | Controls routing, ports, DNS, and connectivity |
| **Identity and access** | Rules about who can do what | Protects the application and its infrastructure |
| **Configuration** | Non-secret runtime settings | Defines behavior per environment |
| **Secrets** | Sensitive runtime values | Protects credentials, tokens, and keys |
| **Observability** | Logs, metrics, traces, and alerts | Helps diagnose failures and performance issues |

### Mental model

Most student applications in the cloud look like this:

```text
User browser
   -> DNS
   -> Public app endpoint
   -> Application runtime
   -> Database and storage services
   -> Logs and monitoring
```

Once students can explain that flow clearly, they already understand the core of beginner cloud architecture.

---

## Part 3 - Cloud Service Models and Good Defaults

Students often hear many labels at once: IaaS, PaaS, containers, serverless, managed services. The important question is not which label sounds more advanced. The important question is which option reduces unnecessary operational work for the current project.

| Model | What you manage | When it fits | Risk for beginners | Brief no-provider alternative |
|-------|------------------|--------------|--------------------|-------------------------------|
| **Virtual machine / IaaS** | OS, runtime, deployment, patches, networking | Useful when you need low-level control | Easy to misconfigure and harder to maintain | A Linux VM in Hyper-V, VirtualBox, or a campus virtualization host |
| **Managed app platform / PaaS** | Mostly the app code and config | Best default for first production deployments | Less flexibility, but much simpler | Dokku, Coolify, or CapRover on one server |
| **Container platform** | Image, runtime config, service wiring | Good when the project already uses Docker well | More moving parts than simple PaaS | Docker Compose, Docker Swarm, or k3s on local hardware |
| **Serverless functions** | Function code, events, config | Good for event-driven or small isolated tasks | Can confuse students if introduced too early | Background workers, cron jobs, or queue consumers on your own server |

### Recommended default for this course

For most students, the best first deployment target is one of these:

- A managed web app platform
- A managed container-based application platform
- A beginner-friendly platform-as-a-service host

That keeps the focus on deployment architecture rather than OS administration.

> **Practical rule:** In an educational setting, choose the simplest cloud option that still teaches real production concepts.

---

## Part 4 - The Minimum Production Checklist

Before deploying a project, students should be able to answer these questions:

1. What is the public URL?
2. What command builds the application?
3. What command starts the application?
4. Which environment variables are required?
5. Which values are secrets and must not live in the repository?
6. Where is the data stored?
7. How will logs be viewed?
8. How do we know the app is healthy after deployment?

### A simple production baseline

A week 15 deployment does not need advanced scaling, blue-green deployment, or multi-region failover. It should include the basics:

- A deployed application reachable through a public URL
- Correct environment variables set in the platform configuration
- Secrets stored outside the codebase
- A valid database or API connection
- Basic logs available for debugging
- HTTPS enabled if the platform supports it
- A simple health check or smoke test

This is enough to move from "it runs locally" to "it runs in a public environment with intention."

---

## Part 5 - Configuration, Secrets, and Environment Parity

One of the fastest ways to break a deployment is to treat production configuration as an afterthought.

### Configuration vs secrets

Students should learn this distinction early:

- **Configuration** changes behavior between environments
- **Secrets** must stay protected because exposure creates risk

Examples of configuration:

- `NODE_ENV=production`
- `APP_BASE_URL=https://yourapp.example.com`
- `LOG_LEVEL=info`
- `STORAGE_BUCKET=project-assets`

Examples of secrets:

- `DATABASE_URL`
- `JWT_SECRET`
- `API_KEY`
- `SMTP_PASSWORD`

### Core rule

> **Never commit secrets into the repository, screenshots, or documentation.**

Cloud platforms usually provide a secure place to define runtime variables. Students should use that feature instead of hardcoding values.

### Why environment parity matters

Production often fails because it is not actually similar to development:

- Different ports
- Different database URLs
- Missing storage configuration
- Different build command
- Missing CORS origin configuration

The closer the environment model is between local and cloud, the fewer surprises the team will face.

---

## Part 6 - DNS, Domains, and HTTPS

Deploying an app is not just about starting a process. Users need a trustworthy path to reach it.

### DNS in one sentence

DNS translates a human-readable domain name into the network destination that actually serves the application.

```text
example.com
   -> DNS record
   -> platform endpoint
   -> application runtime
```

### Why this matters in practice

You should understand the purpose of:

- A default platform URL
- An optional custom domain
- HTTPS or TLS certificates
- Redirecting traffic to the correct service

They do not need to become network specialists this week. They do need to understand that a public application depends on more than code.

---

## Part 7 - A Provider-Agnostic Deployment Workflow

The exact buttons change by platform. The workflow usually does not.

```text
Source code
   -> Build artifact or container image
   -> Cloud runtime configured
   -> Environment variables and secrets added
   -> Database and storage connected
   -> Deploy
   -> Verify logs, health, and public URL
```

### Generic deployment steps

1. Choose a deployment target that matches the current project.
2. Prepare the application for production build and startup.
3. Configure required environment variables and secrets.
4. Connect any database, storage, or third-party services.
5. Deploy the application.
6. Open logs and confirm the app actually started.
7. Verify a health endpoint or critical user flow.
8. Document the deployed URL and setup notes.

### What students should learn from this workflow

The main lesson is that deployment is a system, not a button.

Students should be able to explain:

- What was deployed
- Where it is running
- Which services it depends on
- Which runtime configuration it needs
- How they verified that it works

If you want to extend this topic into Infrastructure as Code, see the companion guide `Week15.1-Terraform-IaC-EN.md`.

---

## Part 8 - Cost, Reliability, and Observability Basics

Early cloud education should include operational realism.

### Cost awareness

Free tiers are useful, but they are not magic. Students should learn to ask:

- Is this service always on, or can it sleep?
- Does usage increase cost?
- Are backups or storage billed separately?
- What happens if logs grow without limits?

### Reliability basics

At this level, reliability means:

- The application starts consistently
- The correct configuration is present
- The database connection works
- There is at least one way to inspect failures

### Observability basics

Students do not need full distributed tracing this week. They do need:

- Application logs
- Basic request or error visibility
- A way to confirm whether a deployment succeeded or failed

> **Key principle:** If you cannot observe a deployment, you cannot trust it.

---

## Part 9 - Common Cloud Deployment Failure Modes

Most beginner deployment failures are not caused by deep cloud complexity. They come from missing basics.

### Frequent issues

- The app listens on the wrong port
- Required environment variables are missing
- Secrets were stored locally but never configured in the platform
- The build command is correct locally but wrong in production
- Database migrations were never run
- Static files or uploads rely on the local filesystem
- CORS allows localhost but not the deployed frontend URL
- The health check path does not exist
- Students verify only that the homepage loads, not that the main feature works

### Teaching takeaway

When a deployment fails:

1. Check logs
2. Check startup command
3. Check environment variables and secrets
4. Check external service connections
5. Check public URL and routing

That habit is much more valuable than memorizing one vendor interface.

---

## Part 10 - Suggested Exercise and Deliverable

### Suggested in-class exercise

Deploy a small application to any managed cloud platform. The project can be:

- A frontend application with runtime configuration
- A backend API with a health endpoint
- A full-stack project with an external database

The deployment should include:

- One public URL
- Runtime configuration through environment variables
- At least one secret stored in the platform configuration
- Logs available for troubleshooting
- A short verification checklist

### Deliverable expectations

A Week 15 deliverable should include:

- A working deployment with a public URL
- A short README section describing where the app is hosted
- A list of required environment variables
- Evidence that secrets are stored outside the repository
- A brief note explaining how the deployment was verified

### Reflection questions

1. Which parts of your deployment process would transfer to another provider with minimal change?
2. Which parts are specific to the platform you chose?
3. What failed the first time, and how did you debug it?
4. If another student had to deploy your project tomorrow, what documentation would they need?

---

## Optional Instructor Alignment - AWS Academy Paths

If you want to support this week with AWS Academy while keeping the course outcomes provider-agnostic, there are two strong starting paths.

### Path 1 - AWS Academy Cloud Foundations

Use this when students need their first structured introduction to cloud concepts.

This path fits well if the asynchronous goal is to reinforce:

- Global infrastructure concepts
- Shared responsibility
- Core service categories
- Basic security and identity concepts
- Cost awareness and cloud value propositions

### Path 2 - AWS Academy Cloud Developing

Use this when students already have a working software project and are ready to think more directly about deployment and cloud-native development.

This path fits well if the asynchronous goal is to reinforce:

- Application deployment patterns
- Runtime configuration
- Working with managed cloud services
- Development workflows for cloud applications
- Monitoring and operational thinking

### Teaching recommendation

Use AWS Academy as supporting material, not as the definition of the week's learning outcomes.

Stay provider-agnostic by assessing students on:

- Their deployment architecture
- Their use of configuration and secrets
- Their ability to explain runtime dependencies
- Their ability to verify and debug a deployment

That way, AWS Academy adds structure without narrowing the course to one provider.

---

## Recommended Reading and Exploration

- [The Twelve-Factor App](https://12factor.net/) - Clear guidance on config, environment separation, and deployable applications
- [NIST Definition of Cloud Computing](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-145.pdf) - A strong conceptual foundation for what cloud actually means
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) - Practical security guidance for handling secrets safely
- [Cloudflare Learning Center - What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/) - A beginner-friendly explanation of DNS and domain routing
