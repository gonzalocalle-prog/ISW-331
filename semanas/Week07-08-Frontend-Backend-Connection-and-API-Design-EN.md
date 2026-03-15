# Weeks 7 & 8 — The Frontend-Backend Connection & RESTful API Design

## Part 1 — How the Web Actually Works (The Mental Model)

Before writing a single endpoint, you need a clear mental model of what happens when your frontend talks to your backend.

### The Request-Response Cycle

Every interaction between a browser (or mobile app) and a server follows this pattern:

```
┌──────────┐         HTTP Request           ┌──────────┐
│          │  ──────────────────────────▶   │          │
│  Client  │    method, URL, headers,       │  Server  │
│(Browser) │    body                        │(Your API)│
│          │  ◀──────────────────────────   │          │
└──────────┘         HTTP Response          └──────────┘
                  status code, headers,
                  body (usually JSON)
```

This cycle is **stateless** — the server does not remember previous requests. Every request must carry all the information the server needs (authentication token, parameters, etc.).

### Anatomy of an HTTP Request

| Part | Purpose | Example |
|------|---------|---------|
| **Method (verb)** | What action to perform | `GET`, `POST`, `PUT`, `DELETE` |
| **URL (path)** | Which resource to act on | `/api/products/42` |
| **Headers** | Metadata about the request | `Authorization: Bearer <token>`, `Content-Type: application/json` |
| **Body** | Data payload (not all methods) | `{ "name": "Widget", "price": 9.99 }` |

### Anatomy of an HTTP Response

| Part | Purpose | Example |
|------|---------|---------|
| **Status code** | Did it work? What went wrong? | `200 OK`, `404 Not Found`, `500 Internal Server Error` |
| **Headers** | Metadata about the response | `Content-Type: application/json`, `Cache-Control: max-age=3600` |
| **Body** | The actual data | `{ "id": 42, "name": "Widget" }` |

### HTTP Status Codes — The Groups That Matter

You don't need to memorize all of them, but you must understand the **families**:

| Range | Meaning | Key Codes |
|-------|---------|-----------|
| **2xx** | Success | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | Redirection | `301 Moved Permanently`, `304 Not Modified` |
| **4xx** | Client error (your frontend's fault) | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity` |
| **5xx** | Server error (your backend's fault) | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

> **Key insight:** A well-designed API never returns `200 OK` with an error message inside the body. The status code should always tell the truth.

---

## Part 2 — REST: The Dominant API Paradigm

### What is REST?

**REST** (Representational State Transfer) is not a protocol or a library — it's a set of **architectural constraints** defined by Roy Fielding in his 2000 PhD dissertation. When people say "REST API" they usually mean an HTTP API that follows certain conventions.

### The Six Constraints of REST

| Constraint | What It Means | Why It Matters |
|-----------|---------------|----------------|
| **Client-Server** | The client and server are separate systems with independent concerns | You can change the frontend without touching the backend and vice versa |
| **Stateless** | Each request contains all the information needed to process it | Simplifies server design, enables horizontal scaling |
| **Cacheable** | Responses declare whether they can be cached | Reduces server load, improves performance |
| **Uniform Interface** | Resources are identified by URLs, manipulated through representations (JSON), and self-descriptive | Consistency across all endpoints |
| **Layered System** | The client doesn't need to know if it's talking to the final server or a proxy/load balancer | Enables CDNs, API gateways, security layers |
| **Code on Demand** (optional) | Server can send executable code to the client | JavaScript in browsers is technically this |

### Resources and URLs

The most important concept in REST is the **resource**. A resource is any thing your system exposes — a user, a product, an order, a comment.

**Naming conventions:**

| Pattern | Example | Purpose |
|---------|---------|---------|
| Collection | `/api/products` | All products |
| Single item | `/api/products/42` | Product with ID 42 |
| Sub-resource | `/api/products/42/reviews` | Reviews for product 42 |
| Filtering | `/api/products?category=electronics&sort=price` | Filtered collection |

**Naming rules:**

- Use **nouns**, not verbs: `/api/products` not `/api/getProducts`
- Use **plural**: `/api/products` not `/api/product`
- Use **kebab-case**: `/api/order-items` not `/api/orderItems`
- Nest logically but don't go deeper than 2 levels: `/api/users/5/orders` is fine, `/api/users/5/orders/10/items/3/reviews` is too deep — flatten it

### HTTP Methods Map to CRUD

| Method | CRUD Operation | On a Collection (`/products`) | On an Item (`/products/42`) |
|--------|---------------|-------------------------------|----------------------------|
| **GET** | Read | List all products | Get one product |
| **POST** | Create | Create a new product | *(rarely used)* |
| **PUT** | Update (full) | *(rarely used)* | Replace entire product |
| **PATCH** | Update (partial) | *(rarely used)* | Update some fields |
| **DELETE** | Delete | *(rarely used)* | Delete one product |

> **PUT vs PATCH:** `PUT` replaces the entire resource (you must send all fields). `PATCH` updates only the fields you send. Most real-world apps use `PATCH` more often than `PUT`.

---

## Part 3 — API Design Decisions

### Request and Response Format

**JSON** is the universal standard. Every request with a body should send `Content-Type: application/json` and every response should return JSON.

#### Consistent Response Shape

Pick one shape and use it everywhere. A common pattern:

**Success:**
```json
{
  "data": { "id": 42, "name": "Widget", "price": 9.99 },
  "meta": { "requestId": "abc-123" }
}
```

**Error:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Price must be greater than 0",
    "details": [
      { "field": "price", "issue": "Must be positive" }
    ]
  }
}
```

> **Why consistency matters:** Your frontend code that handles API responses can be generic. If every error has the same shape, you write one error handler — not one per endpoint.

### Pagination

Never return all records in a collection. Two common strategies:

| Strategy | How It Works | Best For |
|----------|-------------|----------|
| **Offset-based** | `?page=2&limit=20` — skip 20, take 20 | Simple UIs with page numbers |
| **Cursor-based** | `?cursor=abc123&limit=20` — start after this item | Infinite scroll, real-time feeds, large datasets |

Cursor-based is more performant at scale because the database doesn't need to count through all skipped rows.

### Filtering and Sorting

Keep it simple and predictable:

- Filtering: `?status=active&category=electronics`
- Sorting: `?sort=price` (ascending), `?sort=-price` (descending)
- Search: `?q=wireless+headphones`

### Versioning

APIs evolve. When you make breaking changes, old clients shouldn't break. Common strategies:

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL path** | `/api/v1/products` | Obvious, easy to understand | URL pollution |
| **Header** | `Accept: application/vnd.myapp.v2+json` | Clean URLs | Harder to discover |
| **Query param** | `/api/products?version=2` | Easy to test | Easy to forget |

> For personal/small projects, URL path versioning (`/api/v1/`) is the simplest and most common choice.

---

## Part 4 — Beyond REST: Knowing What Else Exists

REST is not the only way to design APIs. Understanding alternatives helps you make informed choices.

| Paradigm | Core Idea | Best For | Trade-offs |
|----------|----------|----------|------------|
| **REST** | Resources + HTTP verbs | CRUD applications, public APIs | Can require many round-trips for complex data |
| **GraphQL** | Client specifies exactly what data it needs in a query language | Complex UIs that aggregate data from many sources | Caching is harder, learning curve |
| **gRPC** | Binary protocol (Protocol Buffers) with strong typing and streaming | Microservice-to-microservice, high performance | Not browser-friendly without a proxy |
| **WebSocket** | Persistent bidirectional connection | Real-time features (chat, live updates, gaming) | More complex server infrastructure |
| **Server-Sent Events (SSE)** | Server pushes updates to client over HTTP | Notifications, live feeds (one-way) | Only server-to-client, limited browser connections |

> **For this course:** REST is the right choice. It's the industry default, every framework supports it natively, and it maps directly to CRUD operations.

---

## Part 5 — Backend Architecture Concepts

### What Does a Backend Actually Do?

A backend is a program that:

1. **Receives** HTTP requests
2. **Validates** the input
3. **Executes** business logic (rules, calculations, decisions)
4. **Reads/writes** data to a database
5. **Returns** an HTTP response

That's it. Everything else — authentication, logging, caching, rate limiting — is a variation or addition to these five steps.

### The Layered Architecture Pattern

Regardless of language or framework, well-organized backends separate concerns into layers:

```
┌──────────────────────────────────────────────────────┐
│                   Routes / Controllers               │
│          (receive request, return response)          │
├──────────────────────────────────────────────────────┤
│                   Service / Use Case                 │
│          (business logic, orchestration)             │
├──────────────────────────────────────────────────────┤
│                   Repository / Data Access           │
│          (talk to the database)                      │
├──────────────────────────────────────────────────────┤
│                   Database                           │
│          (PostgreSQL, MongoDB, etc.)                 │
└──────────────────────────────────────────────────────┘
```

| Layer | Responsibility | Knows About |
|-------|---------------|-------------|
| **Routes / Controllers** | Parse HTTP request, call the right service, format HTTP response | HTTP concepts, request/response shapes |
| **Service / Use Case** | Business rules: "a user can only cancel their own order", "discount applies if cart > $50" | Domain entities, repositories (through interfaces) |
| **Repository / Data Access** | Translate between your application's objects and the database | Database queries, connection details |

**Why separate layers?**

- You can change the database without rewriting business logic
- You can test business logic without a running database (mock the repository)
- Different team members can work on different layers simultaneously
- When something breaks, you know which layer to look at

### Clean Architecture — Layered Architecture with a Strict Rule

Clean Architecture (Robert C. Martin) takes the layered idea and adds one **non-negotiable rule**: dependencies always point **inward**. The inner layers never know about the outer layers.

```
┌──────────────────────────────────────────────────────────┐
│             Frameworks & Drivers (outermost)             │
│   Web framework, database driver, HTTP client, UI        │
│  ┌──────────────────────────────────────────────────┐    │
│  │          Interface Adapters                      │    │
│  │   Controllers, Presenters, Gateways, Repos       │    │
│  │  ┌──────────────────────────────────────────┐    │    │
│  │  │        Application / Use Cases           │    │    │
│  │  │   "Create Order", "Cancel Subscription"  │    │    │
│  │  │  ┌──────────────────────────────────┐    │    │    │
│  │  │  │       Entities / Domain          │    │    │    │
│  │  │  │   Business rules, core objects   │    │    │    │
│  │  │  └──────────────────────────────────┘    │    │    │
│  │  └──────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
                    dependencies point → inward
