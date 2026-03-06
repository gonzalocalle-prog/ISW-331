# Weeks 4 & 5 — State Management and Async API Communication

> **Block 2: Advanced Frontend**
> This week covers the two most critical topics in modern frontend development: how frameworks manage **application state**, and how they **communicate with external services asynchronously.**

---

## Week 4 — State Management: How Each Framework Does It

### What is "state" in a frontend application?

**State** is any data that, when it changes, must be reflected in the UI. It can be:

| Type | Examples |
|------|---------|
| **Local (UI)** | Is the modal open? Which tab is active? |
| **Shared (feature)** | Product list in a shopping cart |
| **Global (app)** | Authenticated user, theme, language |
| **Server (server state)** | Data fetched from an API |

Server state deserves its own category because it has unique behaviors: it can become stale, needs revalidation, and its source is external. Weeks 4 and 5 address them together because they are deeply related.

---

## React — State Management

React does not impose a state solution. The ecosystem offers layers:

### 1. Local State — `useState` and `useReducer`

```tsx
// useState: simple state
const [count, setCount] = useState(0);

// useReducer: complex state with explicit transitions
type Action = { type: 'increment' } | { type: 'decrement' } | { type: 'reset' };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case 'increment': return state + 1;
    case 'decrement': return state - 1;
    case 'reset':     return 0;
  }
}

const [count, dispatch] = useReducer(reducer, 0);
dispatch({ type: 'increment' });
```

**When to use `useReducer` over `useState`:**
- The next state depends on the previous one
- There are multiple related sub-values
- The update logic is complex

---

### 2. Shared State — Context API

Context solves the **prop drilling** problem (passing props through component levels that don't need them).

```tsx
// authContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface User { id: string; name: string; role: 'admin' | 'user' }
interface AuthContextValue {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  return (
    <AuthContext.Provider value={{
      user,
      login: setUser,
      logout: () => setUser(null),
    }}>
      {children}
    </AuthContext.Provider>
  );
}

// Safe hook: throws error if used outside the Provider
export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}
```

**Important limitation of Context:** every change in the context value re-renders **all** consumers. For state that changes frequently (e.g., mouse position) or with many consumers, Context is not the right tool.

---

### 3. Global State — Zustand

Zustand is the most popular global state solution in 2025/2026 due to its minimalist API and performance.

```tsx
// store/cartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware'; // persists to localStorage

interface CartItem { id: string; name: string; price: number; qty: number }

interface CartStore {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'qty'>) => void;
  removeItem: (id: string) => void;
  total: () => number;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],

      addItem: (item) => set((state) => {
        const existing = state.items.find(i => i.id === item.id);
        if (existing) {
          return { items: state.items.map(i =>
            i.id === item.id ? { ...i, qty: i.qty + 1 } : i
          )};
        }
        return { items: [...state.items, { ...item, qty: 1 }] };
      }),

      removeItem: (id) => set((state) => ({
        items: state.items.filter(i => i.id !== id)
      })),

      total: () => get().items.reduce((sum, i) => sum + i.price * i.qty, 0),
    }),
    { name: 'cart-storage' } // localStorage key
  )
);

// Usage in a component — selector prevents unnecessary re-renders
function CartButton() {
  const total = useCartStore(state => state.total());
  const itemCount = useCartStore(state => state.items.length);
  return <button>Cart ({itemCount}) — ${total.toFixed(2)}</button>;
}
```

---

### 4. Global State — Redux Toolkit (RTK)

Redux remains relevant in large enterprise projects. RTK eliminates the historical boilerplate.

```tsx
// store/userSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface UserState { user: User | null; loading: boolean; error: string | null }

const initialState: UserState = { user: null, loading: false, error: null };

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload; // Immer handles immutability internally
    },
    clearUser: (state) => { state.user = null; },
  },
});

export const { setUser, clearUser } = userSlice.actions;

// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
export const store = configureStore({ reducer: { user: userSlice.reducer } });
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**When to choose Redux over Zustand:**
- You need Redux DevTools with time-travel debugging
- The team already knows Redux
- Very large project with many related slices

---

### 5. Custom Hooks — The Pattern That Ties Everything Together

A Custom Hook encapsulates reusable stateful logic. It is the most powerful pattern in React.

```tsx
// hooks/useLocalStorage.ts
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// hooks/useDebounce.ts — widely used for search
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debouncedValue;
}
```

---

## Angular — State Management

Angular has its own approach: **Services** are singletons that are injected and naturally act as shared state. RxJS integrates deeply.

### 1. Services + BehaviorSubject (native pattern)

```typescript
// services/auth.service.ts
import { Injectable, signal } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

