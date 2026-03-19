# 12 Web API Anti-Patterns (.NET)

## The Problems

*Based on content by Anton Martyniuk — antondevtips.com*

---

### 01 — Fat Controllers → Thin Controllers + Services

**Problem:** Business logic, validation, and data access all live inside controller actions. This makes code impossible to test and painful to maintain.

```csharp
// ❌ BAD — Everything crammed into the endpoint
app.MapPost("/api/orders", async (CreateOrderRequest request, AppDbContext db,
    IEmailService emailService) =>
{
    // ⚠️ Validation logic in the controller
    if (string.IsNullOrEmpty(request.CustomerEmail))
        return Results.BadRequest("Email is required");

    // ⚠️ Data access in the controller
    var customer = await db.Customers
        .FirstOrDefaultAsync(c => c.Email == request.CustomerEmail);

    if (customer is null)
        return Results.NotFound("Customer not found");

    // ⚠️ Business logic in the controller
    var order = new Order
    {
        CustomerId = customer.Id,
        Total = request.Items.Sum(i => i.Quantity * i.Price),
        CreatedAt = DateTime.UtcNow
    };

    db.Orders.Add(order);
    await db.SaveChangesAsync();
    await emailService.SendConfirmationAsync(customer.Email, order.Id);

    return Results.Created($"/api/orders/{order.Id}", order);
});
```

**Solution:** Move logic into dedicated services or MediatR handlers. Keep controllers thin — they only accept a request, call a service, and return a response.

```csharp
// ✅ GOOD — Controller delegates to a service
app.MapPost("/api/orders", async (
    CreateOrderRequest request,
    IOrderService orderService) =>
{
    var result = await orderService.CreateOrderAsync(request);

    return result.Match(
        order => Results.Created($"/api/orders/{order.Id}", order),
        notFound => Results.NotFound(notFound.Message),
        validation => Results.BadRequest(validation.Errors));
});
```
---

### 02 — No Input Validation → FluentValidation

**Problem:** User input goes straight into your business logic without any checks. Bad data causes crashes, corrupted records, and security vulnerabilities.

```csharp
// ❌ BAD — No validation at all
app.MapPost("/api/products", async (
    CreateProductRequest request,
    AppDbContext db) =>
{
    // ⚠️ request.Name could be null or empty
    // ⚠️ request.Price could be negative
    // ⚠️ request.Sku could be a duplicate
    var product = new Product
    {
        Name = request.Name,
        Price = request.Price,
        Sku = request.Sku
    };

    db.Products.Add(product);
    await db.SaveChangesAsync();

    return Results.Created($"/api/products/{product.Id}", product);
});
```

**Solution:** Validate every incoming request using FluentValidation or DataAnnotations. Return clear 400 Bad Request responses with details about what went wrong.

```csharp
// ✅ GOOD — Dedicated validator class
public class CreateProductValidator
    : AbstractValidator<CreateProductRequest>
{
    public CreateProductValidator(AppDbContext db)
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .MaximumLength(200);

        RuleFor(x => x.Price)
            .GreaterThan(0);

        RuleFor(x => x.Sku)
            .NotEmpty()
            .MaximumLength(50)
            .MustAsync(async (sku, ct) =>
                !await db.Products.AnyAsync(
                    p => p.Sku == sku, ct))
            .WithMessage("SKU already exists");
    }
}
```

---

### 03 — Returning Raw Exceptions → Global Exception Handler

**Problem:** Stack traces and internal error details leak directly to API clients. This exposes sensitive information about your codebase to attackers.

```csharp
// ❌ BAD — Leaking exception internals to the client
app.MapGet("/api/orders/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    try
    {
        var order = await db.Orders
            .FirstAsync(o => o.Id == id);

        return Results.Ok(order);
    }
    catch (Exception ex)
    {
        // ⚠️ Exposing Message, StackTrace, and Source
        return Results.Json(new
        {
            Error = ex.Message,
            StackTrace = ex.StackTrace,   // 🚨 SECURITY RISK
            Source = ex.Source             // 🚨 SECURITY RISK
        }, statusCode: 500);
    }
});
```

**Solution:** Use global exception handling middleware to catch all unhandled exceptions. Return structured error responses using ProblemDetails format.

```csharp
// ✅ GOOD — Global handler with ProblemDetails
public class GlobalExceptionHandler(
    ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext context, Exception exception,
        CancellationToken cancellationToken)
    {
        logger.LogError(exception,
            "Unhandled exception for {Path}",
            context.Request.Path);

        var problemDetails = new ProblemDetails
        {
            Title = "Internal Server Error",
            Detail = "An unexpected error occurred",
            Status = StatusCodes.Status500InternalServerError,
            Instance = context.Request.Path
        };

        context.Response.StatusCode = 500;
        await context.Response
            .WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}
```