```

| Ring | Contains | Depends On |
|------|----------|-----------|
| **Entities** (center) | Business objects and rules that exist regardless of any application | Nothing |
| **Use Cases** | Application-specific business rules ("what the system does") | Entities only |
| **Adapters** | Translators between use cases and external tools (controllers, repos) | Use Cases and Entities |
| **Frameworks** (outer) | The actual libraries, DB drivers, web servers | Everything inward |

**The key difference from plain layered:** In layered architecture, the service layer often imports the database library directly. In Clean Architecture, the use case defines an **interface** (port) and the outer layer provides the **implementation** (adapter). This is known as **Dependency Inversion**.

```
Plain Layered:     Service → imports → PostgreSQL driver directly
Clean:             UseCase → depends on → IOrderRepository (interface)
                   PostgresOrderRepository (outer layer) implements it
```

> **When to use Clean Architecture:** When you want to be able to swap databases, frameworks, or external services without rewriting business logic. Overkill for a simple CRUD app. Valuable when business rules are complex.

---

### Domain-Driven Design (DDD) — A Different Kind of Tool

DDD is **not** an architecture pattern. It's a **design philosophy** created by Eric Evans (2003). It answers a different question:

| Question | Answered By |
|----------|------------|
| Where does code go? | Layered Architecture |
| Which direction do dependencies flow? | Clean Architecture |
| **What do I name things and how do I model the problem?** | **DDD** |

DDD says: the structure of your code should mirror the structure of the **business problem**, not the technical stack.

#### The Building Blocks of DDD

| Concept | What It Is | Example |
|---------|-----------|---------|
| **Entity** | An object with a unique identity that persists over time | `Order` (has an ID, changes status over its lifetime) |
| **Value Object** | An object defined by its attributes, not by identity. Immutable. | `Money(100, "USD")`, `EmailAddress("a@b.com")` |
| **Aggregate** | A cluster of entities and value objects treated as a single unit for data changes. Has one **root entity**. | `Order` (root) contains `OrderLine` items — you never modify an `OrderLine` directly, only through the `Order` |
| **Repository** | An interface for retrieving and storing aggregates (hides the database) | `IOrderRepository.findById(id)` |
| **Service** | Logic that doesn't naturally belong to any entity | `PaymentService.processPayment(order)` |
| **Domain Event** | Something that happened in the domain that other parts of the system care about | `OrderPlaced`, `PaymentFailed` |

#### Ubiquitous Language

The most powerful idea in DDD is not technical — it's communication. The **ubiquitous language** is a shared vocabulary between developers and business stakeholders:

- If the business calls it an "enrollment", your code should have an `Enrollment` class — not a `Registration` or `Signup`
- If the business says "cancel", your method is `order.cancel()` — not `order.setStatus("cancelled")`
- If the business distinguishes between "refund" and "credit", your code should too

> **Why this matters:** When your code uses the same words as the business, bugs in requirements become visible in the code, and developers can talk to stakeholders without "translating."

#### Bounded Contexts

In a large system, the same word can mean different things in different parts of the business:

| Word | In "Sales" Context | In "Shipping" Context |
|------|-------------------|----------------------|
| **Product** | Has a price, description, discount rules | Has a weight, dimensions, warehouse location |
| **Customer** | Has payment info, purchase history | Has a shipping address, delivery preferences |

DDD says: don't force a single `Product` class to carry all meanings. Each **bounded context** has its own model, its own definition of `Product`, and they communicate through well-defined interfaces.

```
┌──────────────────┐         ┌──────────────────┐
│  Sales Context   │  event  │ Shipping Context │
│                  │ ──────▶ │                  │
│  Product: price, │         │  Product: weight,│
│  discount, SKU   │         │  dimensions      │
│  Customer: $$$   │         │  Customer: addr  │
└──────────────────┘         └──────────────────┘
```

> **For your project:** You almost certainly have one bounded context (your entire app). But understanding this concept prepares you for larger systems.

---

### How They All Relate — The Map

These three concepts are **complementary**, not competing. They operate at different levels:

```
┌─────────────────────────────────────────────────────────┐
│  DDD (Design Philosophy)                                │
│  "Model code around business concepts"                  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Clean Architecture (Dependency Rule)            │    │
│  │  "Dependencies always point inward"              │    │
│  │                                                  │    │
│  │  ┌──────────────────────────────────────────┐   │    │
│  │  │  Layered Architecture (Structure)         │   │    │
│  │  │  "Separate controllers, services, repos"  │   │    │
│  │  └──────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