interface User { id: string; name: string; role: string }

@Injectable({ providedIn: 'root' })
export class AuthService {
  // Option A: RxJS BehaviorSubject (classic pattern)
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  currentUser$: Observable<User | null> = this.currentUserSubject.asObservable();

  // Option B: Angular Signals (Angular 17+, modern pattern)
  currentUser = signal<User | null>(null);

  login(user: User) {
    this.currentUserSubject.next(user); // RxJS
    this.currentUser.set(user);         // Signals
  }

  logout() {
    this.currentUserSubject.next(null);
    this.currentUser.set(null);
  }

  get isAuthenticated$(): Observable<boolean> {
    return this.currentUser$.pipe(map(user => user !== null));
  }
}

// Component consuming the service
@Component({
  template: `
    <!-- with async pipe (RxJS) -->
    <div *ngIf="auth.currentUser$ | async as user">
      Hello, {{ user.name }}
    </div>

    <!-- with signals (compiler optimizes automatically) -->
    @if (auth.currentUser()) {
      Hello, {{ auth.currentUser()!.name }}
    }
  `
})
export class NavComponent {
  constructor(public auth: AuthService) {}
}
```

---

### 2. Angular Signals (Angular 17+)

Signals gradually replaces BehaviorSubject for local and service state.

```typescript
// Angular Signals: reactive state without RxJS
import { signal, computed, effect } from '@angular/core';

const count = signal(0);
const doubled = computed(() => count() * 2); // automatically reactive

// effect: similar to useEffect, runs when the signal changes
effect(() => {
  console.log('count changed:', count());
});

count.set(5);          // replaces the value
count.update(n => n + 1); // updates based on the previous value
```

**Key advantage:** unlike `useState`, signals are granular. Only the component that reads the specific signal re-renders.

---

### 3. NgRx — Redux for Angular

For enterprise applications, NgRx implements the Redux pattern with RxJS.

```typescript
// Define actions
import { createAction, props } from '@ngrx/store';

export const loadProducts = createAction('[Products] Load');
export const loadProductsSuccess = createAction(
  '[Products] Load Success',
  props<{ products: Product[] }>()
);

// Reducer
import { createReducer, on } from '@ngrx/store';

export const productsReducer = createReducer(
  initialState,
  on(loadProductsSuccess, (state, { products }) => ({ ...state, products, loading: false }))
);

// Selector
export const selectProducts = (state: AppState) => state.products.items;

// Component
@Component({...})
export class ProductsComponent {
  products$ = this.store.select(selectProducts);
  constructor(private store: Store<AppState>) {
    this.store.dispatch(loadProducts());
  }
}
```

---

## Astro — State Management

Astro is an "islands architecture" framework: most HTML is generated on the server, and only the interactive parts (islands) use JavaScript on the client. This fundamentally changes how state is managed.

### The state problem in Astro

In React/Angular the entire app runs on the client. In Astro, components are separate islands that **do not share a component tree**. You cannot pass state between islands with normal props.

```
Astro Page (Server)
├── Header.astro (server, no JS)
├── <CartButton client:load />  ← React island
├── <ProductGrid client:visible /> ← separate React island
└── Footer.astro (server, no JS)

CartButton and ProductGrid do NOT share a React context.
They need to communicate differently.
```

### 1. Nanostores (Astro's recommended solution)

```typescript
// stores/cart.ts — works with React, Vue, Svelte, and Vanilla JS
import { atom, map, computed } from 'nanostores';

// atom: single value
export const isCartOpen = atom(false);

// map: reactive object
export const cartItems = map<Record<string, CartItem>>({});

// computed: reactive derived value
export const cartTotal = computed(cartItems, (items) =>
  Object.values(items).reduce((sum, item) => sum + item.price * item.qty, 0)
);

