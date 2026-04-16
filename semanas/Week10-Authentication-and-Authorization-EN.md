# Week 10 — Authentication and Authorization

> **Block 3: Backend and Databases**
> This week covers how to secure your application: verifying who a user is (authentication), controlling what they can do (authorization), and implementing these concepts with industry-standard tools like JWTs, sessions, OAuth, and Role-Based Access Control (RBAC).

---

## Part 1 — Authentication vs Authorization: Two Different Questions

Before writing any code, you need to understand that security answers two fundamentally different questions:

| Concept | Question It Answers | Analogy |
|---------|-------------------|---------|
| **Authentication (AuthN)** | *Who are you?* | Showing your ID at the airport |
| **Authorization (AuthZ)** | *What are you allowed to do?* | Your boarding pass determines which gate you can board at |

These are **not** the same thing, and they happen at different stages:

```
User sends request
       │
       ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Authentication  │────▶│  Authorization   │────▶│  Access Granted  │
│  "Who is this?"  │     │  "Can they do    │     │  (or Denied)     │
│                  │     │   this action?"  │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
       │                        │
       ▼                        ▼
  401 Unauthorized         403 Forbidden
  (we don't know you)     (we know you, but no)
```

> **Key insight:** A `401 Unauthorized` response actually means "unauthenticated" — the server doesn't know who you are. A `403 Forbidden` means "authenticated but not authorized" — the server knows who you are, but you don't have permission.

---

## Part 2 — Authentication Strategies

There are several ways to verify a user's identity. Each has trade-offs.

### Strategy 1: Session-Based Authentication

The oldest and most straightforward approach. The server maintains a record of who is logged in.

**How it works:**

```
1. User sends credentials (email + password)

   POST /api/login
   { "email": "alice@mail.com", "password": "s3cret" }

2. Server verifies credentials against the database

3. Server creates a session (stored in memory, file, or database)
   Session ID: "abc123" → { userId: 1, role: "admin", createdAt: ... }

4. Server sends the session ID back in a cookie
   Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict

5. Browser automatically sends the cookie on every subsequent request
   Cookie: sessionId=abc123

6. Server looks up the session ID to identify the user
```

```
┌──────────┐                              ┌──────────┐
│  Client  │   POST /login (credentials)  │  Server  │
│(Browser) │  ──────────────────────── ─▶ │          │
│          │                               │ Creates  │
│          │  ◀────────────────────────── │ session  │
│          │   Set-Cookie: sessionId=abc   │          │
│          │                               │          │
│          │   GET /api/profile            │          │
│          │   Cookie: sessionId=abc       │          │
│          │  ─────────────────────────▶  │ Looks up │
│          │                               │ session  │
│          │  ◀────────────────────────── │          │
│          │   { "name": "Alice", ... }   │          │
└──────────┘                              └──────────┘
                                          ┌──────────┐
                                          │ Session  │
                                          │  Store   │
                                          │ abc → {  │
                                          │  user: 1 │
                                          │  role:.. │
                                          │ }        │
                                          └──────────┘
```

| Pros | Cons |
|------|------|
| Simple to understand and implement | Server must store sessions (memory/database) |
| Easy to revoke (delete the session) | Harder to scale horizontally (sessions must be shared across servers) |
| Cookie is sent automatically by the browser | Requires CSRF protection |
| `HttpOnly` cookies can't be accessed by JavaScript (XSS-safe) | Tied to a single domain by default |

### Strategy 2: Token-Based Authentication (JWT)

The dominant approach in modern web applications, especially SPAs (Single Page Applications) and mobile apps.

**What is a JWT?**

A JSON Web Token (JWT, pronounced "jot") is a self-contained, signed string that encodes user information. The server does **not** need to store anything — all the information is in the token itself.

**A JWT has three parts, separated by dots:**

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiIsImlhdCI6MTcxMH0.SflKxwRJ...
  ──────────────────   ──────────────────────────────────────────────────────   ──────────
       HEADER                              PAYLOAD                              SIGNATURE
