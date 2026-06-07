# Week 18 - Web Security

> **Block 6: Security and Closure**
> This week shifts the conversation from "the app works" to "the app resists predictable attacks." The goal is not to memorize a random list of packages. The goal is to understand the security model behind modern web applications so teams can choose the right controls, libraries, and platforms with intention.

---

## Part 1 - Why Web Security Matters

Security is not a final polish step.

It directly affects:

- User trust
- Data confidentiality
- Business continuity
- Legal and reputational risk
- The real cost of operating a product

Many student projects are small, but the attack patterns are the same ones used against larger systems:

- Weak authentication flows
- Missing authorization checks
- Unsafe database queries
- Unsanitized user input
- Secrets committed to the repository
- Outdated dependencies with known vulnerabilities

> **Key idea:** A professional application is not only usable and deployable. It is also intentionally defensive.

This week should help answer three questions:

1. What are we trying to protect?

2. Where can the system be attacked?

3. Which controls reduce the most common risks?

---

## Part 2 - A Security Mental Model

Before talking about tools, we need a clear mental model.

### Assets

Assets are the things that matter and need protection.

Examples:

- User accounts
- Password hashes
- Personal data
- Payment or order records
- Admin capabilities
- API keys and secrets
- Source code and deployment configuration

### Actors

Not every actor should be trusted equally.

Common actors include:

- Anonymous visitors
- Authenticated users
- Admin users
- Internal services
- Third-party APIs
- Attackers pretending to be normal users

### Entry points

These are the places where data or actions enter the system:

- Browser forms
- URL parameters
- JSON request bodies
- File uploads
- Cookies
- Background jobs
- Webhooks
- Admin panels

### Trust boundaries

A trust boundary is the point where data moves from a less trusted context to a more trusted one.

Examples:

- Browser to backend API
- Public API to database
- Third-party webhook to internal processing logic
- Client-side role display to server-side permission check

### Attack surface

The attack surface is the set of reachable places an attacker can use to influence the system.

```text
User or attacker
   -> browser or script
   -> public routes, forms, APIs, uploads, auth endpoints
   -> application logic
   -> database, storage, queues, third-party services
```

If we do not understand assets, actors, entry points, and trust boundaries, tool choices become shallow.

---

## Part 3 - Secure-by-Design Principles

Security improves when teams adopt a few stable principles early.

| Principle | What it means in practice |
|-----------|---------------------------|
| **Least privilege** | Give users, services, and tokens only the permissions they actually need |
| **Deny by default** | Access should be granted explicitly, not assumed |
| **Validate input** | Treat all external input as untrusted until proven otherwise |
| **Encode output** | Prevent user-controlled content from being interpreted as code |
| **Fail securely** | Errors should not expose stack traces, secrets, or unsafe fallback behavior |
| **Defense in depth** | Use multiple layers such as validation, auth checks, headers, and monitoring |
| **Keep secrets separate** | Secrets do not belong in source control or frontend bundles |
| **Observe security events** | Authentication failures, privilege errors, and suspicious actions should be visible |

### Why these principles matter

A library can help enforce a control, but no library can rescue a broken security model.

For example:

- A validation library helps, but it does not replace authorization checks
- A secure auth provider helps, but it does not fix broken role logic
- A WAF helps, but it does not replace parameterized queries

---

## Part 4 - OWASP Top 10 as a Prioritization Map

The OWASP Top 10 is useful because it organizes common web application risks. It should be treated as a map, not a memorization exercise.

| OWASP area | What it usually looks like in practice |
|------------|----------------------------------------|
| **Broken Access Control** | A user can view, edit, or delete data they do not own |
| **Cryptographic Failures** | Sensitive data is stored or transmitted without proper protection |
| **Injection** | Untrusted input changes the meaning of a query or command |
| **Insecure Design** | The system logic itself enables abuse even if the code looks clean |
| **Security Misconfiguration** | Debug settings, weak headers, open storage, or unsafe defaults |
| **Vulnerable and Outdated Components** | Dependencies include known vulnerabilities |
| **Identification and Authentication Failures** | Weak session handling, token misuse, or poor password practices |
| **Software and Data Integrity Failures** | The build or dependency path can be tampered with |
| **Security Logging and Monitoring Failures** | Incidents happen but the team cannot see them in time |
| **Server-Side Request Forgery** | The server is tricked into making unintended internal or external requests |

