# Week 11 — Docker and Containerization

> **Block 4: DevOps and CI/CD**
> This week covers containers: the problem they solve, how Docker packages your application into portable units, and how Docker Compose orchestrates multi-container stacks. By the end, you will have your full-stack project running in containers that anyone can start with a single command.

---

## Part 1 — The Problem Docker Solves

### "It works on my machine"

Every developer has heard (or said) this. The sentence reveals a deeper problem: **your application doesn't just depend on your code — it depends on everything around it.**

Consider what a typical full-stack app needs to run:

- A specific Node.js version (or .NET, Python, Java…)
- A specific database engine and version
- Environment variables (API keys, connection strings)
- System libraries (image processing, crypto, etc.)
- A particular OS configuration

When you hand your project to a teammate, a CI server, or a cloud provider, **any** of these can differ. The result: mysterious errors that have nothing to do with your code.

### The Shipping Container Analogy

Before standardized shipping containers, moving goods internationally was chaos. Every port had different equipment, every ship had a different hold shape, and cargo had to be loaded and unloaded by hand. A shipment of coffee from Colombia to Japan might be handled 20 times.

Then someone invented the **intermodal container**: a standard metal box. It didn't matter what was inside — coffee, electronics, furniture. Every crane, truck, train, and ship could handle it the same way.

Docker is the intermodal container for software:

```
Traditional Deployment                 Docker Deployment
─────────────────────                 ─────────────────
                                      
"Install Node 18..."                 ┌────────────────────┐
"Set up PostgreSQL 15..."            │    Docker Image     │
"Configure these 12 env vars..."     │                     │
"Install libvips for images..."      │  Your code          │
"Use Ubuntu 22.04..."                │  + Node 18          │
"Hope nothing conflicts..."          │  + dependencies     │
                                     │  + OS libraries     │
 Result: :(                          │  + configuration    │
                                     └────────────────────┘
                                      
                                      docker run → works everywhere
                                      Result: :)
```

> **Key insight:** Docker doesn't eliminate complexity — it **packages** it. All the dependencies, configuration, and setup are captured once in a file (the Dockerfile), and then reproduced identically everywhere.

---

## Part 2 — Containers vs Virtual Machines

You might think: "This sounds like a virtual machine." It's similar in spirit but fundamentally different in architecture:

```
Virtual Machines                        Containers
─────────────────                       ──────────

┌──────────────────┐                    ┌──────────────────┐
│     Your App     │                    │     Your App     │
├──────────────────┤                    ├──────────────────┤
│   Guest OS       │                    │  Container Layer │
│  (full Ubuntu)   │                    │  (just libs/bins)│
├──────────────────┤                    └──────┬───────────┘
│   Hypervisor     │                           │ shares
├──────────────────┤                    ┌──────▼───────────┐
│    Host OS       │                    │    Host OS       │
├──────────────────┤                    │    Kernel        │
│   Hardware       │                    ├──────────────────┤
└──────────────────┘                    │   Hardware       │
                                        └──────────────────┘
```

| Aspect | Virtual Machine | Container |
|--------|----------------|-----------|
| **Includes** | Full guest operating system | Only app + its dependencies |
| **Size** | Gigabytes | Megabytes |
| **Startup time** | Minutes | Seconds |
| **Isolation** | Complete (separate kernel) | Process-level (shared kernel) |
| **Use case** | Running different OSes | Running different apps |

> **Mental model:** A VM is like renting an entire apartment. A container is like renting a desk in a coworking space — you get your own workspace but share the building's infrastructure.

---

## Part 3 — Core Docker Concepts

Before touching any commands, understand these three building blocks:

### Images

An **image** is a read-only template that contains everything needed to run your application. Think of it as a **snapshot** — frozen at a point in time.

- Images are **built** from a Dockerfile
- Images are **layered** (each instruction creates a layer)
- Images are **shared** via registries (Docker Hub, GitHub Container Registry)
- Images **don't run** — they are blueprints

### Containers

A **container** is a **running instance** of an image. It adds a thin writable layer on top of the read-only image.

