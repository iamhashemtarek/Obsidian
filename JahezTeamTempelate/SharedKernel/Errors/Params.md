The **`params`** keyword in C# allows a method to accept a **variable number of arguments** without explicitly passing an array. It simplifies method calls by letting you pass multiple values directly.

### **How `params` Works**
- The parameter marked with `params` must be **the last parameter** in the method signature.
- **==It allows passing zero or more arguments.**==
- Internally, the arguments are treated as an **array**.

### **Example Usage**
#### **1. Without `params` (Manual Array Passing)**
```csharp
public void PrintNumbers(int[] numbers)
{
    foreach (var num in numbers)
        Console.WriteLine(num);
}

// Calling the method
PrintNumbers(new int[] { 1, 2, 3, 4 });
```
Here, you **must** create an array manually before passing it.

#### **2. With `params` (Simplified Call)**
```csharp
public void PrintNumbers(params int[] numbers)
{
    foreach (var num in numbers)
        Console.WriteLine(num);
}

// Calling the method
PrintNumbers(1, 2, 3, 4); // No need to create an array manually
PrintNumbers(); // Works fine, `numbers` will be an empty array
```
Now, you can pass **individual values** directly, making the call more intuitive.
