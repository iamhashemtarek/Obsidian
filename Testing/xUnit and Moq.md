# Deep Dive: Unit Testing in .NET Core API Using xUnit and Moq

Unit testing is the foundation of reliable software. In .NET Core, `xUnit` and `Moq` provide a powerful combination for writing clean, isolated, and expressive unit tests.

---

## 1. What Is Unit Testing?

**Unit testing** is the practice of testing individual units (usually methods or classes) of source code in isolation. The main goals are:

- **Validation**: Ensure methods produce correct results.
    
- **Regression detection**: Catch unintended side effects during development.
    
- **Design feedback**: Encourage loosely coupled, testable code.
    

A unit test:

- Should run quickly.
    
- Should not rely on external systems (database, file system, network).
    
- Should assert the output or behavior of a small, isolated unit.
    

---

## 2. Test-Driven Development (TDD) vs Traditional Unit Testing

|Style|Description|
|---|---|
|**TDD**|Write tests first, then write code to make tests pass.|
|**Traditional**|Write production code, then write tests.|

TDD ensures testability from the beginning and promotes better code design.

---

## 3. Tooling: xUnit and Moq

### xUnit

- An open-source .NET test framework.
    
- Fully supports .NET Core.
    
- Attributes:
    
    - `[Fact]`: A parameterless test.
        
    - `[Theory]`: A parameterized test.
        
    - `[InlineData(...)]`: Supplies test data.
        

### Moq

- A mocking library for .NET.
    
- Allows creation of in-memory objects that mimic behavior of real dependencies.
    
- Supports behavior verification and stubbing return values.
    

---

## 4. Setting Up the Test Project

```bash
dotnet new xunit -n MyApi.Tests
cd MyApi.Tests
dotnet add reference ../MyApi/MyApi.csproj
dotnet add package Moq
dotnet add package Microsoft.NET.Test.Sdk
dotnet add package coverlet.collector # For coverage reports (optional)
```

---

## 5. Example Project Structure

```
MyApi/
├── Controllers/
│   └── ProductsController.cs
├── Services/
│   ├── IProductService.cs
│   └── ProductService.cs
├── Models/
│   └── Product.cs

MyApi.Tests/
├── Controllers/
│   └── ProductsControllerTests.cs
├── Services/
│   └── ProductServiceTests.cs
```

---

## 6. Sample API Scenario

### Product Model

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

### IProductService Interface

```csharp
public interface IProductService
{
    Product GetById(int id);
    IEnumerable<Product> GetAll();
    bool Delete(int id);
}
```

### ProductService (Implementation)

```csharp
public class ProductService : IProductService
{
    private readonly List<Product> _products = new() {
        new Product { Id = 1, Name = "Phone", Price = 500 },
        new Product { Id = 2, Name = "Tablet", Price = 800 }
    };

    public Product GetById(int id) => _products.FirstOrDefault(p => p.Id == id);
    public IEnumerable<Product> GetAll() => _products;
    public bool Delete(int id)
    {
        var product = GetById(id);
        if (product == null) return false;
        _products.Remove(product);
        return true;
    }
}
```

### ProductsController

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;

    public ProductsController(IProductService service) => _service = service;

    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        var product = _service.GetById(id);
        if (product == null) return NotFound();
        return Ok(product);
    }
}
```

---

## 7. Writing Unit Tests

### Controller Tests with Moq

```csharp
public class ProductsControllerTests
{
    private readonly Mock<IProductService> _mockService;
    private readonly ProductsController _controller;

    public ProductsControllerTests()
    {
        _mockService = new Mock<IProductService>();
        _controller = new ProductsController(_mockService.Object);
    }

    [Fact]
    public void Get_ExistingId_ReturnsOkWithProduct()
    {
        var product = new Product { Id = 1, Name = "Phone", Price = 500 };
        _mockService.Setup(s => s.GetById(1)).Returns(product);

        var result = _controller.Get(1);

        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnedProduct = Assert.IsType<Product>(okResult.Value);
        Assert.Equal(1, returnedProduct.Id);
    }

    [Fact]
    public void Get_InvalidId_ReturnsNotFound()
    {
        _mockService.Setup(s => s.GetById(999)).Returns((Product)null);
        var result = _controller.Get(999);
        Assert.IsType<NotFoundResult>(result);
    }
}
```

### Service Test Without Moq

```csharp
public class ProductServiceTests
{
    private readonly ProductService _service = new();

    [Fact]
    public void GetById_ExistingId_ReturnsProduct()
    {
        var product = _service.GetById(1);
        Assert.NotNull(product);
        Assert.Equal("Phone", product.Name);
    }