| Approach | You Can Use It Without... | Best Combined With... |
|----------|--------------------------|----------------------|
| **Layered** | Clean or DDD — just stack the layers | Clean (to enforce direction) |
| **Clean** | DDD — just enforce the dependency rule | DDD (to model the inner layers well) |
| **DDD** | Clean — apply the naming/modeling ideas anywhere | Clean (DDD entities live in the center ring) |

#### Practical Recommendation

| Project Complexity | Recommended Approach |
|-------------------|---------------------|
| Simple CRUD (< 5 entities) | **Layered Architecture** — separate controllers, services, and repos. Don't overthink it. |
| Medium (5–15 entities, some business rules) | **Clean Architecture** — enforce the dependency rule, use interfaces for repos. Apply DDD naming (ubiquitous language, entities vs value objects). |
| Complex (many entities, complex rules, multiple user roles) | **Full DDD + Clean** — aggregates, domain events, bounded contexts. |

> **Start simple.** You can always refactor from layered → clean → DDD as your project grows. Starting with full DDD on a simple CRUD app adds complexity for no benefit.

---

### Middleware: The Pipeline Pattern

Most backend frameworks use a **middleware pipeline** — functions that run in sequence before and after each request:

```
Request → [Logging] → [Auth] → [Validation] → [Controller] → Response
                                                      ↓
Response ← [Error Handler] ← [CORS Headers] ← ──────┘
```