- You can run **multiple containers** from the same image
- Each container is **isolated** from others
- Containers are **ephimeral** — when they stop, their writable layer is gone (unless you use volumes)

### Volumes

A **volume** is persistent storage that survives container restarts and deletions. This is how you keep your database data alive.

```
┌─────────────────────────────────────────┐
│              Docker Host                 │
│                                          │
│  ┌──────────┐    ┌──────────┐           │
│  │ Container│    │ Container│           │
│  │  (API)   │    │  (DB)    │           │
│  └────┬─────┘    └────┬─────┘           │
│       │               │                 │
│       │          ┌────▼─────┐           │
│       │          │  Volume  │           │
│       │          │ (pgdata) │           │
│       │          └──────────┘           │
│       │               ▲                 │
│       │               │ persists        │
│       │               │ across restarts │
└───────┴───────────────┴─────────────────┘
```

### Quick Reference

| Concept | What it is | Analogy |
|---------|-----------|---------|
| **Image** | Read-only blueprint | A class in OOP |
| **Container** | Running instance | An object (instance of a class) |
| **Volume** | Persistent storage | An external hard drive |
| **Registry** | Image storage (Docker Hub) | npm / NuGet for containers |
| **Dockerfile** | Build instructions | A recipe |

---

## Part 4 — The Dockerfile: Your Build Recipe

A Dockerfile is a text file with instructions for building an image. Each instruction creates a **layer**.

### Anatomy of a Dockerfile (Node.js example)

```dockerfile
# 1. Start from a base image
#    "node:20-alpine" = Node.js 20 on Alpine Linux (tiny, ~50MB)
FROM node:20-alpine

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy dependency files first (for layer caching)
COPY package.json package-lock.json ./

# 4. Install dependencies
RUN npm ci

# 5. Copy the rest of your source code
COPY . .

# 6. Expose the port your app listens on
EXPOSE 3000

# 7. Define the command to start the app
CMD ["node", "src/index.js"]
```

### Why the Order Matters: Layer Caching

Docker caches each layer. If a layer hasn't changed, Docker reuses it instead of rebuilding. This is why we copy `package.json` **before** the source code:

```
Instruction              Layer    Cached?
─────────────────────    ─────    ───────
FROM node:20-alpine      Layer 1  ✅ (rarely changes)
WORKDIR /app             Layer 2  ✅ (never changes)
COPY package*.json ./    Layer 3  ✅ (changes only when deps change)
RUN npm ci               Layer 4  ✅ (reuses if package.json unchanged)
COPY . .                 Layer 5  ❌ (changes when code changes)
CMD [...]                Layer 6  ❌ (rebuilt because layer 5 changed)
```

> **Key insight:** If you `COPY . .` everything first, then `npm ci` runs every single time you change any file. By copying package files first, installs are cached until you actually add or remove a dependency. This can turn a 2-minute build into a 5-second build.

### Common Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:20-alpine` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files from host to image | `COPY . .` |
| `RUN` | Execute a command during build | `RUN npm ci` |
| `EXPOSE` | Document which port the app uses | `EXPOSE 3000` |
| `CMD` | Default command when container starts | `CMD ["node", "index.js"]` |
| `ENV` | Set environment variable | `ENV NODE_ENV=production` |
| `ARG` | Build-time variable | `ARG API_VERSION=v1` |

### .dockerignore

Just like `.gitignore` keeps files out of your repo, `.dockerignore` keeps files out of your image:

```
node_modules
.git
.env
*.md
.vscode
coverage
dist
```

> **Why this matters:** Without `.dockerignore`, `COPY . .` sends your entire `node_modules` folder (often 200MB+) to the Docker daemon, only to be replaced by `npm ci`. The build context becomes enormous and slow.

---

## Part 5 — Building and Running Containers

### Building an Image

```bash
# Build an image and tag it with a name
docker build -t my-api .

# The "." is the build context — Docker sends this directory to the daemon
```

What happens during `docker build`:

```
1. Docker reads the Dockerfile
2. For each instruction, it creates a layer
3. Layers are cached — unchanged layers are reused
4. The final result is an image stored locally
```

