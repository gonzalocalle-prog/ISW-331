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

## Part 2 — Modern CSS: Why Tailwind Is #1 and the Engineering Behind It

### The Problem Tailwind Solves

Traditional CSS has a fundamental scaling problem:

1. **Naming is hard** — Coming up with semantic class names (`.sidebar-inner-wrapper-card-header`) gets painful fast.
2. **CSS grows linearly** — Every new feature adds new CSS. Old CSS is scary to delete because you don't know what depends on it.
3. **Global scope** — Every CSS class is global by default, leading to unexpected side effects when styles collide.
4. **Context switching** — You constantly jump between HTML and CSS files.

### What Is Utility-First CSS?

Instead of writing custom CSS classes, you compose small single-purpose classes directly in your markup:

```html
<!-- Traditional CSS -->
<button class="primary-btn">Save</button>
<style>
  .primary-btn {
    background-color: blue;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 0.375rem;
    font-weight: 600;
  }
  .primary-btn:hover {
    background-color: darkblue;
  }
</style>

<!-- Tailwind (Utility-first CSS) -->
<button class="bg-blue-500 text-white px-4 py-2 rounded-md font-semibold hover:bg-blue-700">
  Save
</button>
```

### The Engineering Behind Tailwind

What actually makes Tailwind special is not just the utility classes — it's the **compiler** behind them:

#### 1. Just-In-Time (JIT) Compilation
Tailwind does **not** ship a giant CSS file with every possible class. Instead, it **scans your source code** (HTML, JSX, Vue templates, etc.), finds every class name you actually use, and generates **only that CSS**. The result:

- Production CSS files are typically **5-15 KB** (gzipped), even for large apps.
- You can use arbitrary values like `bg-[#316ff6]` or `grid-cols-[2fr_1fr]` and Tailwind generates the CSS on the fly.

#### 2. Design System as Configuration
Tailwind's `tailwind.config.js` (or CSS-based theming in v4) acts as a single source of truth for your design tokens — colors, spacing scale, font sizes, breakpoints. Teams get consistency without a separate design system tool.

#### 3. CSS Variables Under the Hood
Internally, features like composition (e.g., stacking `blur-sm` and `grayscale`) work through CSS custom properties:
```css
.blur-sm {
  --tw-blur: blur(var(--blur-sm));
  filter: var(--tw-blur,) var(--tw-brightness,) var(--tw-grayscale,);
}
```
Each utility sets *its own* variable. The `filter` property reads all of them — whatever isn't set falls back to nothing.

#### 4. Variant System (Zero Runtime)
Pseudo-states (`hover:`, `focus:`, `active:`), responsive (`sm:`, `md:`, `lg:`), dark mode (`dark:`), and even complex selectors (`group-hover:`) are all **compile-time** — no JavaScript, no runtime cost. Each variant just wraps the utility in the corresponding CSS media query or pseudo-selector.

### Why Tailwind Won

| Factor | Why it matters |
|--------|---------------|
| **Speed of development** | No naming, no file switching. Designs come together very fast. |
| **Safe changes** | Adding/removing a class only affects *that* element. No fear of breaking other pages. |
| **CSS stops growing** | Utility classes are maximally reusable. A 100-page app might only have 8 KB of CSS. |
| **Excellent DX** | IntelliSense plugin for VS Code, automatic class sorting, responsive preview. |
| **Framework agnostic** | Works with React, Vue, Svelte, Angular, plain HTML, server-rendered templates — anything that produces HTML. |
| **Strong ecosystem** | Headless UI, Tailwind Plus (formerly Tailwind UI), Heroicons, DaisyUI, Flowbite, etc. |

---

### CSS Alternatives — The Full Landscape

CSS approaches fall into several categories. Here is how they compare:

#### Category 1: Utility-First Frameworks

| Framework | How It Works | Trade-off |
|-----------|-------------|-----------|
| **[Tailwind CSS](https://tailwindcss.com/)** | Scans source files, generates only used utilities at build time. | Verbose markup, but near-zero CSS output. **Industry standard.** |
| **[UnoCSS](https://unocss.dev/)** | Same utility-first idea, but as a highly extensible engine. Tailwind-compatible preset available. Extremely fast (Vite-native). | Smaller community, less prescriptive. |
| **[Windi CSS](https://windicss.org/)** | Inspired Tailwind's JIT mode. Now deprecated (team joined UnoCSS). | Archived — historical reference only. |

#### Category 2: CSS-in-JS (Runtime)

| Library | How It Works | Trade-off |
|---------|-------------|-----------|
| **[styled-components](https://styled-components.com/)** | Tagged template literals in JS. Generates unique class names at runtime. | Runtime cost: styles are computed in the browser's JS thread. Bundle size overhead. |
| **[Emotion](https://emotion.sh/)** | Similar to styled-components, with a `css` prop and `styled` API. | Same runtime overhead. React-centric. |

> **Industry trend:** Runtime CSS-in-JS is declining. The React core team itself recommended moving away from it. The JS thread should not be busy generating CSS.

#### Category 3: CSS-in-JS (Zero Runtime / Build-Time Extraction)

| Library | How It Works | Trade-off |
|---------|-------------|-----------|
| **[Vanilla Extract](https://vanilla-extract.style/)** | Write styles in TypeScript files. Extracted to static CSS at build time. Full type safety. | More boilerplate than Tailwind. Requires build integration. |
| **[Panda CSS](https://panda-css.com/)** | Utility-first + CSS-in-JS hybrid. Write style objects in TS, extract to atomic CSS at build. | Newer, smaller community. Configuration-heavy. |
| **[Linaria](https://linaria.dev/)** | Write CSS-in-JS syntax, zero runtime extraction. | Less actively maintained. |
| **[StyleX](https://stylexjs.com/)** | Meta's (Facebook) internal solution, open-sourced. Atomic CSS from JS objects at build time. | Tight coupling with Meta's tooling philosophy. |

#### Category 4: CSS Modules

| Approach | How It Works | Trade-off |
|----------|-------------|-----------|
| **[CSS Modules](https://github.com/css-modules/css-modules)** | Standard `.module.css` files. Class names are locally scoped by the build tool (Vite, Webpack). | You still write regular CSS (naming problem remains), but no global collisions. |

#### Category 5: Component Libraries (Pre-styled)

| Library | How It Works | Trade-off |
|---------|-------------|-----------|
| **[Bootstrap](https://getbootstrap.com/)** | Pre-built components with opinionated design. Ships full CSS by default. | Quick prototypes, but hard to customize. "Looks like Bootstrap." |
| **[Bulma](https://bulma.io/)** | Flexbox-based, no JS, clean syntax. | Smaller ecosystem than Bootstrap or Tailwind. |
| **[DaisyUI](https://daisyui.com/)** | Component classes built on top of Tailwind. `class="btn btn-primary"` | Best of both worlds: Tailwind's engine + semantic component names. |
| **[shadcn/ui](https://ui.shadcn.com/)** | Copy-paste React components using Tailwind + Radix. Not a package — you own the code. | React-only. Requires Tailwind knowledge. |

---

### Decision Guide: Which CSS Approach to Use?

```
Do you want pre-built component styling?
├── Yes → Bootstrap, DaisyUI, or shadcn/ui
└── No → Do you want to write styles in JS/TS?
    ├── Yes → Is runtime cost acceptable?
    │   ├── Yes → styled-components or Emotion
    │   └── No → Vanilla Extract, Panda CSS, or StyleX
    └── No → Do you want a design-system-first approach?
        ├── Yes → Tailwind CSS (recommended) or UnoCSS
        └── No → CSS Modules or plain CSS
```
