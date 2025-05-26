
### ✅ **High-Level Architecture Overview**

1. **CQRS Pattern**
    
    - **Commands**: Represent operations that **change state** (e.g., create, update, delete).
        
    - **Queries**: Represent operations that **read data** (e.g., get by ID, list).
        
    - Each is handled by a separate **handler**, promoting separation of concerns.
        
2. **Cross-Cutting Concerns**:  
    Implemented using the **Decorator pattern**:
    
    - **LoggingDecorator**: Logs command/query start and result (success or failure).
        
    - **ValidationDecorator**: Validates command input using FluentValidation.
        
3. **Dependency Injection (DI)**:  
    Handlers are registered automatically using `Scrutor.Scan()` and **decorated** with validation and logging layers.
    

---

## 🔧 **Detailed Breakdown**

---

### ✅ **1. Interfaces – Messaging Layer**

#### `IQuery<TResponse>` and `IQueryHandler<TQuery, TResponse>`

```csharp
public interface IQuery<TResponse> { }

public interface IQueryHandler<in TQuery, TResponse>
    where TQuery : IQuery<TResponse>
{
    Task<Result<TResponse>> Handle(TQuery query, CancellationToken cancellationToken);
}
```

- `IQuery<TResponse>`: Marker interface for queries.
    
- `IQueryHandler`: Handles queries and returns a `Result<T>` with either data or an error.
    

---

#### `ICommand` and `ICommand<TResponse>`

```csharp
public interface ICommand { }

public interface ICommand<TResponse> { }
```

- Marker interfaces for commands: one for **void commands**, one for **commands returning a result**.
    

---

#### `ICommandHandler<TCommand>` and `ICommandHandler<TCommand, TResponse>`

```csharp
public interface ICommandHandler<in TCommand>
{
    Task<Result> Handle(TCommand command, CancellationToken cancellationToken);
}

public interface ICommandHandler<in TCommand, TResponse>
{
    Task<Result<TResponse>> Handle(TCommand command, CancellationToken cancellationToken);
}
```

- Handles command execution with or without result.
    

---

### ✅ **2. LoggingDecorator – Logs Command/Query Lifecycle**

Located in `Application.Abstractions.Behaviors.LoggingDecorator`.

Each handler is wrapped to log:

- **Before execution**
    
- **After success/failure**
    
- Logs detailed error if failure occurs using `Serilog.Context`
    

Example for command with result:

```csharp
public async Task<Result<TResponse>> Handle(TCommand command, CancellationToken cancellationToken)
{
    logger.LogInformation("Processing command {Command}", commandName);
    
    var result = await innerHandler.Handle(command, cancellationToken);
    
    if (result.IsSuccess)
        logger.LogInformation("Completed command {Command}", commandName);
    else
        logger.LogError("Completed command {Command} with error", commandName);
        
    return result;
}
```

Similar handlers exist for:

- `CommandHandler<TCommand, TResponse>`
    
- `CommandBaseHandler<TCommand>`
    
- `QueryHandler<TQuery, TResponse>`
    

---

### ✅ **3. ValidationDecorator – Validates Input using FluentValidation**

Located in `Application.Abstractions.Behaviors.ValidationDecorator`.

- Uses `IValidator<TCommand>` from FluentValidation.
    
- Runs all registered validators for a command.
    
- If validation fails, wraps the errors in a `ValidationError` and returns a failed `Result`.
    

Example:

```csharp
ValidationFailure[] validationFailures = await ValidateAsync(command, validators);

if (validationFailures.Length == 0)
    return await innerHandler.Handle(command, cancellationToken);

return Result.Failure<TResponse>(CreateValidationError(validationFailures));
```

---

### ✅ **4. Dependency Injection Configuration**

In `Application.DependencyInjection`:

- Scans assemblies and registers:
    
    - Query handlers
        
    - Command handlers (with and without result)
        
    - Domain event handlers
        
- Applies decorators (Validation → Logging)
    
    ```csharp
    services.Decorate(typeof(ICommandHandler<,>), typeof(ValidationDecorator.CommandHandler<,>));
    services.Decorate(typeof(ICommandHandler<,>), typeof(LoggingDecorator.CommandHandler<,>));
    ```
    
- Adds support for localization (Arabic + English).
    
- Adds FluentValidation validators automatically.
    

---

### ✅ **5. BaseController (Web Layer)**

```csharp
public class BaseController : ControllerBase
{
    private readonly IServiceProvider _serviceProvider;

    protected ICommandHandler<TCommand> GetCommandHandler<TCommand>()
    protected ICommandHandler<TCommand, TResponse> GetCommandHandler<TCommand, TResponse>()
    protected IQueryHandler<TQuery, TResult> GetQueryHandler<TQuery, TResult>()
}
```

- Provides base methods to resolve handlers via `IServiceProvider` for controllers.
    
- Promotes handler-based controller methods, e.g.:
    

```csharp
var handler = GetCommandHandler<CreateUserCommand>();
await handler.Handle(command, cancellationToken);
```

---

## 🧠 **What This Architecture Solves**

|Problem|Solution in Your Code|
|---|---|
|Tightly coupled logic|Commands and Queries are separated cleanly|
|Difficult to test|Handlers are isolated and easy to test|
|Repeated validation/logging|Handled by decorators centrally|
|Boilerplate DI setup|Auto-registered via Scrutor scan|
|Unstructured errors|Wrapped in `Result<T>` or `Result` with Error|