// Actions (functions that modify the store)
export function addToCart(product: Product) {
  const current = cartItems.get()[product.id];
  if (current) {
    cartItems.setKey(product.id, { ...current, qty: current.qty + 1 });
  } else {
    cartItems.setKey(product.id, { ...product, qty: 1 });
  }
}
```

```tsx
// CartButton.tsx (React island in Astro)
import { useStore } from '@nanostores/react';
import { cartItems, cartTotal, isCartOpen } from '../stores/cart';

export function CartButton() {
  const items = useStore(cartItems);
  const total = useStore(cartTotal);
  const count = Object.keys(items).length;

  return (
    <button onClick={() => isCartOpen.set(true)}>
      Cart ({count}) — ${total.toFixed(2)}
    </button>
  );
}
```

```js
---
// page.astro — both islands share the same store
import { CartButton } from '../components/CartButton';
import { ProductGrid } from '../components/ProductGrid';
---
<CartButton client:load />
<ProductGrid client:visible />
<!-- Both read/write the same nanostore → they stay in sync -->
```

### 2. Server-side state with Astro

```js
---
// products.astro — server data, no JS on the client
import { db } from '../lib/db';
const products = await db.product.findMany();
---
<!-- Static HTML, zero KB of JavaScript to list products -->
{products.map(p => (
  <article>
    <h2>{p.name}</h2>
    <p>${p.price}</p>
    <!-- Only the interactive part is an island -->
    <AddToCartButton client:load productId={p.id} />
  </article>
))}
```

---

## State Management Comparison

| Aspect | React | Angular | Astro |
|--------|-------|---------|-------|
| **Local state** | `useState`, `useReducer` | Signals, class properties | Props in .astro (server) |
| **Simple shared state** | Context API | Injected Services | Nanostores |
| **Complex global state** | Zustand, Redux Toolkit | NgRx, Akita | Nanostores |
| **Server state** | TanStack Query | HttpClient + RxJS | fetch in frontmatter (server) |
| **Reactivity** | Re-render by tree | Signals / Zone.js | Per island |
| **DevTools** | React DevTools, Redux DevTools | Angular DevTools | Browser DevTools |

---

---

## Week 5 — Async Communication with APIs

### The lifecycle of an HTTP request in the frontend

Every API request goes through the same lifecycle that the UI must always represent:

```
idle → loading → success
                → error