    [Fact]
    public void Delete_NonExistentId_ReturnsFalse()
    {
        var result = _service.Delete(999);
        Assert.False(result);
    }
}
```

---

## 8. Best Practices

### ✅ Test Naming Convention

Use the form: `MethodName_StateUnderTest_ExpectedOutcome`

### ✅ Follow Arrange-Act-Assert Pattern

```csharp
// Arrange
// Act
// Assert
```

### ✅ Use `Theory` + `InlineData` for Parameterized Tests

```csharp
[Theory]
[InlineData(1, "Phone")]
[InlineData(2, "Tablet")]
public void GetById_ReturnsCorrectProduct(int id, string expectedName)
{
    _mockService.Setup(x => x.GetById(id)).Returns(new Product { Id = id, Name = expectedName });
    var result = _controller.Get(id);
    var ok = Assert.IsType<OkObjectResult>(result);
    var product = Assert.IsType<Product>(ok.Value);
    Assert.Equal(expectedName, product.Name);
}
```

### ✅ Verify Mock Interactions

```csharp
_mockService.Verify(s => s.GetById(1), Times.Once);
```

### ✅ Group Tests Logically

Use regions or separate classes for testing different functionalities.

### ✅ Keep Tests Independent

Each test should have no dependency on the order or output of another test.

---

## 9. Integration Testing

Integration tests verify the complete application behavior, including routing, middleware, and interactions between components (e.g., services, databases).

### Setting Up

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

### Example Test Using WebApplicationFactory

```csharp
public class ProductsIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ProductsIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Get_ReturnsSuccessAndProduct()
    {
        var response = await _client.GetAsync("/api/products/1");
        response.EnsureSuccessStatusCode();

        var json = await response.Content.ReadAsStringAsync();
        Assert.Contains("Phone", json);
    }
}
```

Ensure your `Program.cs` or `Startup.cs` is test-friendly (e.g., avoid hardcoded dependencies).

---

## 10. Mocking HttpClient in Unit Tests

You often need to unit test services that use `HttpClient`. To mock it, use `HttpMessageHandler`.

### Step-by-Step Mocking

```csharp
public class ProductClient
{
    private readonly HttpClient _httpClient;

    public ProductClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> GetProductAsync(int id)
    {
        var response = await _httpClient.GetAsync($"https://fakeapi.com/products/{id}");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsStringAsync();
    }
}

public class ProductClientTests
{
    [Fact]
    public async Task GetProductAsync_ReturnsContent()
    {
        var handlerMock = new Mock<HttpMessageHandler>();
        handlerMock
            .Protected()
            .Setup<Task<HttpResponseMessage>>(
                "SendAsync",
                ItExpr.IsAny<HttpRequestMessage>(),
                ItExpr.IsAny<CancellationToken>())
            .ReturnsAsync(new HttpResponseMessage
            {
                StatusCode = HttpStatusCode.OK,
                Content = new StringContent("{\"id\":1,\"name\":\"Phone\"}"),
            });

        var httpClient = new HttpClient(handlerMock.Object);
        var client = new ProductClient(httpClient);

        var result = await client.GetProductAsync(1);
        Assert.Contains("Phone", result);
    }
}
```

---

## 11. Running Tests

### CLI

```bash
dotnet test
```

### Coverage (Optional)

```bash
dotnet test --collect:"XPlat Code Coverage"
```

Results are output as `.coverage` or `.cobertura.xml`, and can be viewed in report tools.

---

## 12. Summary Checklist to Master Unit Testing

|Skill|Description|
|---|---|
|xUnit Attributes|Understand `[Fact]`, `[Theory]`, etc.|
|Moq Basics|Setup, Returns, Verify|
|Isolated Unit Testing|No external system dependencies|
|Arrange-Act-Assert|Clear, structured test logic|
|Naming Conventions|Descriptive test names|
|Parameterized Testing|Reusable, DRY tests|
|Coverage Awareness|Know what parts are untested|
|Integration Testing|End-to-end path verification|
|HttpClient Mocking|Mocking external HTTP calls|

---

## 13. Resources

- [xUnit Docs](https://xunit.net/docs/getting-started/netcore/cmdline)
    
- [Moq Docs](https://github.com/moq/moq4/wiki/Quickstart)
    
- [Microsoft: Unit testing best practices](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
    
- [Mocking HttpClient with Moq](https://github.com/richardbarker/mock-httpclient)
    
- [WebApplicationFactory Docs](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)