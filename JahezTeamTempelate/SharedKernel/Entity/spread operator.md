
```
public abstract class Entity : IEntity
{
    private readonly List<IDomainEvent> _domainEvents = [];

    public List<IDomainEvent> DomainEvents => [.. _domainEvents];
```
The **spread operator** (`..`) is a new feature introduced in **C# 12** that simplifies working with collections like **arrays, lists, and dictionaries**. It allows you to **expand** an existing collection into another collection.

### **How It Works**
The spread operator helps in:
1. **Cloning collections** (creating a shallow copy)
2. **Merging multiple collections** into one
3. **Expanding elements** inside a new collection

### **Examples**
#### **1. Cloning an Array**
```csharp
int[] originalArray = {1, 2, 3};
int[] clonedArray = [.. originalArray]; // Creates a new array with the same elements
```
Here, `clonedArray` is a **new array** containing all elements of `originalArray`.

#### **2. Merging Collections**
```csharp
int[] firstArray = {1, 2, 3};
int[] secondArray = {4, 5, 6};
int[] mergedArray = [.. firstArray, .. secondArray]; // Combines both arrays
```
Now, `mergedArray` contains `{1, 2, 3, 4, 5, 6}`.

#### **3. Using Spread in Lists**
```csharp
List<string> names = ["Alice", "Bob"];
List<string> allNames = [.. names, "Charlie", "David"];
```
This expands `names` into `allNames`, adding `"Charlie"` and `"David"`.
