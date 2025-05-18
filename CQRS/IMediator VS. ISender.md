## 🔧 What's the Difference Between `IMediator` and `ISender`?

MediatR defines two main interfaces:

| Interface   | Description                                                                          | Typical Use                                                     |
| ----------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------- |
| `IMediator` | Full-featured interface that can send requests and publish notifications             | Advanced scenarios (e.g., publishing events + sending requests) |
| `ISender`   | Lightweight interface focused **only on sending requests/commands/queries** (`Send`) | Most common usage in modern CQRS systems                        |

---

## ✅ Quick Definitions

### 📌 `ISender`

```csharp
public interface ISender
{
    Task<TResponse> Send<TResponse>(IRequest<TResponse> request, CancellationToken cancellationToken = default);
}
```

- Used only for **request/response** operations: queries or commands.
    
- Clean, minimal surface area — helps enforce **CQRS** boundaries.
    

---

### 📌 `IMediator`

```csharp
public interface IMediator : ISender
{
    Task Publish(object notification, CancellationToken cancellationToken = default);
}
```

- Supports **both**:
    
    - `.Send()` for request/response (queries/commands)
        
    - `.Publish()` for notifications/events (void fan-out handlers)
        

So `IMediator` inherits from `ISender`, but adds **publishing events** (like Domain Events).

---

## 🧠 When to Use Which?

### ✅ Use `ISender` When:

- You're following **pure CQRS** or **Vertical Slice Architecture**.
    
- You only need to **send queries or commands** (i.e., request/response model).
    
- You want to **enforce separation**: your app only _sends_, it doesn’t _publish_ domain events.
    

**Examples**:

```csharp
public class UsersController : ControllerBase
{
    private readonly ISender _sender;

    public UsersController(ISender sender) => _sender = sender;

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(Guid id)
    {
        var result = await _sender.Send(new GetUserByIdQuery(id));
        return Ok(result);
    }
}
```

---

### ✅ Use `IMediator` When:

- You need both:
    
    - **Send()** for request/response
        
    - **Publish()** for notifications (e.g., Domain Events, Eventual Consistency)
        
- You're implementing **Domain-Driven Design (DDD)** patterns or **event-driven architectures**.
    

**Examples**:

```csharp
public class UserCreatedHandler : IRequestHandler<CreateUserCommand, Guid>
{
    private readonly IMediator _mediator;

    public UserCreatedHandler(IMediator mediator) => _mediator = mediator;

    public async Task<Guid> Handle(CreateUserCommand command, CancellationToken cancellationToken)
    {
        var userId = await _repository.CreateUserAsync(command);

        // Publish domain event
        await _mediator.Publish(new UserCreatedNotification(userId), cancellationToken);

        return userId;
    }
}
```

---

## ✅ Summary Table

|Scenario|Use `ISender`|Use `IMediator`|
|---|---|---|
|Sending query or command|✅|✅|
|Publishing domain events|❌|✅|
|Clean CQRS-only apps|✅|❌|
|DDD or Event-driven apps|❌|✅|
|Want minimal API surface|✅|❌|

---

## 🧪 Best Practice (CQRS):

- Inject **`ISender`** into controllers or handlers — keeps the surface small and focused.
    
- Only inject **`IMediator`** into domain logic or layers that **publish events**.
    

---

## ✅ Example Combining Both

```csharp
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    private readonly IOrderRepository _repository;
    private readonly IMediator _mediator;

    public CreateOrderHandler(IOrderRepository repository, IMediator mediator)
    {
        _repository = repository;
        _mediator = mediator;
    }

    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var order = new Order(request.CustomerId, request.Items);
        await _repository.AddAsync(order);

        // Publish domain event
        await _mediator.Publish(new OrderCreatedEvent(order.Id));

        return order.Id;
    }
}
```

In your **controller**, just use `ISender`:

```csharp
public class OrdersController : ControllerBase
{
    private readonly ISender _sender;

    public OrdersController(ISender sender) => _sender = sender;

    [HttpPost]
    public async Task<IActionResult> CreateOrder(CreateOrderCommand command)
    {
        var id = await _sender.Send(command);
        return CreatedAtAction(nameof(GetById), new { id }, null);
    }
}
```