```

Ignoring any of these states is a UX bug. The goal of the tools in this week is to handle this cycle elegantly.

---

## React — API Communication

### 1. Native Fetch (baseline)

```tsx
// The problem with native fetch: boilerplate repetition
async function getProducts(): Promise<Product[]> {
  const res = await fetch('/api/products');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// In a component → you need to handle loading/error manually
function ProductList() {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    getProducts()
      .then(setProducts)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Skeleton />;
  if (error) return <ErrorMessage message={error} />;
  return <ul>{products.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

This pattern works but has problems in real projects:
- **Race conditions**: if the component unmounts before the response arrives, calling setState on a dead component = warning
- **No caching**: every component mount triggers a new request
- **No revalidation**: data can become stale
- **No deduplication**: if two components request the same endpoint simultaneously, two requests are made

---

### 2. Axios — HTTP Client with Better Ergonomics

```tsx
// lib/api.ts — configured instance
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10_000,
  headers: { 'Content-Type': 'application/json' },
});

// Request interceptor: automatically adds token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response interceptor: centralized error handling
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Expired token: redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

// services/products.ts
export const productsService = {
  getAll: () => api.get<Product[]>('/products').then(r => r.data),
  getById: (id: string) => api.get<Product>(`/products/${id}`).then(r => r.data),
  create: (data: CreateProductDto) => api.post<Product>('/products', data).then(r => r.data),
  update: (id: string, data: UpdateProductDto) => api.put<Product>(`/products/${id}`, data).then(r => r.data),
  delete: (id: string) => api.delete(`/products/${id}`),
};
```

**Advantages of Axios over fetch:**
- Better HTTP error handling (fetch only throws on network errors, not 4xx/5xx)
- Simpler request cancellation
- Interceptors for cross-cutting logic
- Automatic JSON transformation
- Compatible with Node.js without polyfills

---

### 3. TanStack Query (React Query) — The Standard for Server State

TanStack Query automatically handles caching, revalidation, deduplication, and loading states.

```tsx
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // data stays "fresh" for 5 minutes
      retry: 2,                  // retry 2 times on error
    },
  },
});

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

```tsx
// hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { productsService } from '../services/products';

// Query keys: identify and group requests in the cache
export const productKeys = {
  all: ['products'] as const,
  byId: (id: string) => ['products', id] as const,
};

// Read data
export function useProducts() {
  return useQuery({
    queryKey: productKeys.all,
    queryFn: productsService.getAll,
  });
}

export function useProduct(id: string) {
  return useQuery({
    queryKey: productKeys.byId(id),
    queryFn: () => productsService.getById(id),
    enabled: !!id, // only runs if there is an id
  });
}

// Mutation: create, update, delete
export function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: productsService.create,
    onSuccess: () => {
      // Invalidate cache → React Query refetches automatically
      queryClient.invalidateQueries({ queryKey: productKeys.all });
    },
    onError: (error) => {
      console.error('Failed to create product:', error);
    },
  });
}
```

```tsx
// ProductList.tsx — clean code with TanStack Query
function ProductList() {
  const { data: products, isLoading, isError, error } = useProducts();
  const createProduct = useCreateProduct();

  if (isLoading) return <Skeleton count={6} />;
  if (isError) return <ErrorBoundary message={error.message} />;

  return (
    <div>
      <button
        onClick={() => createProduct.mutate({ name: 'New Product', price: 9.99 })}
        disabled={createProduct.isPending}
      >
        {createProduct.isPending ? 'Creating...' : 'Add Product'}
      </button>
      <ul>
        {products.map(p => <ProductCard key={p.id} product={p} />)}
      </ul>
    </div>
  );
}
```

**What TanStack Query does for you automatically:**
- **Caching**: same queryKey = a single request even if 10 components ask for it
- **Background refetch**: updates data when the user returns to the tab
- **Stale-while-revalidate**: shows old data while updating in the background
- **Deduplication**: multiple components mounting at the same time → 1 single request
- **Devtools**: see the state of the entire cache in real time

---

### 4. SWR — Lightweight Alternative from Vercel

```tsx
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (isLoading) return <Spinner />;
  if (error) return <Error />;
  return <div>Hello {data.name}</div>;
}
```

SWR is simpler than TanStack Query. Choose TanStack Query if you need: mutations with invalidation, dependent queries, prefetching, or advanced pagination.

---

## Angular — API Communication

### 1. HttpClient + RxJS (native pattern)

Angular has its own built-in HTTP client. The fundamental difference from React: everything returns **Observables**, not Promises.

```typescript
// services/products.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, retry, map } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class ProductsService {
  private http = inject(HttpClient);
  private baseUrl = '/api/products';

  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>(this.baseUrl).pipe(
      retry(2),
      catchError(this.handleError)
    );
  }

  getById(id: string): Observable<Product> {
    return this.http.get<Product>(`${this.baseUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }

  create(product: CreateProductDto): Observable<Product> {
    return this.http.post<Product>(this.baseUrl, product);
  }

  update(id: string, product: UpdateProductDto): Observable<Product> {
    return this.http.put<Product>(`${this.baseUrl}/${id}`, product);
  }

  delete(id: string): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    const message = error.status === 0
      ? 'Network error'
      : `Server error: ${error.status}`;
    return throwError(() => new Error(message));
  }
}
```

```typescript
// app.config.ts — configure HttpClient (Angular 17+ standalone)
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './interceptors/auth.interceptor';

export const appConfig = {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor]))
  ]
};

// interceptors/auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token');
  if (token) {
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
    return next(authReq);
  }
  return next(req);
};
```

### 2. Key RxJS Operators for APIs

```typescript
// In a modern Angular component (with signals)
import { Component, inject, signal } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { switchMap, debounceTime, distinctUntilChanged } from 'rxjs/operators';

@Component({...})
export class SearchComponent {
  private productsService = inject(ProductsService);

  searchTerm = signal('');
  results = signal<Product[]>([]);
  loading = signal(false);

  constructor() {
    // Search with debounce — very common pattern
    toObservable(this.searchTerm).pipe(
      debounceTime(300),          // wait 300ms after last keystroke
      distinctUntilChanged(),     // ignore if value didn't change
      switchMap(term => {         // cancel previous request if new one comes in
        this.loading.set(true);
        return this.productsService.search(term);
      }),
      takeUntilDestroyed()        // cancel when component is destroyed
    ).subscribe({
      next: (results) => {
        this.results.set(results);
        this.loading.set(false);
      },
      error: (err) => {
        console.error(err);
        this.loading.set(false);
      }
    });
  }
}
```

**Essential RxJS operators for HTTP:**
| Operator | Use |
|----------|-----|
| `switchMap` | Real-time search (cancels previous) |
| `mergeMap` | Parallel requests without cancellation |
| `concatMap` | Sequential requests (one after another) |
| `debounceTime` | Reduce requests while user types |
| `catchError` | Handle errors without breaking the Observable |
| `retry` | Automatically retry on failures |
| `finalize` | Always clean up loading state (success or error) |

### 3. Angular Resource API (Angular 19+ — experimental)

Angular 19 introduces `resource()` to natively integrate HTTP requests with signals, similar to TanStack Query:

```typescript
import { resource, signal } from '@angular/core';
import { inject } from '@angular/core';

@Component({...})
export class ProductDetailComponent {
  productId = signal<string>('');

  productResource = resource({
    request: this.productId,
    loader: ({ request: id }) =>
      fetch(`/api/products/${id}`).then(r => r.json())
  });

  // productResource.value() — data
  // productResource.isLoading() — loading state
  // productResource.error() — error state
}
```

---

## Astro — API Communication

Astro has two execution contexts: **server** (build time / SSR) and **client** (islands).

### 1. Server Fetch (the primary approach)

```astro
---
// products.astro
// This code runs on the SERVER — never reaches the client
const res = await fetch('https://api.example.com/products', {
  headers: { Authorization: `Bearer ${import.meta.env.API_SECRET}` }
});

if (!res.ok) {
  return Astro.redirect('/error');
}

const products: Product[] = await res.json();
---

<!-- Static HTML, 0 KB of JS for the user -->
<ul>
  {products.map(p => <li>{p.name} - ${p.price}</li>)}
</ul>
```

**Advantage**: the API key never reaches the client. The user only receives HTML.

### 2. API Routes — Backend in the Same Project

```typescript
// src/pages/api/products.ts
import type { APIRoute } from 'astro';
import { db } from '../../lib/db';

export const GET: APIRoute = async ({ url }) => {
  const search = url.searchParams.get('search') ?? '';
  const products = await db.product.findMany({
    where: { name: { contains: search } }
  });

  return new Response(JSON.stringify(products), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  });
};

export const POST: APIRoute = async ({ request }) => {
  const body = await request.json();
  const product = await db.product.create({ data: body });
  return new Response(JSON.stringify(product), { status: 201 });
};
```

### 3. Fetch in Islands (client-side)

When a React island in Astro needs to make client-side requests, it uses the same React tools (TanStack Query, SWR, Axios):

```tsx
// components/LiveSearch.tsx — React island with TanStack Query
import { useQuery } from '@tanstack/react-query';

export function LiveSearch() {
  const [term, setTerm] = useState('');
  const debouncedTerm = useDebounce(term, 300);

  const { data, isLoading } = useQuery({
    queryKey: ['search', debouncedTerm],
    queryFn: () =>
      fetch(`/api/products?search=${debouncedTerm}`).then(r => r.json()),
    enabled: debouncedTerm.length > 2,
  });

  return (
    <div>
      <input onChange={e => setTerm(e.target.value)} />
      {isLoading && <Spinner />}
      {data?.map(p => <SearchResult key={p.id} product={p} />)}
    </div>
  );
}
```

```astro
---
// page.astro
import { LiveSearch } from '../components/LiveSearch';
---
<!-- QueryClientProvider required for TanStack Query in Astro -->
<QueryProvider client:load>
  <LiveSearch client:load />
</QueryProvider>
```

---

## API Communication Comparison

| Aspect | React | Angular | Astro |
|--------|-------|---------|-------|
| **Native fetch** | `fetch` (manual) | `HttpClient` (Observable) | `fetch` in frontmatter (server) |
| **HTTP Client** | Axios (popular) | HttpClient (built-in) | Axios in islands |
| **Caching / Server State** | TanStack Query / SWR | `resource()` (experimental) / TanStack Query | Server fetch (automatic) |
| **Interceptors** | Axios interceptors | `HttpInterceptorFn` | Astro middleware |
| **Cancellation** | AbortController | RxJS `takeUntil` / `switchMap` | AbortController |
| **Real-time** | WebSocket / SSE hooks | RxJS WebSocket | SSE in API routes |
| **Auth headers** | Axios interceptor | HTTP Interceptor | Headers in server fetch |

---

## Advanced Patterns

### Optimistic Updates (React + TanStack Query)

Updates the UI **before** the server responds to give a feel of instantaneity:

```tsx
export function useDeleteProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: productsService.delete,

    onMutate: async (id) => {
      // Cancel ongoing refetches
      await queryClient.cancelQueries({ queryKey: productKeys.all });

      // Snapshot of previous state (for rollback)
      const previousProducts = queryClient.getQueryData<Product[]>(productKeys.all);

      // Optimistically update the UI
      queryClient.setQueryData<Product[]>(productKeys.all, old =>
        old?.filter(p => p.id !== id) ?? []
      );

      return { previousProducts };
    },

    onError: (err, id, context) => {
      // Revert if the server fails
      queryClient.setQueryData(productKeys.all, context?.previousProducts);
    },

    onSettled: () => {
      // Always revalidate when done
      queryClient.invalidateQueries({ queryKey: productKeys.all });
    },
  });
}
```

### Pagination and Infinite Scroll (React + TanStack Query)

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';

function InfiniteProductList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['products', 'infinite'],
    queryFn: ({ pageParam = 1 }) =>
      productsService.getPaginated(pageParam, 20),
    getNextPageParam: (lastPage) => lastPage.nextPage ?? undefined,
    initialPageParam: 1,
  });

  const products = data?.pages.flatMap(page => page.items) ?? [];

  return (
    <>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage ? 'Loading...' : 'Load more'}
      </button>
    </>
  );
}
```

### Error Boundaries (React)

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }: FallbackProps) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// Wraps sections that can fail
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <ProductList />
</ErrorBoundary>
```