---

### 04 — Blocking Async → async/await All the Way


**Problem:** Calling `.Result` or `.Wait()` on async methods blocks threads and can cause deadlocks. Under load, your thread pool gets exhausted and your API stops responding.

```csharp
// ❌ BAD — Blocking async calls
public class PaymentController : ControllerBase
{
    private readonly IPaymentService _paymentService;

    [HttpPost("process")]
    public IActionResult ProcessPayment(PaymentRequest request)
    {
        // ⚠️ .Result blocks the thread pool thread
        var user = _paymentService.GetUserAsync(request.UserId)
            .Result;

        var validation = _paymentService
            .ValidatePaymentMethodAsync(request.CardToken)
            .Result;                    // ⚠️ Another blocking call

        if (!validation.IsValid)
            return BadRequest("Invalid payment method");

        var charge = _paymentService.ChargeAsync(request.Amount, request.CardToken)
            .GetAwaiter().GetResult();  // ⚠️ Still blocking!

        return Ok(new { TransactionId = charge.Id, Status = charge.Status });
    }
}
```

**Solution:** Use async/await all the way down from controllers to database calls. Never mix sync and async code in the same call chain.

```csharp
// ✅ GOOD — Fully async with CancellationToken
public class PaymentController : ControllerBase
{
    private readonly IPaymentService _paymentService;

    [HttpPost("process")]
    public async Task<IActionResult> ProcessPayment(PaymentRequest request,
        CancellationToken cancellationToken)
    {
        var user = await _paymentService.GetUserAsync(request.UserId,
            cancellationToken);

        var validation = await _paymentService.ValidatePaymentMethodAsync(
            request.CardToken, cancellationToken);

        if (!validation.IsValid)
            return BadRequest("Invalid payment method");

        var charge = await _paymentService.ChargeAsync(request.Amount,
            request.CardToken, cancellationToken);

        return Ok(new { TransactionId = charge.Id, Status = charge.Status });
    }
}
```
---

### 05 — Ignoring CancellationTokens → Pass Them Everywhere

**Problem:** When a client cancels a request, your API keeps processing the work. Database queries and HTTP calls continue wasting server resources for nothing.

```csharp
// ❌ BAD — No CancellationToken passed anywhere
public class OrderService
{
    private readonly HttpClient _httpClient;
    private readonly IOrderRepository _repository;

    public async Task<OrderResult> ProcessOrderAsync(int orderId)
    {
        // ⚠️ No cancellationToken parameter
        var order = await _repository.GetOrderAsync(orderId);
        var inventory = await _httpClient.GetFromJsonAsync<Inventory>(
            $"https://api.inventory.com/check/{order.ProductId}");

        if (inventory.Available)
        {
            await _repository.UpdateOrderStatusAsync(orderId, "Confirmed");
            await _httpClient.PostAsJsonAsync("https://api.payment.com/charge", order);
        }

        return new OrderResult { Success = inventory.Available };
    }
}
```

**Solution:** Accept CancellationToken in every async endpoint and pass it to EF Core queries and HttpClient calls. This frees up resources for requests that actually matter.

```csharp
// ✅ GOOD — CancellationToken threaded through every call
public class OrderService
{
    private readonly HttpClient _httpClient;
    private readonly IOrderRepository _repository;

    public async Task<OrderResult> ProcessOrderAsync(int orderId,
        CancellationToken cancellationToken)
    {
        var order = await _repository.GetOrderAsync(orderId, cancellationToken);
        var inventory = await _httpClient.GetFromJsonAsync<Inventory>(
            $"https://api.inventory.com/check/{order.ProductId}", cancellationToken);

        if (inventory.Available)
        {
            await _repository.UpdateOrderStatusAsync(orderId, "Confirmed",
                cancellationToken);

            await _httpClient.PostAsJsonAsync("https://api.payment.com/charge", order,
                cancellationToken);
        }

        return new OrderResult { Success = inventory.Available };
    }
}
```

---

### 06 — No Pagination → Skip/Take or Cursor-Based


**Problem:** Returning entire database tables in a single API response. As data grows, response times explode and memory usage spikes.

```csharp
// ❌ BAD — Loading everything into memory
app.MapGet("/api/products", async (AppDbContext db) =>
{
    // ⚠️ No limit — could return millions of rows
    var products = await db.Products
        .Include(p => p.Category)
        .Include(p => p.Reviews)
        .ToListAsync();

    return Results.Ok(products);
});
```

