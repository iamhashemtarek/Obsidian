### **Understanding `Result` Class**
==The `Result` class is a **utility type** for handling **success and failure outcomes** in a structured way. It eliminates `null` checks and exceptions by encapsulating **valid results** and **errors**.==

---

### **Breakdown of `Result`**
```csharp
public class Result
```
- Represents an **operation result** that can be **successful** or **failed**.
- Stores an `Error` instance if the operation fails.

#### **1. Constructor**
```csharp
public Result(bool isSuccess, Error error)
{
    if (isSuccess && error != Error.None ||
        !isSuccess && error == Error.None)
    {
        throw new ArgumentException("Invalid error", nameof(error));
    }

    IsSuccess = isSuccess;
    Error = error;
    Message = LocalizationHelper.GetString(error.Code, error.Args);
}
```
- ==**Validates that success cannot have an error, and failure must have an error.==**
- Stores:
  - `IsSuccess`: **True** if successful.
  - `Error`: The corresponding error.
  - `Message`: Localized error message using `LocalizationHelper.GetString(...)`.

	#### **2. Properties**
```csharp
public bool IsSuccess { get; }
public bool IsFailure => !IsSuccess;
public Error Error { get; }
public string Message { get; private set; }
```
- **`IsSuccess` / `IsFailure`**: Boolean flags to check success/failure status.
- **`Error`**: Stores the failure reason.
- **`Message`**: Stores the localized error message.

#### **3. Static Factory Methods**
```csharp
public static Result Success() => new(true, Error.None);
public static Result<TValue> Success<TValue>(TValue value) =>
    new(value, true, Error.None);

public static Result Failure(Error error) => new(false, error);
public static Result<TValue> Failure<TValue>(Error error) =>
    new(default, false, error);
```
- **Creates reusable success/failure results.**
- The **generic versions (`Result<TValue>`)** allow returning values for successful operations.

---

### **Breakdown of `Result<TValue>`**
```csharp
public class Result<TValue> : Result
```
- Generic version of `Result` that **stores a return value** (`TValue`).

#### **1. Constructor**
```csharp
public Result(TValue value, bool isSuccess, Error error)
    : base(isSuccess, error)
{
    _value = value;
}
```
- Stores `_value` if **successful**, otherwise keeps an **error**.

#### **2. Value Property**
```csharp
[NotNull]
public TValue Value => IsSuccess
    ? _value!
    : throw new InvalidOperationException("The value of a failure result can't be accessed.");
```
- **Throws an exception** if accessed on a failure result.
- **`NotNull` attribute** ensures correctness in nullable contexts.

#### **3. Implicit Conversion Operator**
```csharp
public static implicit operator Result<TValue>(TValue value) =>
    value is not null ? Success(value) : Failure<TValue>(Error.NullValue);
```
- Allows automatic conversion from `TValue` to `Result<TValue>`.
- Example:
  ```csharp
  Result<string> result = "Hello"; // Implicitly calls Success("Hello")
  Result<string> nullResult = null; // Calls Failure(Error.NullValue)
  ```

#### **4. Validation Failure Factory**
```csharp
public static Result<TValue> ValidationFailure(Error error) =>
    new(default, false, error);
```
- Creates validation failure results.

---
