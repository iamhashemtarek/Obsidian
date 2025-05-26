
## 🧱 1. Conceptual Architecture

```
Controller → Command/Query → Handler → (Decorators: Validation/Logging) → Business Logic → Result<T>
```

- **Commands** = actions that **change state** (e.g., CreateOrder).
    
- **Queries** = actions that **fetch data** (e.g., GetOrderById).
    
- **Handlers** = classes that execute the command/query logic.
    
- **Decorators**:
    
    - `ValidationDecorator` → runs FluentValidation on the input before the handler.
        
    - `LoggingDecorator` → logs before and after the handler runs.
        
- **Result** → A wrapper for success/failure responses (like `OneOf`, `Either`, or `ErrorOr`).
    

---

## 🛠️ 2. How to Use This Pattern

### ✅ Step-by-Step: Creating and Using a Command

Let’s say you want to implement **CreateTodoCommand**.

---

### 1. **Define the Command**

```csharp
public sealed record CreateTodoCommand(string Title) : ICommand<Guid>; // Return created ID
```

---

### 2. **Implement the Command Handler**

```csharp
public class CreateTodoCommandHandler : ICommandHandler<CreateTodoCommand, Guid>
{
    public async Task<Result<Guid>> Handle(CreateTodoCommand command, CancellationToken cancellationToken)
    {
        // Business logic
        var todoId = Guid.NewGuid(); // Example
        return Result.Success(todoId);
    }
}
```

---

### 3. **Optional: Add Validation**

```csharp
public class CreateTodoCommandValidator : AbstractValidator<CreateTodoCommand>
{
    public CreateTodoCommandValidator()
    {
        RuleFor(c => c.Title)
            .NotEmpty().WithErrorCode("TitleRequired")
            .MaximumLength(100).WithErrorCode("TitleTooLong");
    }
}
```

---

### 4. **Call the Command from Controller**

```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateTodoRequest request)
{
    var command = new CreateTodoCommand(request.Title);
    var handler = GetCommandHandler<CreateTodoCommand, Guid>();
    var result = await handler.Handle(command, CancellationToken.None);

    return result.IsSuccess
        ? Ok(result.Value)
        : BadRequest(result.Error);
}
```

---

### 5. **Done. Decorators handle validation + logging**

You don’t need to manually validate or log — the **decorators** are automatically applied via `.Decorate()` calls in `DependencyInjection.cs`.

---

## 🔄 3. Workflow Lifecycle of a Command or Query

Here’s what happens under the hood when you execute a command:

### 🧠 Lifecycle: `CreateTodoCommand`

1. **Controller** creates `CreateTodoCommand`.
    
2. `BaseController` calls:
    
    ```csharp
    GetCommandHandler<CreateTodoCommand, Guid>()
    ```
    
    → This resolves the decorated command handler from DI container.
    
3. **Service Collection** has registered:
    
    - `CreateTodoCommandHandler` as the inner handler.
        
    - Decorated by:
        
        - `ValidationDecorator.CommandHandler<CreateTodoCommand, Guid>`
            
        - `LoggingDecorator.CommandHandler<CreateTodoCommand, Guid>`
            
4. Call stack becomes:
    
    ```plaintext
    LoggingDecorator
       → ValidationDecorator
          → CreateTodoCommandHandler
    ```
    
5. Execution order:
    
    1. LoggingDecorator logs "Processing CreateTodoCommand".
        
    2. ValidationDecorator validates the command using FluentValidation.
        
    3. If valid → Calls CreateTodoCommandHandler.
        
    4. If handler returns Result.Success → Logging logs "Completed".
        
    5. If Result.Failure → Error pushed to Serilog context and logged.
        
    6. Result is returned to controller.
        

---

## 🔁 Same Logic for Queries

You define:

```csharp
public sealed record GetTodoByIdQuery(Guid Id) : IQuery<TodoDto>;
```

And a corresponding `IQueryHandler<TQuery, TResponse>` implementation.

Then inject it like:

```csharp
var result = await GetQueryHandler<GetTodoByIdQuery, TodoDto>()
    .Handle(new GetTodoByIdQuery(id), cancellationToken);
```

---

## 📦 Example Use Case Flow (Summary)

Let’s say you hit:

`POST /api/todos` with `{ "title": "Buy Milk" }`

### Flow:

1. Controller → creates `CreateTodoCommand("Buy Milk")`
    
2. Resolves `ICommandHandler<CreateTodoCommand, Guid>` from DI.
    
3. Decorators wrap the handler:
    
    - `ValidationDecorator` → validates title.
        
    - `LoggingDecorator` → logs before and after.
        
4. `CreateTodoCommandHandler` executes business logic.
    
5. Returns `Result<Guid>` with new Todo ID.
    
6. Controller sends `200 OK` with ID.
    