Each middleware has a single responsibility:
- **Logging:** Record what request came in
- **Authentication:** Verify the token, attach the user to the request
- **Validation:** Check if the body matches expected schema
- **Error handler:** Catch any thrown errors, format a proper error response
- **CORS:** Add headers that allow the browser to call the API from a different domain

> **Key insight:** Middleware is the **separation of concerns** principle applied to request processing. Each concern is isolated and reusable.

### Input Validation: Trust Nothing

Every piece of data from the outside world must be validated before it reaches your business logic. This is a **security boundary**.

What to validate:

| Check | Why | Example |
|-------|-----|---------|
| **Type** | Is it the right data type? | Expected a number, got a string |
| **Presence** | Is the required field present? | Missing `email` in registration |
| **Format** | Does it match the expected pattern? | Email must contain `@` |
| **Range** | Is the value within acceptable bounds? | Price must be between 0.01 and 99999 |
| **Length** | Is the string within size limits? | Username must be 3–30 characters |
| **Business rules** | Does it make sense in context? | Can't set end date before start date |

> **Validation libraries** exist in every language (Joi, Zod, FluentValidation, Pydantic, etc.). Don't write validation logic by hand — use a schema-based library.

### Error Handling Strategy

Errors are not exceptional — they're expected. A good backend has a clear strategy:

| Error Type | Who Caused It | Status Code | Example |
|-----------|--------------|-------------|---------|
| **Validation error** | Client | `400` or `422` | Missing required field |
| **Authentication error** | Client | `401` | Invalid or expired token |
| **Authorization error** | Client | `403` | User doesn't have permission |
| **Not found** | Client | `404` | Requested resource doesn't exist |
| **Business logic error** | Client | `409` or `422` | Can't cancel an already shipped order |
| **Unexpected error** | Server | `500` | Database connection lost, null pointer |

> **Rule:** Never expose internal error details (stack traces, SQL queries) to the client. Log them on the server, return a generic message to the client.

---

## Part 6 — Connecting Frontend to Backend

### The Integration Points

When your frontend talks to your backend, several things must align:

| Concern | Question | Solution |
|---------|----------|----------|
| **URL** | Where is the API? | Environment variables (`API_URL`), never hardcoded |
| **Authentication** | How does the server know who I am? | Send a token in the `Authorization` header |
| **Data shape** | What does the request/response look like? | Agree on a contract (API documentation) |
| **Error handling** | What if the request fails? | Handle network errors, 4xx errors, and 5xx errors differently |
| **Loading states** | What does the user see while waiting? | Loading indicators, skeleton screens |
| **CORS** | Will the browser allow the request? | Backend must send proper CORS headers |

### CORS — The Most Confusing Beginner Problem

**CORS** (Cross-Origin Resource Sharing) is a browser security mechanism. It prevents `evil-site.com` from making requests to `your-bank.com` using your cookies.

The problem hits you when:
- Your frontend runs on `http://localhost:3000`
- Your backend runs on `http://localhost:8080`
- These are **different origins** (different ports count!)

The solution: your backend must include headers that explicitly allow your frontend's origin:

```
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

Every backend framework has a CORS middleware/package that handles this. You configure it once.

> **Common mistake:** Setting `Access-Control-Allow-Origin: *` in production. This allows any website to call your API. In production, whitelist only your frontend's domain.

### Environment Variables: Keeping Secrets Out of Code

Never put URLs, API keys, database passwords, or tokens directly in your code. Use environment variables:

| Environment | `API_URL` Value | `DB_HOST` Value |
|-------------|----------------|-----------------|
| Development | `http://localhost:8080` | `localhost` |
| Staging | `https://api-staging.myapp.com` | `staging-db.myapp.com` |
| Production | `https://api.myapp.com` | `prod-db.myapp.com` |

Your code reads from the environment, never from hardcoded strings. This way:
- You never accidentally commit secrets to Git
- The same code runs in all environments
- You can change configuration without changing code

---

## Part 7 — API Documentation

An API without documentation is an API nobody can use (including future you).

### Why Document?

- Your frontend team (or your future self) needs to know what endpoints exist, what they expect, and what they return
- Consumers of your API shouldn't need to read your source code
- Documentation catches design mistakes early — if you can't describe the endpoint clearly, the design might be wrong

### The OpenAPI Standard (Swagger)

**OpenAPI** is the industry standard for describing REST APIs. It's a YAML or JSON file that describes every endpoint, parameter, request body, and response.

| Component | What It Describes |
|-----------|------------------|
| **Paths** | Each endpoint URL and its methods |
| **Parameters** | Query parameters, path parameters, headers |
| **Request body** | Shape of the JSON body for POST/PUT/PATCH |
| **Responses** | Status codes and their response shapes |
| **Schemas** | Reusable data type definitions |
| **Security** | Authentication schemes (Bearer token, API key) |

> There are two approaches: **design-first** (write the spec, then implement) or **code-first** (annotate your code, generate the spec). Both are valid. For beginners, code-first is easier because you write code and get documentation for free.

### Minimal Documentation Checklist

Even if you don't use OpenAPI, every endpoint should have:

- [ ] **URL and method** — `GET /api/products`
- [ ] **Description** — What does this endpoint do?
- [ ] **Request parameters** — What query params or path params does it accept?
- [ ] **Request body** — What JSON shape does it expect? (for POST/PUT/PATCH)
- [ ] **Success response** — What status code and body does it return?
- [ ] **Error responses** — What can go wrong and what does the error look like?
- [ ] **Authentication** — Does it require a token? What role?

---

## Part 8 — Preparing for the Exam Demo

### What You Should Be Able to Explain

The exam is not just about showing working code. You need to demonstrate **understanding**. Here's what you should be ready to answer:

#### About Your Architecture

- Why did you choose this folder structure?
- How does data flow from the user's click to the API call and back to the screen?
- What state management approach are you using and why?
- If you had to add a new feature, where would the code go?

#### About Your Frontend

- How does your authentication flow work? (login → token storage → sending token → handling expiry)
- Why did you choose this routing structure?
- How do you handle loading and error states?
- What happens if the API is down?

#### About Your API Connection

- What endpoints does your frontend call?
- Why did you structure the URLs this way?
- What does the data look like going in and coming out?
- How are you handling CORS?

### Demo Checklist

Before presenting, verify:

- [ ] Application runs from a clean start (clone → install → run)
- [ ] You can show the complete user flow (login → main feature → logout)
- [ ] Error handling is visible (disconnect API, show what happens)
- [ ] You can navigate your codebase efficiently during live questions
- [ ] Your architecture diagram (C4) matches the actual code structure
- [ ] Your README explains how to run the project

---

## Recommended Reading

- [HTTP — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP) — The definitive reference for HTTP concepts
- [REST API Tutorial](https://restfulapi.net/) — Practical REST design guidance
- [Roy Fielding's Dissertation, Chapter 5](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) — The original REST definition
- [CORS — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) — Complete CORS explanation with diagrams
- [Swagger/OpenAPI Documentation](https://swagger.io/docs/) — Getting started with API documentation
- [Best Practices for REST API Design](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/) — Stack Overflow's practical guide
- [12-Factor App: Config](https://12factor.net/config) — Why configuration belongs in the environment