### Recommended course emphasis

For this course, the highest-value focus areas are usually:

- Broken access control
- Authentication failures
- Injection
- Security misconfiguration
- Vulnerable dependencies
- Weak logging around security-relevant events
 
---

## Part 5 - XSS, CSRF, and SQL Injection

These three topics remain foundational because they expose core security ideas.

| Risk | What it means | Common mistake | Core prevention |
|------|---------------|----------------|-----------------|
| **XSS** | Untrusted content executes as JavaScript in the browser | Rendering unsafe HTML or trusting user input in the UI | Output encoding, template auto-escaping, sanitization when HTML is required, CSP |
| **CSRF** | A browser sends an unwanted authenticated request | State-changing endpoints trust cookies without anti-CSRF protections | CSRF tokens, `SameSite` cookies, origin checks, avoiding unnecessary cookie-based cross-site flows |
| **SQL Injection** | User input changes the meaning of a SQL query | Building queries with string concatenation | Parameterized queries, ORM query builders, strict validation |

### XSS

XSS is mostly about confusing data with code.

Important lessons:

- Escaping output is usually safer than trying to blacklist dangerous strings
- Frameworks that auto-escape output help, but unsafe escape hatches still exist
- Rendering user-provided HTML should be rare and heavily controlled

If you are using React, normal JSX output is escaped by default. The danger appears when teams bypass that protection and inject raw HTML.

### CSRF

CSRF matters when the browser automatically includes authentication context, especially cookies.

Important lessons:

- Authentication and authorization are not enough by themselves
- The server must verify that a state-changing request is intentional
- Cookie settings such as `HttpOnly`, `Secure`, and `SameSite` are part of the security model

### SQL Injection

SQL injection is one of the clearest examples of why untrusted input must never directly shape executable instructions.

Important lessons:

- Input validation is useful, but parameterization is still required
- ORMs can reduce risk when used correctly
- Dynamic query building still needs discipline

---

## Part 6 - Authentication, Sessions, and Authorization

Security problems often appear here because teams mix different concerns.

### Authentication vs authorization

- **Authentication** answers: who are you?
- **Authorization** answers: what are you allowed to do?

A user can be authenticated and still be unauthorized to access a resource.

### Common mistakes

- Trusting frontend role checks without backend enforcement
- Using long-lived tokens carelessly
- Storing sensitive tokens in unsafe browser locations
- Forgetting ownership checks on update and delete endpoints
- Assuming "logged in" means "allowed"

### Practical rules

- Hash passwords with modern adaptive algorithms
- Protect session cookies with secure attributes
- Re-check authorization on the server for every sensitive action
- Separate ordinary user capabilities from admin capabilities clearly
- Design token expiration and refresh behavior intentionally

### A simple access control example

```text
Wrong model:
Any authenticated user -> can update any project by guessing an ID

Better model:
Authenticated user
   -> request to update project 42
   -> server checks ownership or role
   -> action allowed or denied
```

---

## Part 7 - Secrets, Configuration, and Dependency Hygiene

Many breaches happen without a brilliant attacker. They happen because teams leave important things exposed.

### Secrets

Examples of secrets:

- Database credentials
- JWT signing keys
- API keys
- SMTP passwords
- Cloud access tokens

Core rules:

- Never commit secrets to the repository
- Never hardcode secrets into frontend code
- Use environment variables locally and a secret manager in production when possible
- Rotate exposed secrets instead of only deleting them from the code

### Dependency hygiene

Dependencies improve productivity, but they also extend the attack surface.

Key practices:

- Prefer maintained libraries with active ecosystems
- Remove packages you do not need
- Review high-impact dependency updates regularly
- Scan for known vulnerabilities
- Be careful with install scripts and untrusted packages

### Supply chain awareness

If a package, build step, or CI pipeline is compromised, the application can become compromised even if your own business logic looks fine.

That is why security also includes:

- Lockfiles
- Trusted package sources
- Protected CI/CD workflows
- Review of dependency changes in pull requests

---

## Part 8 - Browser, API, and Infrastructure Defenses

