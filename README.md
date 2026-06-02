# Minimal API Demo

An ASP.NET Core 9 Minimal API showing:

- **Endpoints split across files** — each resource has its own file under `Endpoints/` exposing an `IEndpointRouteBuilder` extension method, so `Program.cs` stays tiny.
- **Global exception handling** — a single `IExceptionHandler` turns exceptions into RFC 7807 `ProblemDetails` responses.
- **API documentation with Scalar** — OpenAPI document plus an interactive Scalar UI.

## Run it

```bash
cd MinimalApiDemo
dotnet run
```

Then open the interactive docs:

- Scalar UI: <http://localhost:5080/scalar/v1>
- Raw OpenAPI JSON: <http://localhost:5080/openapi/v1.json>

(Ports come from `Properties/launchSettings.json` — adjust as needed.)

## Project layout

```
MinimalApiDemo/
├── Program.cs                 # wires up services + pipeline, maps endpoint groups
├── Models/Catalog.cs          # Product, Category + request DTOs
├── Data/CatalogStore.cs       # in-memory store (swap for a real DB)
├── Exceptions/
│   ├── AppExceptions.cs       # NotFoundException, ValidationException
│   └── GlobalExceptionHandler.cs
└── Endpoints/
    ├── ProductEndpoints.cs    # /api/products  (full CRUD)
    ├── CategoryEndpoints.cs   # /api/categories (full CRUD)
    └── HealthEndpoints.cs     # /api/health, /api/health/boom
```

## How the pieces fit

**Splitting endpoints.** Each file defines a `static` class with one extension method
on `IEndpointRouteBuilder` (e.g. `MapProductEndpoints`). It creates a route group
(`app.MapGroup("/api/products")`) and hangs the handlers off it. `Program.cs` just calls
`app.MapProductEndpoints()`. Add a new resource by creating one more file and one more call.

**Exception handling.** Handlers throw `NotFoundException` / `ValidationException` instead of
manually building error responses. `GlobalExceptionHandler` maps those to 404/400, and anything
else to a 500 (without leaking internal details). Every error comes back as `application/problem+json`.

**Documentation.** `AddOpenApi()` + `MapOpenApi()` produce the OpenAPI document. The
`.WithSummary()`, `.WithDescription()`, `.Produces<T>()`, and `.ProducesProblem()` calls on each
endpoint enrich that document, and Scalar renders it as a browsable, try-it-out UI.

Try `GET /api/health/boom` in the Scalar UI to see the global handler produce a clean 500 ProblemDetails.

## Swapping in a real database

The endpoints depend only on `CatalogStore`'s methods, not on how data is stored. To use a real
database, replace `CatalogStore` with an EF Core `DbContext` (or a repository), register it with
`builder.Services.AddDbContext<...>()`, and inject it into the handlers the same way. No endpoint
code needs to change beyond the parameter type.