### Running a Container

```bash
# Basic run
docker run my-api

# Run with port mapping (host:container)
docker run -p 3000:3000 my-api

# Run in detached mode (background)
docker run -d -p 3000:3000 my-api

# Run with environment variables
docker run -p 3000:3000 -e DATABASE_URL=postgres://... my-api

# Run with a name (easier to reference later)
docker run -d -p 3000:3000 --name api my-api
```

### Port Mapping Explained

The container has its own network. Port mapping connects a host port to a container port:

```
Your machine (host)              Container
───────────────────             ──────────
                                
localhost:3000  ──────────────▶  :3000 (your app)
localhost:5432  ──────────────▶  :5432 (postgres)

-p 3000:3000  means  hostPort:containerPort
-p 8080:3000  means  access via localhost:8080
                     → forwards to container's port 3000
```

### Essential Container Commands

| Command | Purpose |
|---------|---------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop <name>` | Gracefully stop a container |
| `docker rm <name>` | Remove a stopped container |
| `docker logs <name>` | View container output |
| `docker exec -it <name> sh` | Open a shell inside a running container |
| `docker images` | List local images |
| `docker rmi <image>` | Remove an image |

---

## Part 6 — Docker Compose: Orchestrating Multiple Containers

A real application is rarely a single container. A typical full-stack app needs:

- A **frontend** container (React, Angular, Vue…)
- A **backend** container (Express, ASP.NET, Django…)
- A **database** container (PostgreSQL, MySQL, MongoDB…)
- Maybe a **cache** container (Redis)

Managing each one individually with `docker run` commands would be painful. **Docker Compose** lets you define all containers in one file and start everything with a single command.

### docker-compose.yml — The Orchestration File

```yaml
# docker-compose.yml
services:
  # --- Backend API ---
  api:
    build: ./api                    # Build from ./api/Dockerfile
    ports:
      - "3000:3000"                 # Expose on localhost:3000
    environment:
      DATABASE_URL: postgres://postgres:secret@db:5432/myapp
      JWT_SECRET: dev-secret-change-in-prod
    depends_on:
      - db                          # Start db before api
    volumes:
      - ./api/src:/app/src          # Hot reload: sync source code

  # --- Database ---
  db:
    image: postgres:16-alpine       # Use official image (no Dockerfile needed)
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data   # Persist data

  # --- Frontend ---
  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    volumes:
      - ./frontend/src:/app/src     # Hot reload

# Named volumes (managed by Docker)
volumes:
  pgdata:
```

### What Compose Gives You

1. **One command to rule them all:** `docker compose up` starts everything
2. **Networking for free:** Containers can reach each other by service name (`db`, `api`, `frontend`)
3. **Dependency ordering:** `depends_on` ensures the database starts before the API
4. **Volume management:** Named volumes persist data; bind mounts enable hot reload

### How Service Networking Works

```
┌──────────────────────────────────────────────┐
│          Docker Compose Network              │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ frontend │  │   api    │  │    db    │  │
│  │          │──▶│          │──▶│          │  │
│  │ :5173    │  │ :3000    │  │ :5432    │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                              │
│  frontend calls: http://api:3000/api/users  │
│  api connects:   postgres://db:5432/myapp   │
└──────────────────────────────────────────────┘
```

> **Key insight:** Inside the Compose network, containers use **service names** (not `localhost`) to talk to each other. The `api` service connects to the database at `db:5432`, not `localhost:5432`.

### Essential Compose Commands

| Command | Purpose |
|---------|---------|
| `docker compose up` | Start all services (foreground) |
| `docker compose up -d` | Start all services (background) |
| `docker compose up --build` | Rebuild images before starting |
| `docker compose down` | Stop and remove all containers |
| `docker compose down -v` | Stop, remove containers **and volumes** (deletes data!) |
| `docker compose logs api` | View logs for a specific service |
| `docker compose exec api sh` | Shell into a running service |
| `docker compose ps` | List running services |

---

## Part 7 — Multi-Stage Builds: Smaller, Safer Images

A development image has everything: compilers, dev dependencies, source maps, test frameworks. A production image should have **only what's needed to run**.

Multi-stage builds let you use multiple `FROM` instructions. Each stage can copy artifacts from previous stages:

### Node.js Multi-Stage Example

```dockerfile
# ── Stage 1: Build ──
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build          # Produces /app/dist

