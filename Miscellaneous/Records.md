Sure! Let’s dive deep into **records in C#** — one of the most important features introduced in **C# 9.0+** (used in .NET 5 and later). This explanation will cover everything you need as a backend developer using .NET Core — including syntax, behavior, use cases, how they're different from classes, and performance aspects.

---

## 🧾 What Are Records in C#?

### ➤ **Definition:**

> A **record** is a special reference type that provides built-in **value-based equality**, **immutability**, and **non-destructive mutation** via `with`-expressions.

In simple terms:

- Think of **class** = focuses on **identity**
    
- Think of **record** = focuses on **data and equality**
    

---

## 🛠️ Syntax Overview

### ✅ Positional Record (Immutable, concise)

```csharp
public record User(string Name, int Age);
```

### ✅ Object-Initializer Style Record

```csharp
public record User
{
    public string Name { get; init; }
    public int Age { get; init; }
}
```

- `init` accessor allows you to set the property **only during initialization**.
    
- Once set, it behaves like a read-only property.
    

---

## 💡 Key Features of Records

### 1. ✅ **Value-Based Equality**

Unlike classes (which compare by reference), records compare values.

```csharp
var u1 = new User("Ali", 30);
var u2 = new User("Ali", 30);

Console.WriteLine(u1 == u2); // True ✅
```

> Classes would return `False` unless you override `Equals()` and `GetHashCode()` manually.

---

### 2. 🔁 **Non-Destructive Mutation (`with`-expression)**

Instead of changing an existing object, you create a new one.

```csharp
var user1 = new User("Ali", 30);
var user2 = user1 with { Age = 31 };

Console.WriteLine(user1); // User { Name = Ali, Age = 30 }
Console.WriteLine(user2); // User { Name = Ali, Age = 31 }
```

---

### 3. 📃 **Auto-generated `ToString()`**

```csharp
Console.WriteLine(user1); 
// Output: User { Name = Ali, Age = 30 }
```

Much better for logging or debugging than default class `ToString()`.

---

### 4. 📏 **Inheritance Support (Record Classes)**

Records can be **inherited** like classes:

```csharp
public record Animal(string Name);
public record Dog(string Name, string Breed) : Animal(Name);
```

But immutability is maintained.

---

### 5. 🔐 **Immutability by Default**

With positional records or `init` accessors, properties are read-only after creation.

This supports functional programming and prevents bugs caused by accidental modification.

---

## 🆚 Records vs Classes: Detailed Comparison

|Feature|`class`|`record`|
|---|---|---|
|Default equality|Reference-based|Value-based ✅|
|Mutability|Mutable|Immutable by default ✅|
|`with`-expression|❌ Not supported|✅ Supported|
|`ToString()`|Needs override|Auto-generated ✅|
|`Equals()` & `GetHashCode()`|Manual|Auto-implemented ✅|
|Use case|Domain models with identity|DTOs, query/command models, snapshots, log entries|

---

## 📦 Use Cases for Records in .NET Core

|Scenario|Use Records?|Why|
|---|---|---|
|**CQRS Command/Query Models**|✅ Yes|Immutable data, easy to clone with `with`|
|**DTOs (Data Transfer Objects)**|✅ Yes|Lightweight, clean, and immutable|
|**API Request/Response Models**|✅ Yes|Value-based logic, better for serialization|
|**Domain Models (DDD)**|⚠️ Be careful|Classes may be better if identity and behavior matter|
|**Configuration or Snapshot**|✅ Yes|Snapshots shouldn’t change after creation|
|**Entity Framework Models**|❌ No|EF Core requires mutable classes and navigation properties|

---

## 🧠 Behind the Scenes

When you declare:

```csharp
public record User(string Name, int Age);
```

The compiler **automatically generates**:

- Constructor
    
- Properties with `init` setters
    
- `ToString()`
    
- `Equals()` and `GetHashCode()` (value-based)
    
- `Deconstruct()` method for pattern matching
    
- `Clone()` method used by `with`
    

---

## ⚠️ Pitfalls to Watch Out For

### 1. 🔁 Don't use `record` with mutable `set` unless you have to:

```csharp
public record User
{
    public string Name { get; set; } // ❌ Makes it mutable
}
```

This breaks immutability and causes unexpected behavior with `with` and equality checks.

### 2. 🔍 EF Core + Records? Use carefully:

- EF Core prefers classes for mutable tracked entities.
    
- You can use records for **read models**, not entities.
    

### 3. 🧩 Record is a **reference type**, not a value type

- That means they live on the heap and behave like classes in memory.
    
- But equality behaves like value types.
    

---

## 🧪 Example: Using Records in a CQRS .NET Core Web API

```csharp
public record CreateUserCommand(string Name, int Age);

public class CreateUserHandler : IRequestHandler<CreateUserCommand, UserResponse>
{
    public Task<UserResponse> Handle(CreateUserCommand request, CancellationToken ct)
    {
        // request.Name and request.Age are immutable!
        return Task.FromResult(new UserResponse(request.Name, request.Age));
    }
}
```

> Safe, clean, and testable – perfect for CQRS + MediatR.

---

## 🔚 Summary

|Feature|Description|
|---|---|
|**Records**|Reference types that provide value-based equality, immutability, and cloning via `with`|
|**Default behavior**|Immutable, value-focused objects|
|**Best use cases**|DTOs, request/response models, commands, queries, logs, snapshots|
|**Avoid in**|Entity Framework entities or deeply mutable models|
|**Benefits**|Cleaner code, easier testing, thread safety, predictable behavior|

---

Would you like a code sample or GitHub repo using records with MediatR and CQRS in a real .NET Core API setup?