```

| Part | Content | Purpose |
|------|---------|---------|
| **Header** | `{ "alg": "HS256", "typ": "JWT" }` | Declares the signing algorithm |
| **Payload** | `{ "userId": 1, "role": "admin", "iat": 1710000000, "exp": 1710003600 }` | The actual data (called "claims") |
| **Signature** | `HMAC-SHA256(base64(header) + "." + base64(payload), SECRET_KEY)` | Proves the token hasn't been tampered with |

> **Critical:** The payload is **Base64-encoded, not encrypted.** Anyone with the token can read the payload. Never put passwords, credit card numbers, or sensitive data in a JWT. The signature only guarantees **integrity** (nobody changed the payload), not **confidentiality** (nobody can read it).

**How it works:**

```
1. User sends credentials
   POST /api/login
   { "email": "alice@mail.com", "password": "s3cret" }

2. Server verifies credentials against the database

3. Server creates and signs a JWT
   jwt.sign({ userId: 1, role: "admin" }, SECRET_KEY, { expiresIn: "1h" })

4. Server sends the JWT back in the response body
   { "token": "eyJhbG..." }

5. Client stores the token (localStorage, sessionStorage, or memory)

6. Client sends the token in the Authorization header on every request
   Authorization: Bearer eyJhbG...

7. Server verifies the signature and extracts the payload
```

| Pros | Cons |
|------|------|
| Stateless — server stores nothing | Cannot be revoked easily (unless you maintain a blacklist) |
| Scales horizontally with no extra infrastructure | Token size grows with payload (sent on every request) |
| Works across domains (great for APIs) | Vulnerable to XSS if stored in localStorage |
| Works with mobile apps, SPAs, microservices | Must implement refresh token logic |

### Strategy 3: OAuth 2.0 / Social Login

OAuth 2.0 is **not** an authentication protocol — it's an **authorization framework**. However, when combined with OpenID Connect (OIDC), it's used for authentication ("Login with Google/GitHub").

**The simplified flow (Authorization Code Grant):**

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐
│  Your    │     │  Auth        │     │  Identity    │
│  App     │     │  Provider    │     │  Provider    │
│(Frontend)│     │  (Your API)  │     │  (Google,    │
│          │     │              │     │   GitHub)    │
│          │     │              │     │              │
│ 1. User clicks "Login with Google"  │              │
│ ────────────────────────────────────────────────▶  │
│          │     │              │     │              │
│ 2. Redirects to Google login page   │              │
│ ◀───────────────────────────────────────────────── │
│          │     │              │     │              │
│ 3. User logs in with Google, grants permission     │
│ ────────────────────────────────────────────────▶  │
│          │     │              │     │              │
│ 4. Google redirects back with authorization code   │
│ ◀───────────────────────────────────────────────── │
│          │     │              │     │              │
│ 5. Your backend exchanges code for tokens          │
│          │     │ ────────────────────────────────▶ │
│          │     │              │     │              │
│ 6. Google returns access token + user info         │
│          │     │ ◀──────────────────────────────── │
│          │     │              │     │              │
│ 7. Your backend creates/finds user, issues own JWT │
│ ◀──────────── │              │     │              │
└──────────┘     └──────────────┘     └──────────────┘
```

| Pros | Cons |
|------|------|
| Users don't create yet another password | Complex implementation |
| Leverages the security of Google/GitHub | Depends on third-party availability |
| Higher conversion (easier sign-up) | Must handle account linking (what if user signs up with email then tries Google?) |
| MFA is handled by the provider | Privacy concerns for some users |

> **For this course:** Implement email/password auth first with JWT. OAuth is a great "advanced layer" feature but not required.

### Comparison Summary

| Factor | Sessions | JWT | OAuth 2.0 |
|--------|----------|-----|-----------|
| **State** | Server-side | Stateless (client-side) | Depends on implementation |
| **Storage** | Server memory/DB + cookie | Client (header/storage) | Provider handles identity |
| **Revocation** | Delete session (instant) | Blacklist or wait for expiry | Revoke at provider |
| **Scale** | Needs shared session store | Scales easily | Delegated to provider |
| **Best for** | Traditional web apps (SSR) | SPAs, mobile apps, APIs | Social login, SSO |
| **Complexity** | Low | Medium | High |