# ── Stage 2: Production ──
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev      # Only production dependencies
COPY --from=builder /app/dist ./dist

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### What This Achieves

```
Stage 1 (builder)              Stage 2 (production)
─────────────────              ────────────────────
✅ All devDependencies          ❌ No devDependencies
✅ TypeScript compiler          ❌ No TypeScript
✅ Source code                   ❌ No source code
✅ Test frameworks              ❌ No test tools

Image size: ~400MB              Image size: ~80MB
Attack surface: large           Attack surface: minimal
```

> **Why this matters for security:** Every package in your image is a potential vulnerability. Multi-stage builds ensure your production image only contains what it needs to run.

### .NET Multi-Stage Example

```dockerfile
# ── Stage 1: Build ──
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# ── Stage 2: Runtime ──
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

---

## Part 8 — Development vs Production Workflows

Docker behaves differently depending on whether you're developing or deploying.

### Development: Fast Feedback Loop

During development, you want:
- **Hot reload**: see code changes instantly without rebuilding
- **Source maps and debugging**: full error traces
- **All dev tools**: linters, test runners, etc.

This is achieved with **bind mounts** that sync your host files into the container:

```yaml
# docker-compose.yml (dev)
services:
  api:
    build: ./api
    ports:
      - "3000:3000"
    volumes:
      - ./api/src:/app/src     # Your code is synced live
    environment:
      NODE_ENV: development
    command: npm run dev        # nodemon / tsx watch
```

When you edit `./api/src/index.js` on your machine, the change is **immediately** visible inside the container. Nodemon (or tsx) detects the change and restarts.

### Production: Optimized and Locked Down

In production, you want:
- **No source code in the image** (use compiled output)
- **No devDependencies**
- **No bind mounts** (the image is self-contained)
- **Non-root user**

```yaml
# docker-compose.prod.yml
services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile     # uses multi-stage build
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
    # No volumes — image contains everything
```

### Compose File Override Pattern

A common pattern is to have a base file and override files:

```
docker-compose.yml          # Base configuration (shared)
docker-compose.override.yml # Dev overrides (auto-loaded)
docker-compose.prod.yml     # Production overrides
```

```bash
# Development (auto-loads override)
docker compose up

# Production (explicit file)
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## Part 9 — Common Gotchas and Troubleshooting

### Gotcha 1: "My database is empty every time I restart"

**Cause:** No volume defined. Container storage is ephemeral.

**Fix:** Add a named volume:

```yaml
services:
  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data   # ← this persists

volumes:
  pgdata:    # ← declare the named volume
```

### Gotcha 2: "The API can't connect to the database"

**Cause:** Using `localhost` in the connection string. Inside Docker, each container has its own `localhost`.

**Fix:** Use the **service name** from docker-compose.yml:

```
❌  DATABASE_URL=postgres://postgres:secret@localhost:5432/myapp
✅  DATABASE_URL=postgres://postgres:secret@db:5432/myapp
                                            ^^
                                         service name
```

### Gotcha 3: "My changes aren't showing up"

**Cause:** You changed code but the container is running from a cached image.

**Fix:** Rebuild the image:

```bash
docker compose up --build
```

### Gotcha 4: "npm install added a new package but the container doesn't have it"

**Cause:** `node_modules` inside the container was created at build time. Your bind mount doesn't include `node_modules`.

**Fix:** Rebuild the image after changing dependencies:

```bash
docker compose up --build
```

Or use an anonymous volume to preserve container's `node_modules`:

```yaml
volumes:
  - ./api/src:/app/src
  - /app/node_modules    # ← anonymous volume, preserves container's modules
```

### Gotcha 5: "Port already in use"