**Solution:** Add pagination to every collection endpoint using skip/take or cursor-based approach. Support filtering and sorting so clients request only the data they need.

```csharp
// ✅ GOOD — Paginated, filtered, and projected
app.MapGet("/api/products", async (
    AppDbContext db,
    [AsParameters] PaginationQuery query) =>
{
    var products = await db.Products
        .Where(p => query.Category == null
            || p.Category.Name == query.Category)
        .OrderBy(p => p.Name)
        .Skip((query.Page - 1) * query.PageSize)
        .Take(query.PageSize)
        .Select(p => new ProductResponse(
            p.Id, p.Name, p.Price))
        .ToListAsync();

    return Results.Ok(new PagedResponse<ProductResponse>(
        products, query.Page, query.PageSize));
});
```

---

### 07 — Wrong HTTP Status Codes → Semantic Status Codes

**Problem:** Returning 200 OK for everything, including errors and failures. Frontend developers can't distinguish success from failure without parsing the body.

```csharp
// ❌ BAD — 200 OK for errors
app.MapGet("/api/customers/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    var customer = await db.Customers.FindAsync(id);

    if (customer is null)
        return Results.Ok(new { Error = "Customer not found" }); // ⚠️ 200 for not-found

    return Results.Ok(customer);
});

app.MapPost("/api/customers", async (
    CreateCustomerRequest request, AppDbContext db) =>
{
    if (string.IsNullOrEmpty(request.Email))
        return Results.Ok(new { Error = "Email is required" }); // ⚠️ 200 for bad input

    return Results.Ok(new { Success = true });
});
```

**Solution:** Use proper codes: 400 for bad input, 404 for missing resources, 409 for conflicts, 500 for server failures. This makes your API predictable and easy to integrate with.

```csharp
// ✅ GOOD — Correct status codes with ProblemDetails
app.MapGet("/api/customers/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    var customer = await db.Customers.FindAsync(id);

    return customer is null
        ? Results.NotFound(new ProblemDetails
        {
            Title = "Customer Not Found",
            Detail = $"No customer found with ID {id}",
            Status = StatusCodes.Status404NotFound
        })
        : Results.Ok(new CustomerResponse(
            customer.Id, customer.Name, customer.Email));
});
```

---

### 08 — Over-fetching Data → Projections with Select()

**Problem:** Querying all columns and loading all related entities when the client only needs a few fields. This wastes database resources and slows down response times.

```csharp
// ❌ BAD — Eager-loading a massive object graph
app.MapGet("/api/orders/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    var order = await db.Orders
        .Include(o => o.Customer)              // ⚠️ Loading full Customer
            .ThenInclude(c => c.Address)
            .ThenInclude(a => a.Country)
        .Include(o => o.Items)                 // ⚠️ Loading all Items
            .ThenInclude(i => i.Product)
                .ThenInclude(p => p.Category)
        .Include(o => o.Items)
            .ThenInclude(i => i.Product)
                .ThenInclude(p => p.Reviews)   // ⚠️ Even Reviews!
        .Include(o => o.Payments)
        .FirstOrDefaultAsync(o => o.Id == id);

    return Results.Ok(order);
});
```

**Solution:** Use projections with `Select()` to return only the fields you need. Avoid eager loading related entities unless they are required for the response.

```csharp
// ✅ GOOD — Only fetching what the client needs
app.MapGet("/api/orders/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    var order = await db.Orders
        .Where(o => o.Id == id)
        .Select(o => new OrderResponse(
            o.Id, o.Customer.Name, o.Total,
            o.Items.Select(i => new OrderItemResponse(
                i.Product.Name, i.Quantity, i.Price
            )).ToList()))
        .FirstOrDefaultAsync();

    return order is null
        ? Results.NotFound()
        : Results.Ok(order);
});
```
---

### 09 — Returning EF Entities → Dedicated DTOs

**Problem:** Exposing database models directly as API responses leaks your internal structure to clients. It also causes circular reference issues and exposes sensitive fields.

```csharp
// ❌ BAD — Entity with sensitive fields returned directly
public class Customer
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string PasswordHash { get; set; }    // 🚨 EXPOSED
    public string InternalNotes { get; set; }   // 🚨 EXPOSED
    public List<Order> Orders { get; set; }     // ⚠️ Circular ref
    public List<Address> Addresses { get; set; }
}

app.MapGet("/api/customers/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    var customer = await db.Customers
        .Include(c => c.Orders)
        .Include(c => c.Addresses)
        .FirstOrDefaultAsync(c => c.Id == id);

    return Results.Ok(customer); // ⚠️ Raw entity sent to client
});
```

**Solution:** Map EF entities to dedicated DTOs or response models before returning them. Use mapping libraries like Mapster or Mapperly to keep the code clean.