---

## Part 3 — Implementing JWT Authentication (Step by Step)

### The Password Problem

**Never store passwords in plain text.** This is the number one security rule. If your database is breached, plaintext passwords expose every user — and since people reuse passwords, you've potentially compromised their bank account, email, and social media.

**The solution: hashing with bcrypt.**

```
plaintext password ──▶ bcrypt.hash() ──▶ "$2b$10$abc...xyz"  (60 characters)
                                               │
                              ┌────────────────┘
                              │  Cannot be reversed
                              │  Same input always produces DIFFERENT output
                              │  (because of the random salt)
                              ▼
                        Stored in database
```

| Approach | Safe? | Why |
|----------|-------|-----|
| Plain text: `"s3cret"` | **No** | Anyone with DB access sees every password |
| MD5/SHA-256: `hash("s3cret")` | **No** | Fast to compute → vulnerable to brute force and rainbow tables |
| SHA-256 + salt: `hash("s3cret" + randomSalt)` | **Better** | Still too fast — modern GPUs can try billions per second |
| **bcrypt**: `bcrypt.hash("s3cret", 10)` | **Yes** | Intentionally slow (cost factor), includes salt automatically |

### The Complete Auth Flow

Here's the full implementation broken into steps. The examples use Node.js/Express, but the concepts apply to any stack.

#### Step 1: User Registration

```javascript
// POST /api/auth/register
import bcrypt from 'bcrypt';

async function register(req, res) {
  const { email, password, name } = req.body;

  // 1. Validate input
  if (!email || !password || !name) {
    return res.status(400).json({
      error: { code: 'VALIDATION_ERROR', message: 'All fields are required' }
    });
  }

  // 2. Check if user already exists
  const existingUser = await db.user.findByEmail(email);
  if (existingUser) {
    return res.status(409).json({
      error: { code: 'CONFLICT', message: 'Email already registered' }
    });
  }

  // 3. Hash the password (cost factor 10 = ~100ms per hash)
  const hashedPassword = await bcrypt.hash(password, 10);

  // 4. Create user in database (store the HASH, never the plaintext)
  const user = await db.user.create({
    email,
    name,
    password: hashedPassword,  // "$2b$10$abc...xyz"
    role: 'user'               // Default role
  });

  // 5. Return success (never return the password hash!)
  res.status(201).json({
    data: { id: user.id, email: user.email, name: user.name }
  });
}
```

#### Step 2: User Login

```javascript
// POST /api/auth/login
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

async function login(req, res) {
  const { email, password } = req.body;

  // 1. Find user by email
  const user = await db.user.findByEmail(email);
  if (!user) {
    // Don't reveal whether the email exists or not
    return res.status(401).json({
      error: { code: 'INVALID_CREDENTIALS', message: 'Invalid email or password' }
    });
  }

  // 2. Compare plaintext password with stored hash
  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) {
    // Same error message — don't reveal which part was wrong
    return res.status(401).json({
      error: { code: 'INVALID_CREDENTIALS', message: 'Invalid email or password' }
    });
  }

  // 3. Generate access token (short-lived)
  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  // 4. Generate refresh token (long-lived)
  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  // 5. Store refresh token in database (so it can be revoked)
  await db.refreshToken.create({
    token: refreshToken,
    userId: user.id,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  });

  // 6. Return tokens
  res.json({
    data: {
      accessToken,
      refreshToken,
      user: { id: user.id, email: user.email, name: user.name, role: user.role }
    }
  });
}
```

> **Security note:** The login endpoint returns the same error message for "wrong email" and "wrong password." This is intentional — it prevents attackers from enumerating which emails exist in your system (user enumeration attack).

#### Step 3: Auth Middleware (Protecting Routes)

