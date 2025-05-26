## 🧠 What Is the Domain Event Pattern?

In **Domain-Driven Design (DDD)**, the **Domain Event Pattern** is a way to **capture important things that happen inside the domain** and **react to them** — _without creating tight coupling between domain logic and side effects_.

---

## 🏭 Problem It Solves

Imagine this:

```plaintext
When a Todo is marked as done:
- You want to update a report.
- You want to notify the user.
- You want to log the action.
```

If you do all of that **inside the domain method**, your entity becomes a **God object**. It's tightly coupled to services it shouldn’t know about.

❌ Bad:

```csharp
// Inside the domain entity
_reportService.Update();
_emailService.Notify();
_logger.Log();
```

✅ Better:

```csharp
// Inside the domain entity
Raise(new TodoMarkedAsDoneDomainEvent(todoId));
```

The entity only raises the event. The rest happens **outside**, in **event handlers**.

---

## 🧱 Building Blocks in Your Code

### 1. **IDomainEvent**

```csharp
public interface IDomainEvent { }
```

- A marker interface — tells the system "this is a domain event."
    

---

### 2. **IDomainEventHandler**

```csharp
public interface IDomainEventHandler<in T> where T : IDomainEvent
{
    Task Handle(T domainEvent, CancellationToken cancellationToken);
}
```

- Any logic that **reacts** to a domain event will implement this interface.
    

---

### 3. **IEntity with DomainEvents**

```csharp
public interface IEntity
{
    List<IDomainEvent> DomainEvents => [];
    void Raise(IDomainEvent domainEvent);
    void ClearDomainEvents();
}
```

- Each domain entity holds a list of **unpublished domain events**.
    
- `Raise()` is called inside domain logic (e.g., in `MarkAsDone()`).
    
- `ClearDomainEvents()` is called after events are dispatched.
    

---

### 4. **DomainEventsDispatcher**

This is the **engine** that:

- Goes through all entities
    
- Collects domain events
    
- Finds matching handlers
    
- Calls each handler
    

### 🔁 How It Works:

1. From `SaveChangesAsync()`, after saving to DB.
    
2. Collects all domain events raised by entities.
    
3. Creates a scoped service provider to get handlers.
    
4. Uses reflection and generics to match `IDomainEventHandler<T>` implementations.
    
5. Invokes `Handle()` for each handler.
    

---

### 5. **ApplicationDbContext Integration**

This is where it all ties together:

```csharp
public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
{
    // Before saving: set CreatedBy, timestamps, etc.
    foreach (EntityEntry<IEntity> entry in ChangeTracker.Entries<IEntity>())
    {
        // auditing logic
    }

    int result = await base.SaveChangesAsync(cancellationToken);

    // After saving: publish domain events
    await PublishDomainEventsAsync();

    return result;
}
```

This ensures domain events are **only published after the database transaction succeeds.**

---

## 🧬 Full Lifecycle of a Domain Event

### Scenario: A Todo is marked as done.

#### 1. **Inside the domain**

```csharp
public void MarkAsDone()
{
    IsDone = true;
    Raise(new TodoMarkedAsDoneEvent(this.Id));
}
```

#### 2. **Entity holds the event**

```csharp
todo.DomainEvents = [TodoMarkedAsDoneEvent];
```

#### 3. **EF SaveChangesAsync() calls dispatcher**

```csharp
await domainEventsDispatcher.DispatchAsync(events);
```

#### 4. **Dispatcher locates the handler**

```csharp
public class TodoMarkedAsDoneHandler : IDomainEventHandler<TodoMarkedAsDoneEvent>
{
    public async Task Handle(TodoMarkedAsDoneEvent domainEvent, CancellationToken ct)
    {
        // Send notification
    }
}
```

---

## 🔧 Why Use It?

|✅ Benefit|💬 Explanation|
|---|---|
|**Decoupling**|Domain model doesn't know infrastructure like email or logging.|
|**Testability**|Handlers are easy to test in isolation.|
|**Consistency**|Events only fired **after** database changes are saved.|
|**Extensibility**|Add new behavior by just registering a new handler.|
|**Single Responsibility**|Each piece of logic has a focused responsibility.|

---

## 🔍 Where to Use Domain Events?

Use them to **react to important state changes** in your domain that have **side effects**:

- Send email when order placed
    
- Publish integration events
    
- Update audit logs
    
- Trigger workflows
    
- Update read models (CQRS)
    

---

## 🧵 Summary

- Domain events **capture state changes** in the domain layer.
    
- Entities **raise events**, but don’t handle side effects directly.
    
- A dispatcher collects and dispatches them to handlers after saving changes.
    
- It promotes **clean architecture**, **DDD**, and **loose coupling**.
    