### Loading Skeletons

```tsx
// components/ProductSkeleton.tsx
function ProductSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-48 bg-gray-200 rounded-lg mb-4" />
      <div className="h-4 bg-gray-200 rounded w-3/4 mb-2" />
      <div className="h-4 bg-gray-200 rounded w-1/2" />
    </div>
  );
}

// Usage
if (isLoading) return (
  <div className="grid grid-cols-3 gap-4">
    {Array.from({ length: 6 }).map((_, i) => <ProductSkeleton key={i} />)}
  </div>
);
```

---

## Deliverables

### Week 4

- [ ] **Auth Context (React)** or **AuthService (Angular)** or **AuthStore (Astro/Nanostores)** working
- [ ] Protected routes implemented (redirect to login if no session)
- [ ] At least one reusable Custom Hook / Service / Store extracted
- [ ] Loading/error state handled correctly

### Week 5

- [ ] Frontend connected to a real or mock API (JSON Server / MSW)
- [ ] TanStack Query / HttpClient / server fetch configured
- [ ] Loading skeletons implemented on at least one list
- [ ] Error boundary or error handling implemented
- [ ] At least one mutation (create or delete) with cache invalidation

---

## Resources

### State Management
- [Zustand docs](https://zustand.docs.pmnd.rs/)
- [Redux Toolkit docs](https://redux-toolkit.js.org/)
- [Angular Signals guide](https://angular.dev/guide/signals)
- [Nanostores](https://github.com/nanostores/nanostores)

### API Communication
- [TanStack Query docs](https://tanstack.com/query/latest)
- [Angular HttpClient guide](https://angular.dev/guide/http)
- [Astro API Routes](https://docs.astro.build/en/guides/endpoints/)
- [SWR docs](https://swr.vercel.app/)
- [Axios docs](https://axios-http.com/)