```javascript
// middleware/auth.js
import jwt from 'jsonwebtoken';

function authenticate(req, res, next) {
  // 1. Extract token from Authorization header
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      error: { code: 'NO_TOKEN', message: 'Authentication required' }
    });
  }

  const token = authHeader.split(' ')[1];

  try {
    // 2. Verify token signature and expiration
    const payload = jwt.verify(token, process.env.JWT_SECRET);

    // 3. Attach user info to request object
    req.user = { userId: payload.userId, role: payload.role };

    next(); // Proceed to the route handler
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: { code: 'TOKEN_EXPIRED', message: 'Token has expired' }
      });
    }
    return res.status(401).json({
      error: { code: 'INVALID_TOKEN', message: 'Invalid token' }
    });
  }
}
```

#### Step 4: Using the Middleware

```javascript
import express from 'express';
import { authenticate } from './middleware/auth.js';

const router = express.Router();

// Public routes — no auth required
router.post('/auth/register', register);
router.post('/auth/login', login);
router.post('/auth/refresh', refreshToken);

// Protected routes — auth required
router.get('/profile', authenticate, getProfile);
router.put('/profile', authenticate, updateProfile);
router.get('/orders', authenticate, getOrders);

// Admin routes — auth + role check required
router.get('/admin/users', authenticate, authorize('admin'), getAllUsers);
router.delete('/admin/users/:id', authenticate, authorize('admin'), deleteUser);
```

---

## Part 4 — Refresh Tokens: Solving the Expiration Problem

### The Dilemma

Short-lived access tokens are more secure (if stolen, they expire quickly). But you don't want users to log in every 15 minutes.

**The solution: a two-token system.**

| Token | Lifespan | Purpose | Stored In |
|-------|----------|---------|-----------|
| **Access token** | 15 minutes | Authenticates API requests | Memory or localStorage |
| **Refresh token** | 7 days | Gets new access tokens | Database (server-side) + HttpOnly cookie or client storage |

### The Refresh Flow

```
┌─────────────┐                              ┌─────────────┐
│   Client    │                              │   Server    │
│             │                              │             │
│ 1. Access token expires                    │             │
│    (15 min)                                │             │
│             │                              │             │
│ 2. POST /auth/refresh                      │             │
│    { refreshToken: "xyz..." }              │             │
│ ───────────────────────────────────────▶   │             │
│             │                              │ 3. Verify   │
│             │                              │    refresh  │
│             │                              │    token in │
│             │                              │    database │
│             │                              │             │
│ ◀──────────────────────────────────────── │ 4. Issue    │
│ { accessToken: "new...", refreshToken:     │    new pair │
│   "alsonew..." }                           │             │
│             │                              │ 5. Revoke   │
│             │                              │    old      │
│             │                              │    refresh  │
│             │                              │    token    │
└─────────────┘                              └─────────────┘
```

```javascript
// POST /api/auth/refresh
async function refresh(req, res) {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(400).json({
      error: { code: 'MISSING_TOKEN', message: 'Refresh token is required' }
    });
  }

  // 1. Check if the refresh token exists in the database
  const storedToken = await db.refreshToken.findByToken(refreshToken);
  if (!storedToken) {
    return res.status(401).json({
      error: { code: 'INVALID_TOKEN', message: 'Invalid refresh token' }
    });
  }

  // 2. Check if it's expired
  if (storedToken.expiresAt < new Date()) {
    await db.refreshToken.delete(storedToken.id);
    return res.status(401).json({
      error: { code: 'TOKEN_EXPIRED', message: 'Refresh token has expired' }
    });
  }

  try {
    // 3. Verify the JWT signature
    const payload = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);

    // 4. Get user to include current role in new token
    const user = await db.user.findById(payload.userId);

    // 5. Generate new token pair
    const newAccessToken = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    const newRefreshToken = jwt.sign(
      { userId: user.id },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: '7d' }
    );

    // 6. Rotate: delete old refresh token, store the new one
    await db.refreshToken.delete(storedToken.id);
    await db.refreshToken.create({
      token: newRefreshToken,
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
    });

    res.json({ data: { accessToken: newAccessToken, refreshToken: newRefreshToken } });
  } catch (err) {
    return res.status(401).json({
      error: { code: 'INVALID_TOKEN', message: 'Invalid refresh token' }
    });
  }
}
```

