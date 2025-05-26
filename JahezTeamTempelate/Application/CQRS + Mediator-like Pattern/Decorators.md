## 🧩 What Are Decorators?

**Decorators** are a **design pattern** used to add extra behavior **before or after** the core logic of a class **without modifying the class itself**.

> They _wrap_ the original class to extend or enhance its behavior **transparently**.

Think of it like this:

- You have a **handler** that performs the main job (e.g., `CreateTodoCommandHandler`).
    
- You want to **add logging**, **validation**, or **caching** around it...
    
- But you **don’t want to write that code inside the handler** itself every time.
    

Instead, you create a **Decorator** that wraps the handler and adds that behavior.

---
## 🧪 Basic Example in CSharp

### 1. **Main Interface**

```csharp
public interface ICommandHandler<TCommand, TResult>
{
    Task<Result<TResult>> Handle(TCommand command, CancellationToken cancellationToken);
}
```

---

### 2. **Concrete Handler**

```csharp
public class CreateTodoCommandHandler : ICommandHandler<CreateTodoCommand, Guid>
{
    public Task<Result<Guid>> Handle(CreateTodoCommand command, CancellationToken cancellationToken)
    {
        // Business logic
        return Task.FromResult(Result.Success(Guid.NewGuid()));
    }
}
```

---

### 3. **Validation Decorator**

```csharp
public class ValidationDecorator<TCommand, TResult> : ICommandHandler<TCommand, TResult>
    where TCommand : ICommand<TResult>
{
    private readonly ICommandHandler<TCommand, TResult> _inner;
    private readonly IEnumerable<IValidator<TCommand>> _validators;

    public ValidationDecorator(
        ICommandHandler<TCommand, TResult> inner,
        IEnumerable<IValidator<TCommand>> validators)
    {
        _inner = inner;
        _validators = validators;
    }

    public async Task<Result<TResult>> Handle(TCommand command, CancellationToken cancellationToken)
    {
        foreach (var validator in _validators)
        {
            var result = await validator.ValidateAsync(command, cancellationToken);
            if (!result.IsValid)
            {
                // Stop pipeline and return error
                return Result.Failure<TResult>(result.Errors.Select(e => e.ErrorMessage).ToList());
            }
        }

        // Call the next handler
        return await _inner.Handle(command, cancellationToken);
    }
}
```

---

### 4. **Logging Decorator**

```csharp
public class LoggingDecorator<TCommand, TResult> : ICommandHandler<TCommand, TResult>
    where TCommand : ICommand<TResult>
{
    private readonly ICommandHandler<TCommand, TResult> _inner;
    private readonly ILogger<LoggingDecorator<TCommand, TResult>> _logger;

    public LoggingDecorator(
        ICommandHandler<TCommand, TResult> inner,
        ILogger<LoggingDecorator<TCommand, TResult>> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task<Result<TResult>> Handle(TCommand command, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Handling command: {CommandType}", typeof(TCommand).Name);
        var result = await _inner.Handle(command, cancellationToken);
        _logger.LogInformation("Command handled: {CommandType}, Success: {Success}", typeof(TCommand).Name, result.IsSuccess);
        return result;
    }
}
```

---

## 🔄 Decoration Pipeline

When resolving a handler from DI, you **wrap it in decorators like layers**:

```plaintext
LoggingDecorator
    → ValidationDecorator
        → CreateTodoCommandHandler
```

Each one calls the next:

- Logging logs before/after
    
- Validation checks command input
    
- Main handler does the business logic
    