Good security is layered. The browser, backend, and infrastructure should all contribute.

### Browser-facing defenses

- HTTPS for transport security
- Content Security Policy to reduce XSS impact
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy`
- Frame protections such as `frame-ancestors` or related header strategies
- Secure cookie settings

### API and backend defenses

- Request validation at the boundary
- Authorization checks close to protected actions
- Rate limiting on auth and public write endpoints
- Safe file upload handling
- Parameterized database access
- Error handling that avoids leaking internal details

### Infrastructure and platform defenses

- Secret stores
- Network restrictions where appropriate
- Managed TLS certificates
- Web application firewalls as defense in depth
- Audit logs and centralized monitoring

### One important clarification

**CORS is not an authentication system.**

CORS is a browser policy about which origins can make certain cross-origin requests. It does not replace server-side authorization.

---

## Part 9 - From Concepts to Tools and Libraries

Once the principles are clear, tools become easier to evaluate.

> **Important rule:** Choose tools that reinforce your security model. Do not expect tools to define it for you.

| Need | Security concept first | Example technologies or libraries |
|------|------------------------|-----------------------------------|
| **Input validation** | Define strict data contracts at API boundaries | Zod, Joi, Pydantic, FluentValidation, framework-native model validation |
| **Safe database access** | Keep user input separate from executable queries | Prisma, Drizzle, EF Core, SQLAlchemy, parameterized SQL APIs |
| **Password storage** | Use slow adaptive hashing, never plain text or fast hashes | Argon2, bcrypt libraries in your stack |
| **Security headers** | Harden browser behavior through explicit response policies | Helmet for Express, ASP.NET Core middleware, Nginx or Caddy config |
| **HTML sanitization** | Sanitize only when rendering limited trusted HTML is truly necessary | DOMPurify |
| **Rate limiting** | Reduce brute force and abuse on exposed endpoints | express-rate-limit, ASP.NET Core rate limiting middleware, API gateway features |
| **Dependency scanning** | Find known vulnerabilities in packages and images | Dependabot, npm audit, pnpm audit, Snyk, OSV-Scanner, Trivy |
| **Dynamic security testing** | Test the running app for common attack paths | OWASP ZAP, Burp Suite Community Edition |
| **Secret management** | Keep secrets outside code and centralize rotation | Azure Key Vault, AWS Secrets Manager, Google Secret Manager |
| **Edge protection / WAF** | Filter obvious malicious traffic before it reaches the app | Cloudflare WAF, AWS WAF, Azure WAF |
| **Security monitoring** | Detect auth abuse, suspicious failures, and incident signals | Sentry, Application Insights, Cloud Logging, Datadog |

We should be able to explain:

- Which risk a tool addresses
- Which layer it belongs to
- Which limitations it still has
- Which concept would still matter if the tool changed tomorrow

That keeps the course focused on transferable security reasoning.

---

### Security report mindset

The goal is not to claim "the app is fully secure." The goal is to show that the team can identify risks, reduce them, and explain the remaining ones honestly.

---

## Part 11 - Suggested Exercise and Deliverable

### Suggested exercise

Run a structured security review of the personal project.

Possible workflow:

1. List sensitive assets and public entry points
2. Review authentication and authorization flows
3. Inspect forms, APIs, uploads, and raw HTML rendering paths
4. Check database access patterns for injection risk
5. Review secrets handling and environment configuration
6. Run a dependency vulnerability scan
7. Apply at least three concrete security fixes
8. Document what changed and why

### Reflection questions

1. Which issue in your project was the highest-risk one, and why?
2. Which controls came from better design decisions rather than from adding a package?
3. Which parts of your security setup are framework-specific, and which concepts transfer anywhere?
4. If your app were opened to real public traffic tomorrow, what would still worry you most?

---

## Recommended Reading and Exploration

- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - The most common categories of web application risk
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) - Practical implementation guidance across many topics
- [OWASP ZAP Docs](https://www.zaproxy.org/docs/) - Dynamic application security testing for running web apps
- [MDN Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) - A strong starting point for browser-side defense
- [MDN HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) - Session and cookie security fundamentals
- [MDN CORS Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) - Understand what CORS does and does not solve