> **Refresh token rotation:** Each time a refresh token is used, it's deleted and a new one is issued. If an attacker steals an old refresh token and tries to use it after the legitimate user already rotated it, the token won't be found in the database — detection of theft.

---

## Part 5 — Authorization: Role-Based Access Control (RBAC)

### What Is RBAC?

RBAC is a model where permissions are assigned to **roles**, and roles are assigned to **users**. Users don't get individual permissions — they inherit them from their role.

```
┌─────────┐     ┌───────────┐     ┌──────────────┐
│  Users  │────▶│   Roles   │────▶│ Permissions  │
│         │     │           │     │              │
│ Alice ──────▶ │ admin   ──────▶ │ create_user  │
│ Bob   ──────▶ │ editor  ──────▶ │ edit_post    │
│ Carol ──────▶ │ viewer  ──────▶ │ view_post    │
└─────────┘     └───────────┘     └──────────────┘
```

### Designing Roles

Start simple. For most applications, three roles are enough:

| Role | Description | Typical Permissions |
|------|------------|-------------------|
| **admin** | Full access to everything | Manage users, manage content, configure system, view reports |
| **editor** / **manager** | Can create and manage content | Create, edit, delete own content; view other users' content |
| **user** / **viewer** | Basic access | View content, manage own profile, perform own actions |

> **Rule:** Start with the fewest roles possible. You can always add more later. It's much harder to remove or merge roles once users are assigned to them.

### Implementing RBAC

#### The Authorization Middleware

```javascript
// middleware/authorize.js

function authorize(...allowedRoles) {
  return (req, res, next) => {
    // req.user is set by the authenticate middleware
    if (!req.user) {
      return res.status(401).json({
        error: { code: 'NOT_AUTHENTICATED', message: 'Authentication required' }
      });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: { code: 'FORBIDDEN', message: 'You do not have permission to perform this action' }
      });
    }

    next();
  };
}
```

#### Using It in Routes

```javascript
// Anyone authenticated can access
router.get('/profile', authenticate, getProfile);

// Only admins
router.get('/admin/users', authenticate, authorize('admin'), getAllUsers);
router.delete('/admin/users/:id', authenticate, authorize('admin'), deleteUser);

// Admins and editors
router.post('/posts', authenticate, authorize('admin', 'editor'), createPost);
router.put('/posts/:id', authenticate, authorize('admin', 'editor'), updatePost);

// All roles (but must be logged in)
router.get('/posts', authenticate, authorize('admin', 'editor', 'user'), listPosts);
```

### Resource-Level Authorization

Role-based checks are not always enough. Consider: "An editor can edit **their own** posts, but not other editors' posts." This requires checking the **resource ownership**, not just the role.

```javascript
async function updatePost(req, res) {
  const post = await db.post.findById(req.params.id);

  if (!post) {
    return res.status(404).json({
      error: { code: 'NOT_FOUND', message: 'Post not found' }
    });
  }

  // Admin can edit any post; others can only edit their own
  if (req.user.role !== 'admin' && post.authorId !== req.user.userId) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'You can only edit your own posts' }
    });
  }

  // Proceed with update...
  const updatedPost = await db.post.update(req.params.id, req.body);
  res.json({ data: updatedPost });
}
```

### Database Schema for RBAC

**Simple approach (role as a column):**

