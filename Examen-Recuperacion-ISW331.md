# ISW-331 — Examen de Recuperación

**Materia:** Programación Web II  
**Duración:** 30 minutos  
**Puntuación total:** 96 puntos  
**Cobertura:** Semanas 2, 3, 4–5 y 7–8  

**Instrucciones:** Responda todas las secciones. Sea conciso y preciso. Código debe ser sintácticamente correcto.

---

## Sección A — Arquitectura de Software & CSS Moderno (20 pts)

### A1. (6 pts) — Niveles de Abstracción

MVC, Clean Architecture y Microservices **no** son opciones que compiten entre sí. Operan en niveles distintos.

Complete la siguiente tabla indicando **qué es** cada concepto y en **qué nivel** opera:

| Concepto | ¿Qué es? (patrón, filosofía, topología…) | Nivel (código, aplicación, sistema) |
|----------|------------------------------------------|-------------------------------------|
| **MVC** | ________________________________________ | _______________ |
| **Clean Architecture** | ________________________________________ | _______________ |
| **Microservices** | ________________________________________ | _______________ |

---

### A2. (6 pts) — Clean Architecture: Regla de Dependencia

Observe el siguiente diagrama de Clean Architecture:

```
┌──────────────────────────────────────────┐
│  Frameworks & Drivers                    │  ← Capa A
│  ┌──────────────────────────────────┐    │
│  │  Interface Adapters              │    │  ← Capa B
│  │  ┌──────────────────────────┐    │    │
│  │  │  Use Cases               │    │    │  ← Capa C
│  │  │  ┌──────────────────┐    │    │    │
│  │  │  │  Entities        │    │    │    │  ← Capa D
│  │  │  └──────────────────┘    │    │    │
│  │  └──────────────────────────┘    │    │
│  └──────────────────────────────────┘    │
└──────────────────────────────────────────┘
```

**(a)** ¿Cuál es la regla **no negociable** de Clean Architecture respecto a las dependencias?  
**(b)** Un desarrollador escribe un Use Case que importa directamente el driver de PostgreSQL. ¿Qué principio viola y cómo debería corregirlo?

---

### A3. (4 pts) — Tailwind CSS: Ingeniería

**(a)** Explique qué es la compilación JIT (Just-In-Time) de Tailwind y por qué los archivos CSS de producción son tan pequeños (5–15 KB gzip).  
**(b)** ¿Qué problema fundamental del CSS tradicional resuelve el enfoque utility-first respecto al **scope global**?

---

### A4. (4 pts) — Enfoques de CSS

Describe con **una oración** qué hace cada enfoque de CSS y da un ejemplo de herramienta:

| Enfoque | ¿Qué hace? | Ejemplo de herramienta |
|---------|-----------|------------------------|
| **Utility-First CSS** | __________________________________ | ________________ |
| **CSS-in-JS (Runtime)** | __________________________________ | ________________ |
| **CSS Modules** | __________________________________ | ________________ |

---

## Sección B — Branching Strategies & Code Quality (20 pts)

### B1. (8 pts) — Comparación de Estrategias

Lea cada descripción y escriba a qué estrategia de branching corresponde (**Git Flow**, **GitHub Flow** o **Trunk-Based Development**):

| Descripción | Estrategia |
|-------------|------------|
| Tiene exactamente 5 tipos de branch: `main`, `develop`, `feature`, `release` y `hotfix`. Ideal para releases programados. | __________________ |
| La regla es simple: cualquier cosa en `main` es desplegable. Se trabaja en branches de corta duración y se integran mediante Pull Requests. | __________________ |
| Todos los desarrolladores hacen commit directamente a una sola rama. Requiere feature flags para ocultar funcionalidades incompletas. | __________________ |
| Las branches viven horas o días, nunca semanas. Es el flujo más usado en startups y proyectos con CI/CD. | __________________ |
| Es el flujo usado por equipos de élite como Google y Meta, y está respaldado por la investigación DORA. | __________________ |
| Es el más complejo pero protege las versiones en producción con branches `hotfix/*` para emergencias. | __________________ |
| No requiere feature flags y usa Pull Requests como primera clase. | __________________ |
| La frecuencia de deploy es múltiples veces al día y el riesgo por deploy es bajo. | __________________ |

---

### B2. (8 pts) — Continuous Integration Real

Un equipo dice que "hace CI" porque usa GitHub Actions para correr tests. Sin embargo, sus feature branches viven 2–3 semanas antes de hacer merge.

**(a)** ¿Están realmente haciendo Continuous Integration? Explique brevemente por qué sí o por qué no. (2 pts)  
**(b)** ¿Cuál es el principal riesgo técnico de tener branches que viven varias semanas sin hacer merge? (2 pts)

### B3. (4 pts) — Static Analysis

