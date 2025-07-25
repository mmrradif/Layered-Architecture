# Layered-Architecture

Layered-Architecture is a comprehensive, step-by-step guide to building clean, maintainable, and testable ASP.NET Core Web API applications following a well-defined layered architecture pattern.


## 1. What is Layered Architecture?

Layered Architecture divides an application into multiple logical layers, each with a clear responsibility and role. This pattern is widely adopted in enterprise software for:

- **Separation of Concerns:** Isolates responsibilities into distinct layers.
- **Maintainability:** Allows independent development, testing, and modification.
- **Testability:** Enables mocking and unit testing of individual layers.
- **Scalability & Flexibility:** Supports evolution of layers without impacting others.



## 2. Core Layers and Their Responsibilities

| Layer                   | Responsibility                              | Typical Contents                                  |
|-------------------------|---------------------------------------------|--------------------------------------------------|
| **API Layer (Presentation)** | Expose HTTP endpoints, handle requests/responses | Controllers, routing, filters, middleware         |
| **Business Logic Layer (BLL)** | Implement business rules, validation, workflows | Service interfaces & implementations, DTO mapping |
| **Data Access Layer (DAL)**   | Manage data persistence and queries          | DbContext, entity models, repositories             |
| **Shared/Common Layer**       | Share reusable models and utilities across layers | DTOs, AutoMapper profiles, helpers, constants      |



## 3. Folder Structure Overview

![Folder Structure](https://github.com/mmrradif/Layered-Architecture/blob/3e4502ce811fc95b9597c2f8c32516f5a1076691/Folder%20Structure.png)

The solution is organized into four main projects/layers, each with clear responsibilities:

- **MyApp.API** — Presentation Layer (Controllers, Program.cs, DI)
- **MyApp.BLL** — Business Logic Layer (Services, Interfaces, DI)
- **MyApp.DAL** — Data Access Layer (Entities, DbContext, Repositories, DI)
- **MyApp.Shared** — Shared/Common Layer (DTOs, Mappings, Helpers)

This folder structure promotes modularity, separation of concerns, and maintainability.



## 4. Layer Responsibilities and Interaction

### API Layer
- Entry point for HTTP clients.
- Contains thin controllers only.
- Calls **only** the Business Logic Layer (BLL).
- Example: `UsersController` calls `IUserService`.

### Business Logic Layer (BLL)
- Core business rules, validations, workflows.
- Contains service interfaces and their implementations.
- Coordinates repository calls to the Data Access Layer (DAL).
- Maps between entities and DTOs.
- Example: `UserService` calls `IUserRepository`.

### Data Access Layer (DAL)
- Isolates data persistence.
- Contains EF Core `DbContext`, entity models, and repositories.
- Encapsulates CRUD and query logic.
- Example: `UserRepository` implements `IUserRepository`.

### Shared Layer
- Contains reusable DTOs, mapping profiles, and helpers.
- Shared models used across API, BLL, and DAL.
- Example: `UserDto`, `UserMappingProfile`.



## 5. Dependency Injection Setup

### DALInjector.cs

```csharp
public static class DALInjector
{
    public static IServiceCollection AddDALServices(this IServiceCollection services)
    {
        services.AddScoped<IUserRepository, UserRepository>();
        services.AddScoped<IOrderRepository, OrderRepository>();
        return services;
    }
}
```

### BLLInjector.cs

```csharp
public static class BLLInjector
{
    public static IServiceCollection AddBLLServices(this IServiceCollection services)
    {
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IOrderService, OrderService>();
        return services;
    }
}
```

### Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDALServices();
builder.Services.AddBLLServices();

builder.Services.AddControllers();
builder.Services.AddSwaggerGen();
builder.Services.AddAutoMapper(typeof(UserMappingProfile).Assembly);

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```


## 6. Layer Interaction Examples

### Controller calls BLL service (recommended):

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;

    public UsersController(IUserService userService) => _userService = userService;

    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(Guid id)
    {
        var userDto = await _userService.GetUserByIdAsync(id);
        if (userDto == null) return NotFound();
        return Ok(userDto);
    }
}
```


### BLL calls DAL repository:

```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly IMapper _mapper;

    public UserService(IUserRepository userRepository, IMapper mapper)
    {
        _userRepository = userRepository;
        _mapper = mapper;
    }

    public async Task<UserDto?> GetUserByIdAsync(Guid id)
    {
        var entity = await _userRepository.GetByIdAsync(id);
        return entity == null ? null : _mapper.Map<UserDto>(entity);
    }
}

```

## Best Practices

To ensure a robust and maintainable ASP.NET Core Web API, adhere to these guidelines:

1. **Minimize Controller Logic**  
   - Keep controllers lightweight, focusing on HTTP request/response handling.
   - Delegate all business logic to the BLL for better separation of concerns.

2. **Encapsulate Data Access**  
   - Restrict EF Core entities to the DAL to prevent leakage.
   - Use repositories to abstract database operations.

3. **Leverage Interfaces**  
   - Implement interfaces for services and repositories to promote loose coupling.
   - Enhances testability by allowing mocking during unit tests.

4. **Avoid Direct DbContext Injection**  
   - Inject repositories instead of `DbContext` into controllers or services.
   - Reduces tight coupling and improves maintainability.

5. **Use DTOs for API Contracts**  
   - Expose DTOs to external clients instead of EF Core entities.
   - Protects internal data models and ensures controlled data exposure.

**Why It Matters**: These practices create a clean, scalable, and testable architecture, simplifying maintenance and future enhancements.

## Further Resources

- 📄 [Layered Architecture Guide](https://github.com/mmrradif/Layered-Architecture/blob/ff2f344cbc6862e849b7e43e4168718c7267cc71/Layered%20Architecture.pdf) for full implementation details.


