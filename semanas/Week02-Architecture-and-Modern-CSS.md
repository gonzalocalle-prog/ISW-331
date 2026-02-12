# Week 2 — Software Architecture & Modern CSS

## Overview

This week we clear up a very common misconception: **MVC, Clean Architecture, and Microservices are NOT competing choices**. They operate at different levels of abstraction and are often used *together* in the same project. We also take a deeper look at modern CSS — specifically *why* Tailwind CSS became the dominant approach and the engineering principles that make it work.

---

## Part 1 — Software Architecture (Getting the Concepts Right)

### The Misconception

Many tutorials frame the decision as _"MVC **vs** Clean Architecture **vs** Microservices"_, as if you pick one and discard the others. In reality:

| Concept | What it is | Level |
|---------|-----------|-------|
| **MVC** (Model-View-Controller) | A **design pattern** that separates input handling, business logic, and presentation | Code-level (how you organize files and classes inside a single app) |
| **Clean Architecture** | An **architecture philosophy** that enforces dependency rules between layers (entities → use cases → adapters → frameworks) | Application-level (how the layers of a single application relate to each other) |
| **Microservices** | A **deployment / system topology** where a system is split into independently deployable services | System-level (how multiple applications talk to each other) |

They work **in conjunction**:

> A system built with **microservices** can have each service internally organized using **Clean Architecture**, and within the adapter layer of each service, the web framework may follow the **MVC** pattern.

Think of it as nesting dolls: **System topology → Application architecture → Code patterns**.

---

### 1. Design Patterns (Code-Level Organization)

#### MVC — Model-View-Controller

The oldest and most widespread UI pattern. The **Model** holds data and business rules, the **View** renders the UI, and the **Controller** handles input and coordinates between the two.

**When to use:** Traditional server-rendered web apps (Rails, Laravel, ASP.NET MVC, Django), simple CRUD applications.

**Alternatives:**

| Pattern | Key Difference | Common In |
|---------|---------------|-----------|
| **MVVM** (Model-View-ViewModel) | The ViewModel exposes data via bindings; the View auto-updates. No direct Controller. | WPF, SwiftUI, Angular, Vue.js |
| **MVP*** (Model-View-Presenter) | The Presenter talks to the View through an interface; easier to unit test than classic MVC. | Android (legacy), desktop apps |
| **Flux / Redux** (Unidirectional data flow) | Single source of truth (store), actions, reducers. One-way data flow instead of two-way bindings. | React ecosystem |

**Example repos:**

- [TodoMVC](https://github.com/tastejs/todomvc) — The same Todo app built in dozens of MVC/MVVM/Flux frameworks
- [RealWorld (Conduit)](https://github.com/gothinkster/realworld) — A full Medium.com clone built in 100+ different frontend and backend stacks, all following the same API spec

---

### 2. Application Architecture (Layer-Level Organization)

#### Clean Architecture

Proposed by Robert C. Martin (Uncle Bob). The core rule: **dependencies always point inward**. Your business entities and use cases never depend on frameworks, databases, or UI — those live in outer rings.

```
┌──────────────────────────────────────────┐
│  Frameworks & Drivers (Web, DB, UI)      │  ← outermost
│  ┌──────────────────────────────────┐    │
│  │  Interface Adapters (Controllers,│    │
│  │  Presenters, Gateways)           │    │
│  │  ┌──────────────────────────┐    │    │
│  │  │  Use Cases / Application │    │    │
│  │  │  ┌──────────────────┐    │    │    │
│  │  │  │   Entities       │    │    │    │  ← innermost
│  │  │  └──────────────────┘    │    │    │
│  │  └──────────────────────────┘    │    │
│  └──────────────────────────────────┘    │
└──────────────────────────────────────────┘
```

**When to use:** Medium-to-large applications where you want to swap frameworks or databases without rewriting business logic; teams that value testability.

**Alternatives:**

| Architecture | Key Difference | Common In |
|-------------|---------------|-----------|
| **Hexagonal (Ports & Adapters)** | Very similar to Clean — defines "ports" (interfaces) and "adapters" (implementations). Coined by Alistair Cockburn. | Java/Spring, Kotlin |
| **Onion Architecture** | Same dependency direction rule as Clean, just different naming (Domain Model → Domain Services → Application Services → Infrastructure). Coined by Jeffrey Palermo. | .NET |
| **Vertical Slice Architecture** | Instead of horizontal layers, each feature is a self-contained "slice" with its own handler, validation, and data access. Less coupling between features. | .NET (MediatR-based), Node.js |
| **Simple Layered (N-Tier)** | Traditional layers: Presentation → Business → Data Access. Dependencies can go both ways (the classic trap). | Legacy enterprise apps |

> Clean, Hexagonal, and Onion are essentially the **same idea** with different vocabulary. The core principle is always dependency inversion.

**Example repos:**

- [jasontaylordev/CleanArchitecture](https://github.com/jasontaylordev/CleanArchitecture) — Clean Architecture template for ASP.NET Core with Angular/React
- [amitshekhariitbhu/go-backend-clean-architecture](https://github.com/amitshekhariitbhu/go-backend-clean-architecture) — Clean Architecture in Go with Gin + MongoDB
- [mattia-battiston/clean-architecture-example](https://github.com/mattia-battiston/clean-architecture-example) — Java example with clear layer separation

---

### 3. System Topology (Deployment-Level Organization)

#### Microservices

Instead of deploying one large application (monolith), you split the system into small, independently deployable services that communicate over the network (HTTP/REST, gRPC, message queues).

**When to use:** Large teams, high-scale systems, when different parts of the system need independent scaling or different tech stacks.

**Alternatives:**

| Topology | Key Difference | When to pick it |
|----------|---------------|-----------------|
| **Monolith** | Single deployable unit. Much simpler to develop, test, and deploy. | Startups, small teams, new projects |
| **Modular Monolith** | One deployable, but internally separated into clear modules with defined boundaries. Best of both worlds. | Growing projects before they need microservices |
| **Serverless (FaaS)** | No server at all — you deploy individual functions that run on demand (AWS Lambda, Azure Functions, Vercel Edge Functions). | Event-driven workloads, APIs with bursty traffic |
| **Service-Oriented Architecture (SOA)** | Predecessor to microservices, with heavier middleware (ESBs). | Legacy enterprise systems |

> **Hot take:** Start with a **monolith** (or modular monolith) for your personal project. Microservices add tremendous operational complexity (networking, service discovery, distributed tracing, eventual consistency). You can always extract services later.

**Example repos:**

- [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo) — A complete microservices e-commerce app by Google 
- [dotnet-architecture/eShopOnContainers](https://github.com/dotnet/eShop) — Microsoft's reference microservices app in .NET 
- [microservices-patterns/ftgo-application](https://github.com/microservices-patterns/ftgo-application) — Companion code for *"Microservices Patterns"* by Chris Richardson


---