```sql
CREATE TABLE "user" (
    id             SERIAL PRIMARY KEY,
    email          VARCHAR(320) NOT NULL UNIQUE,
    name           VARCHAR(100) NOT NULL,
    password_hash  VARCHAR(60) NOT NULL,
    role           VARCHAR(20) NOT NULL DEFAULT 'user'
                   CHECK (role IN ('admin', 'editor', 'user')),
    is_active      BOOLEAN NOT NULL DEFAULT TRUE,
    created_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Flexible approach (role and permission tables — for complex systems):**

```sql
CREATE TABLE role (
    id   SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE permission (
    id   SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE  -- e.g., 'post:create', 'user:delete'
);

CREATE TABLE role_permission (
    role_id       INT NOT NULL REFERENCES role(id),
    permission_id INT NOT NULL REFERENCES permission(id),
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_role (
    user_id INT NOT NULL REFERENCES "user"(id),
    role_id INT NOT NULL REFERENCES role(id),
    PRIMARY KEY (user_id, role_id)
);
```

> **For this course:** The simple approach (role as a column) is sufficient. Use the flexible approach only if your application requires users to have multiple roles or fine-grained, dynamic permissions.

---

## Part 6 — Protecting the Frontend

### Route Protection in React

Your backend must **always** enforce authorization. Frontend route protection is a **UX convenience**, not a security measure — anyone can bypass it with browser dev tools.

```jsx
// components/ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

function ProtectedRoute({ children, requiredRole }) {
  const { user, isLoading } = useAuth();

  if (isLoading) return <div>Loading...</div>;

  // Not logged in → redirect to login
  if (!user) return <Navigate to="/login" replace />;

  // Logged in but wrong role → redirect to unauthorized page
  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}
```

```jsx
// App.jsx — Route definitions
<Routes>
  {/* Public routes */}
  <Route path="/login" element={<LoginPage />} />
  <Route path="/register" element={<RegisterPage />} />

  {/* Protected: any authenticated user */}
  <Route path="/dashboard" element={
    <ProtectedRoute>
      <DashboardPage />
    </ProtectedRoute>
  } />

  {/* Protected: admin only */}
  <Route path="/admin" element={
    <ProtectedRoute requiredRole="admin">
      <AdminPage />
    </ProtectedRoute>
  } />
</Routes>
```

### Storing Tokens on the Client

Where you store the access token has direct security implications:

| Storage | XSS Vulnerable? | CSRF Vulnerable? | Persists on Refresh? | Recommendation |
|---------|-----------------|-------------------|---------------------|---------------|
| `localStorage` | **Yes** — any JS on the page can read it | No | Yes | Acceptable for learning; avoid in production with sensitive data |
| `sessionStorage` | **Yes** — any JS on the page can read it | No | No (cleared on tab close) | Slightly better than localStorage |
| **HttpOnly cookie** | **No** — JS cannot access it | Yes (needs CSRF protection) | Yes | Most secure for web apps |
| **In-memory variable** | **No** — only your JS has access | No | No (lost on page refresh) | Most secure but worst UX |

> **Practical recommendation:** For this course, storing the access token in memory (a React state/context variable) and the refresh token in an HttpOnly cookie is a solid approach. The access token is never persisted to disk, and the refresh token is invisible to JavaScript.

---

## Part 7 — Security Best Practices

### The OWASP Authentication Checklist

| Practice | Why | How |
|----------|-----|-----|
| **Hash passwords with bcrypt** | Prevents plaintext exposure on database breach | `bcrypt.hash(password, 10)` |
| **Use HTTPS everywhere** | Prevents tokens/passwords from being intercepted in transit | TLS certificate (automatic on Vercel, Railway, Azure) |
| **Short-lived access tokens** | Limits damage window if a token is stolen | `expiresIn: '15m'` |
| **Rotate refresh tokens** | Detects token theft | Issue new refresh token on each use, invalidate old one |
| **Don't reveal user existence** | Prevents user enumeration | "Invalid email or password" (not "Email not found") |
| **Rate-limit auth endpoints** | Prevents brute-force attacks | Max 5 attempts per minute per IP |
| **Validate input server-side** | Prevents injection and malformed data | Validate email format, password length, etc. |
| **Use environment variables for secrets** | Prevents hardcoded secrets in source code | `process.env.JWT_SECRET` |
| **Set cookie flags correctly** | Prevents cookie theft and misuse | `HttpOnly`, `Secure`, `SameSite=Strict` |

### Password Requirements

Don't over-complicate password rules. Research (NIST SP 800-63B) shows that:

| Old Rule (Don't Do This) | Better Rule (Do This) | Why |
|--------------------------|----------------------|-----|
| Must include uppercase, lowercase, number, special character | Minimum 8 characters, check against breached password lists | Complex rules lead to predictable patterns ("Password1!") |
| Must change every 90 days | Change only when compromised | Forced rotation leads to weaker passwords |
| No spaces allowed | Allow spaces (passphrases are stronger) | "correct horse battery staple" is stronger than "P@ssw0rd!" |

### Common Mistakes to Avoid

| Mistake | Risk | Fix |
|---------|------|-----|
| Storing JWT secret in code | Anyone with repo access can forge tokens | Use environment variables |
| Not verifying token signature | Accepting any Base64 string as valid | Always call `jwt.verify()`, never just `jwt.decode()` |
| Sending password in URL | URLs are logged in server logs and browser history | Always send credentials in request body via POST |
| Not setting token expiration | Tokens valid forever = permanent access if stolen | Always set `expiresIn` |
| Using the same secret for access and refresh tokens | Compromise of one compromises both | Use separate secrets |
| Returning password hash in API responses | Leaks hashing algorithm and hash for offline cracking | Exclude password from all query results |
| Not handling logout | Refresh tokens remain valid after "logout" | Delete refresh token from database on logout |

---

## Part 8 — Implementing Logout

Logout is often overlooked, but it's important for security.

```javascript
// POST /api/auth/logout
async function logout(req, res) {
  const { refreshToken } = req.body;

  if (refreshToken) {
    // Delete the refresh token from the database
    await db.refreshToken.deleteByToken(refreshToken);
  }

  res.status(204).send();  // 204 No Content
}
```

**On the client side:**

```javascript
async function logout() {
  const refreshToken = getStoredRefreshToken();

  await fetch('/api/auth/logout', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ refreshToken })
  });

  // Clear tokens from client
  clearStoredTokens();

  // Redirect to login
  navigate('/login');
}
```

> **Note:** Since JWTs are stateless, the access token remains technically valid until it expires (even after logout). This is why access tokens should be short-lived (15 minutes). For immediate invalidation, you'd need a server-side token blacklist — but for most applications, a short expiry is sufficient.

---

## Part 9 — Putting It All Together: Auth Architecture

Here's the complete picture of how authentication and authorization flow through your application:

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AUTHENTICATION FLOW                          │
│                                                                     │
│  ┌────────────┐    POST /login     ┌────────────────────────────┐   │
│  │            │ ─────────────────▶ │  Auth Controller           │  │
│  │            │                    │  ├─ Validate input         │   │
│  │   Client   │                    │  ├─ Find user by email     │   │
│  │  (React)   │                    │  ├─ bcrypt.compare()       │   │
│  │            │ ◀──────────────── │  ├─ jwt.sign() access      │   │
│  │            │  { accessToken,    │  ├─ jwt.sign() refresh     │   │
│  │            │    refreshToken }  │  └─ Store refresh in DB    │   │
│  └────────────┘                    └────────────────────────────┘   │
│       │                                                             │
│       │  GET /api/orders                                            │
│       │  Authorization: Bearer <accessToken>                        │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     MIDDLEWARE CHAIN                        │    │
│  │                                                             │    │
│  │  authenticate()         authorize('admin','editor')         │    │
│  │  ┌─────────────────┐   ┌─────────────────────────┐          │    │
│  │  │ Extract Bearer  │   │ Check req.user.role     │          │    │
│  │  │ token from      │──▶│ against allowed roles  │──▶ ...   │    │
│  │  │ header          │   │                         │          │    │
│  │  │ jwt.verify()    │   │ 403 if not allowed      │          │    │
│  │  │ Set req.user    │   │                         │          │    │
│  │  │ 401 if invalid  │   │                         │          │    │
│  │  └─────────────────┘   └─────────────────────────┘          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│       │                                                             │
│       ▼                                                             │
│  ┌──────────────────┐                                               │
│  │ Route Handler    │                                               │
│  │ (Business Logic) │                                               │
│  │ Has req.user     │                                               │
│  │ with userId and  │                                               │
│  │ role available   │                                               │
│  └──────────────────┘                                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Part 10 — Implementing Auth in Your Project 

### The Exercise

Implement a complete authentication and authorization system in your personal project.

#### 1. Implement the Endpoints

| Endpoint | Method | Auth Required? | Description |
|----------|--------|---------------|-------------|
| `/api/auth/register` | POST | No | Create a new user |
| `/api/auth/login` | POST | No | Authenticate and receive tokens |
| `/api/auth/refresh` | POST | No | Exchange refresh token for new token pair |
| `/api/auth/logout` | POST | Yes | Invalidate refresh token |
| `/api/auth/me` | GET | Yes | Get current user profile |

#### 2. Protect Your Existing Routes

Apply the `authenticate` middleware to all routes that require a logged-in user. Apply `authorize` to routes that require specific roles.

#### 3. Protect Your Frontend Routes

Add a `ProtectedRoute` component and wrap routes that require authentication.

#### 4. Test the Complete Flow

Walk through these scenarios manually:

- [ ] Register a new user
- [ ] Login with correct credentials → receive tokens
- [ ] Login with wrong password → 401 error
- [ ] Access protected route with valid token → success
- [ ] Access protected route without token → 401 error
- [ ] Access admin route as regular user → 403 error
- [ ] Use refresh token to get new access token
- [ ] Logout → refresh token no longer works

---

## Part 11 — Common Auth Mistakes

| Mistake | Why It's Dangerous | Fix |
|---------|-------------------|-----|
| Storing plaintext passwords | Database breach = every password exposed | Use bcrypt with cost factor ≥ 10 |
| JWT secret = `"secret"` or `"123456"` | Trivially guessable → anyone can forge tokens | Use a long, random string (32+ characters) |
| No token expiration | A stolen token grants permanent access | Access: 15 min, refresh: 7 days |
| Same error for "missing" vs "invalid" token | Fine — this is actually a good practice | Distinguish in logs, not in responses |
| Checking auth only on the frontend | Anyone can call your API directly with curl | **Always** enforce auth in backend middleware |
| Returning `200` with `{ success: false }` | Breaks HTTP semantics; clients can't rely on status codes | Use proper HTTP status codes (401, 403) |
| `SELECT *` including password hash | Leaks hashes to the client | Explicitly select columns: `SELECT id, email, name, role` |
| Not indexing the `refresh_token.token` column | Slow lookups as tokens accumulate | `CREATE INDEX idx_refresh_token ON refresh_token(token)` |

---

## Deliverable

By the end of this week, you should have:

- [ ] **User registration** endpoint that hashes passwords with bcrypt
- [ ] **Login** endpoint that returns JWT access + refresh tokens
- [ ] **Refresh token** endpoint with token rotation
- [ ] **Logout** endpoint that invalidates the refresh token
- [ ] **Auth middleware** that protects routes and attaches user info to the request
- [ ] **Authorization middleware** with at least 2 roles (e.g., `admin` and `user`)
- [ ] **Protected frontend routes** that redirect unauthenticated users to login
- [ ] **Auth system fully functional** with at minimum: register → login → access protected resource → refresh → logout

---

## Recommended Reading

- [JWT.io](https://jwt.io/) — Decode, verify, and experiment with JWTs interactively
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — Industry-standard security checklist
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) — Best practices for session handling
- [NIST SP 800-63B — Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html) — The research behind modern password recommendations
- [Auth0 Blog: Refresh Tokens](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/) — Deep dive into refresh token mechanics
- [bcrypt explained](https://codahale.com/how-to-safely-store-a-password/) — The classic post on why bcrypt matters
- [The Bearer Token Usage RFC (RFC 6750)](https://datatracker.ietf.org/doc/html/rfc6750) — The official specification for Bearer tokens in HTTP