**Cause:** Something on your host is already using that port (another container, or a local process).

**Fix:** Either stop the conflicting process, or change the host port:

```yaml
ports:
  - "3001:3000"    # Use 3001 on host, 3000 inside container
```

---

## Part 10 — Docker Security Basics

A few rules that matter from day one:

### 1. Never Put Secrets in Images

```dockerfile
# ❌ NEVER do this
ENV DATABASE_PASSWORD=my-secret-password
COPY .env .

# ✅ Pass secrets at runtime
# docker run -e DATABASE_PASSWORD=secret my-api
# Or use docker compose environment/env_file
```

> Anything in a Dockerfile or image layer is visible to anyone with access to the image. Secrets go in environment variables, injected at runtime.

### 2. Use Non-Root Users

By default, containers run as root. If an attacker breaks into your container, they have root access:

```dockerfile
# Add to your Dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

### 3. Use Specific Image Tags

```dockerfile
# ❌ "latest" can change unexpectedly
FROM node:latest

# ✅ Pin to a specific version
FROM node:20-alpine
```

### 4. Keep Images Small

Smaller images = fewer packages = smaller attack surface.

| Base Image | Size | Use Case |
|-----------|------|----------|
| `node:20` | ~350MB | Full Debian, everything included |
| `node:20-slim` | ~200MB | Minimal Debian |
| `node:20-alpine` | ~50MB | Minimal Alpine Linux |

---

## Part 11 — Guided Exercise: Dockerize a Full-Stack App

This is the exercise to follow during the synchronous session.

### Goal

Take a project with a frontend (React/Vite), a backend (Express or ASP.NET), and a database (PostgreSQL), and make it runnable with a single command: `docker compose up`.

### Step-by-Step

**Step 1: Create the API Dockerfile** (`./api/Dockerfile`)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

**Step 2: Create the Frontend Dockerfile** (`./frontend/Dockerfile`)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 5173
CMD ["npm", "run", "dev", "--", "--host"]
```

> The `--host` flag tells Vite to listen on `0.0.0.0` instead of `localhost`, which is necessary inside a container.

**Step 3: Create docker-compose.yml**

```yaml
services:
  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:secret@db:5432/myapp
    depends_on:
      - db
    volumes:
      - ./api/src:/app/src

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    volumes:
      - ./frontend/src:/app/src

  db:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

**Step 4: Add .dockerignore** (in both `./api` and `./frontend`)

```
node_modules
.git
.env
*.md
.vscode
```

**Step 5: Run it**

```bash
docker compose up --build
```

**Step 6: Verify**

- Frontend: `http://localhost:5173`
- API: `http://localhost:3000/api/health`
- Database: connect via any client to `localhost:5432`

### Deliverable

A `docker-compose.yml` at the root of your project that starts the entire application. A teammate should be able to clone your repo and run `docker compose up` with no other setup.

---

## Quick Reference — CLI Cheat Sheet

### Images

```bash
docker build -t name:tag .        # Build image from Dockerfile
docker images                     # List local images
docker rmi image_name             # Remove an image
docker pull image_name            # Download from registry
docker push image_name            # Upload to registry
```

### Containers

```bash
docker run -d -p 3000:3000 image  # Run in background with port mapping
docker ps                         # List running containers
docker ps -a                      # List all containers
docker stop container_name        # Stop a container
docker rm container_name          # Remove a container
docker logs container_name        # View logs
docker exec -it container sh      # Shell into container
```

### Docker Compose

```bash
docker compose up                 # Start all services
docker compose up -d              # Start in background
docker compose up --build         # Rebuild and start
docker compose down               # Stop and remove
docker compose down -v            # Stop, remove, and delete volumes
docker compose logs service       # View service logs
docker compose exec service sh    # Shell into service
docker compose ps                 # List services
```

### Cleanup

```bash
docker system prune               # Remove unused data
docker system prune -a            # Remove ALL unused data (including images)
docker volume prune               # Remove unused volumes
```

---

## Further Reading

- [Docker Documentation — Get Started](https://docs.docker.com/get-started/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Node.js Docker Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