Ordene las siguientes herramientas del espectro **Cosmético → Crítico**:

`Type Checkers`  `Formatters`  `Security Scanners`  `Linters`

Cosmético ← ________ → ________ → ________ → ________ → Crítico

---

## Sección C — State Management (25 pts)

### C1. (15 pts) — Tipos de Estado

Clasifique cada ejemplo en su tipo de estado correcto (**Local/UI**, **Shared/Feature**, **Global/App**, **Server**):

| Ejemplo | Tipo |
|---------|------|
| Si un dropdown está abierto o cerrado | __________ |
| El usuario autenticado y su rol | __________ |
| Lista de productos obtenida de `/api/products` | __________ |
| Items en un carrito de compras compartido entre componentes | __________ |
| El tema (dark/light) de la aplicación | __________ |

---

### C2. (6 pts) — React: Context API vs Zustand

**(a)** ¿Cuál es la **limitación principal** del Context API de React que lo hace inadecuado para estado que cambia frecuentemente?


### C3. (4 pts) — Código: ¿Qué está mal?

Identifique **dos problemas** en este componente React que hace un fetch:

```tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, []);

  return <h1>{user?.name}</h1>;
}
```

---

## Sección D — API Communication & RESTful Design (35 pts)

### D1. (4 pts) — HTTP Status Codes

Para cada escenario, indique el **status code** correcto:

| Escenario | Status Code |
|-----------|-------------|
| Se creó un nuevo recurso exitosamente | _____ |
| El token de autenticación es inválido o expiró | _____ |
| Error interno del servidor (crash inesperado) | _____ |

---

### D2. (10 pts) — Diseño de URLs RESTful

Una aplicación gestiona **universidades**, cada universidad tiene **facultades**, y cada facultad tiene **carreras**.

**(a)** Diseñe las URLs para las siguientes operaciones (use convenciones REST):

| Operación | Método HTTP | URL |
|-----------|-------------|-----|
| Listar todas las universidades | _____ | __________________________ |
| Obtener la universidad con ID 5 | _____ | __________________________ |
| Crear una nueva facultad en la universidad 5 | _____ | __________________________ |
| Listar carreras de la facultad 3 de la universidad 5, filtradas por `status=active` y ordenadas por nombre | _____ | __________________________ |

**(b)** Un desarrollador diseña esta URL: `/api/getUniversityStudents?universityId=5`. Mencione **dos** violaciones a las convenciones REST.

---

### D3. (6 pts) — Backend: Capas y Responsabilidades

Un endpoint `POST /api/orders` debe: validar el input, verificar que el cliente existe, calcular el total con descuento si aplica, guardar en la base de datos y devolver la respuesta.

Indique **en qué capa** (Controller/Route, Service/Use Case, Repository) pertenece cada responsabilidad:

| Responsabilidad | Capa |
|----------------|------|
| Parsear el body del HTTP request y devolver el HTTP response | _____________ |
| Regla: "Si el carrito supera $50, aplicar 10% de descuento" | _____________ |
| Ejecutar `INSERT INTO orders ...` en la base de datos | _____________ |
| Verificar que todos los campos requeridos están presentes (schema) | _____________ |
| Coordinar el flujo: buscar cliente → calcular total → guardar orden | _____________ |

---

### D4. (5 pts) — CORS

**(a)** Su frontend corre en `http://localhost:5173` y su backend en `http://localhost:3000`. Explique **por qué** el browser bloquea la petición y qué header debe enviar el backend para permitirla.  
**(b)** ¿Por qué `Access-Control-Allow-Origin: *` es peligroso en producción?

---

### D5. (5 pts) — TanStack Query vs Fetch Nativo

**(a)** Mencione **dos** problemas concretos del fetch nativo que TanStack Query resuelve automáticamente. (2 pts)  
**(b)** Después de un `useMutation` exitoso (ej: crear un producto), TanStack Query no actualiza automáticamente la lista. ¿Qué concepto/mecanismo debe usar para que la lista se refresque? No necesita escribir código, explíquelo en una oración. (2 pts)

---

### D6. (5 pts) — Análisis de Código: Anti-Pattern

Observe este endpoint y mencione **tres** anti-patterns o problemas:

```csharp
app.MapPost("/api/orders", async (CreateOrderRequest req, AppDbContext db) =>
{
    // No validation
    var customer = await db.Customers
        .FirstOrDefaultAsync(c => c.Email == req.Email);

    var order = new Order
    {
        CustomerId = customer.Id,    // customer could be null!
        Total = req.Items.Sum(i => i.Qty * i.Price),
        CreatedAt = DateTime.UtcNow
    };

    db.Orders.Add(order);
    await db.SaveChangesAsync();

    return Results.Ok(order);        // wrong status code for creation
});
```

---

## Fin del Examen

**Revise sus respuestas antes de entregar.**