```csharp
// ✅ GOOD — Dedicated response record, projected at the query level
public record CustomerResponse(
    Guid Id, string Name, string Email, int TotalOrders);

app.MapGet("/api/customers/{id:guid}", async (
    Guid id, AppDbContext db) =>
{
    var customer = await db.Customers
        .Where(c => c.Id == id)
        .Select(c => new CustomerResponse(
            c.Id, c.Name, c.Email, c.Orders.Count))
        .FirstOrDefaultAsync();

    return customer is null
        ? Results.NotFound()
        : Results.Ok(customer);
});
```
---

### 10 — No Rate Limiting → Built-in Middleware

**Problem:** Your API has no throttling, leaving it wide open to abuse. A single client can flood your server with requests and bring it down.

```csharp
// ❌ BAD — No rate limiting at all
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapPost("/api/auth/login", async (
    LoginRequest request,
    IAuthService authService) =>
{
    // ⚠️ Nothing prevents brute-force attacks
    var result = await authService.LoginAsync(
        request.Email, request.Password);

    return result.IsSuccess
        ? Results.Ok(result.Token)
        : Results.Unauthorized();
});

app.Run();
```

**Solution:** Use the built-in Rate Limiting middleware in ASP.NET Core. Configure fixed window, sliding window, or token bucket policies based on your needs.

```csharp
// ✅ GOOD — Rate limiting configured and applied
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("auth", opt =>
    {
        opt.PermitLimit = 5;
        opt.Window = TimeSpan.FromMinutes(1);
    });

    options.RejectionStatusCode =
        StatusCodes.Status429TooManyRequests;
});

var app = builder.Build();
app.UseRateLimiter();

app.MapPost("/api/auth/login", handler)
    .RequireRateLimiting("auth");
```
---

### 11 — No Observability → OpenTelemetry + Serilog

**Problem:** You have zero visibility into what is happening inside your API. When something breaks in production, you are guessing instead of diagnosing.

```csharp
// ❌ BAD — Silent failures, no logging
app.MapPost("/api/orders", async (
    CreateOrderRequest request,
    IOrderService orderService) =>
{
    try
    {
        var order = await orderService
            .CreateOrderAsync(request);

        return Results.Created(
            $"/api/orders/{order.Id}", order);
    }
    catch (Exception)
    {
        // ⚠️ Exception is swallowed — no logging, no trace
        return Results.StatusCode(500);
    }
});
```

**Solution:** Add structured logging with Serilog, distributed tracing, and metrics using OpenTelemetry. Use dashboards to monitor performance and set up alerts for anomalies.

```csharp
// ✅ GOOD — Full observability stack
builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r
        .AddService("OrdersApi"))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddEntityFrameworkCoreInstrumentation()
        .AddOtlpExporter())
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddOtlpExporter());

builder.Services.AddSerilog(config => config
    .Enrich.FromLogContext()
    .Enrich.WithProperty("Service", "OrdersApi")
    .WriteTo.OpenTelemetry());
```

---

### 12 — No Idempotency → Idempotency Keys

**Problem:** When a client retries a request, your API creates duplicate records or triggers side effects twice. This leads to double charges, duplicate orders, and corrupted data.

```csharp
// ❌ BAD — Retries cause duplicate charges
app.MapPost("/api/payments", async (
    ProcessPaymentRequest request,
    AppDbContext db,
    IPaymentGateway gateway) =>
{
    var payment = new Payment
    {
        OrderId = request.OrderId,
        Amount = request.Amount,
        CreatedAt = DateTime.UtcNow
    };

    // ⚠️ Charged again on every retry
    await gateway.ChargeAsync(
        request.CardToken, request.Amount);

    db.Payments.Add(payment);
    await db.SaveChangesAsync();

    return Results.Created(
        $"/api/payments/{payment.Id}", payment);
});
```

**Solution:** Implement idempotency keys on POST and PUT operations. If the key was already processed, return the original response instead of executing again.

```csharp
// ✅ GOOD — Idempotency key prevents duplicate processing
app.MapPost("/api/payments", async (
    [FromHeader(Name = "Idempotency-Key")]
    string idempotencyKey,
    ProcessPaymentRequest request,
    IPaymentService paymentService) =>
{
    var existing = await paymentService
        .GetByIdempotencyKeyAsync(idempotencyKey);

    if (existing is not null)
    {
        return Results.Ok(existing); // Return cached result
    }

    var payment = await paymentService
        .ProcessPaymentAsync(request, idempotencyKey);

    return Results.Created(
        $"/api/payments/{payment.Id}", payment);
});
```
