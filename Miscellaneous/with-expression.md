## The `with`-expression

The `with`-expression allows you to create a **copy** of an immutable record with **some properties changed**.
```csharp
public record Product(string Name, decimal Price);

var product1 = new Product("Laptop", 1000);
var product2 = product1 with { Price = 900 }; // new object

Console.WriteLine(product1); // Product { Name = Laptop, Price = 1000 }
Console.WriteLine(product2); // Product { Name = Laptop, Price = 900 }

```
### 🧠 Internally:

- `with` uses a **copy constructor** behind the scenes.
    
- Only works with **records**, not normal classes (unless you manually implement it).