# 12 Web API Anti-Patterns (.NET)

## The Problems

*Based on content by Anton Martyniuk — antondevtips.com*

---

### 01 — Fat Controllers

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

---

### 02 — No Input Validation

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

---

### 03 — Returning Raw Exceptions

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

---

### 04 — Blocking Async with .Result or .Wait()

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

---

### 05 — Ignoring CancellationTokens

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

---

### 06 — No Pagination

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

---

### 07 — Wrong HTTP Status Codes

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

---

### 08 — Over-fetching Data

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

---

### 09 — Returning EF Entities as API Responses

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

---

### 10 — No Rate Limiting

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

---

### 11 — No Observability

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

---

### 12 — No Idempotency on Mutating Endpoints

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
