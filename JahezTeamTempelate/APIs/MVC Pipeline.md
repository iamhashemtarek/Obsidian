# 🏗️ **MVC Request Pipeline Overview**

The **MVC pipeline** is a sub-pipeline within the broader **ASP.NET Core middleware pipeline**. When a request reaches the MVC middleware (typically via `app.UseEndpoints()`), the following sequence occurs:

```
HTTP Request
  ↓
Middleware (Authentication, Routing, etc.)
  ↓
Endpoint Routing
  ↓
MVC Pipeline
  ├── Controller Selection
  ├── Model Binding
  ├── Model Validation
  ├── Action Execution
  ├── Filters (Authorization, Resource, Action, Exception, Result)
  ├── Result Execution
  ↓
HTTP Response
```

---

## 🔹 1. **Routing and Endpoint Selection**

This is handled by ASP.NET Core **middleware** (`UseRouting` and `UseEndpoints`).

- Matches the incoming request to a route (e.g. `/api/account/login`)
    
- Selects the correct `Controller` and `Action` using routing data
    

🔧 In `Startup.cs` (pre .NET 6):

```csharp
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});
```

In .NET 6+ with minimal hosting model:

```csharp
builder.Services.AddControllers();
app.MapControllers();
```

---

## 🔹 2. **Controller Activation**

- ASP.NET Core creates an instance of the controller using **dependency injection (DI)**.
    
- Calls the constructor and injects required services (e.g., `ILogger<AccountController>`, `IUserService`).
    

---

## 🔹 3. **Model Binding**

Model binding maps **HTTP request data** to method parameters.

Supported sources:

- Route (`[FromRoute]`)
    
- Query string (`[FromQuery]`)
    
- Body (`[FromBody]`)
    
- Form (`[FromForm]`)
    
- Header (`[FromHeader]`)
    

Example:

```csharp
[HttpPost]
public IActionResult Register([FromBody] RegisterDto dto)
```

ASP.NET Core parses JSON from the body and maps it to `RegisterDto`.

---

## 🔹 4. **Model Validation**

If you decorate your model with attributes like `[Required]`, `[StringLength]`, etc., **automatic validation** is triggered _if `[ApiController]` is applied_:

```csharp
[ApiController]
[Route("api/[controller]")]
public class AccountController : ControllerBase
```

If the model is invalid:

- The pipeline **short-circuits**
    
- A `400 Bad Request` is returned automatically
    

You can disable this or use manual validation:

```csharp
if (!ModelState.IsValid)
    return BadRequest(ModelState);
```

---

## 🔹 5. **Action Execution**

The controller method is now executed.

- If async: `Task<IActionResult>`
    
- If sync: `IActionResult` or object
    

Return types can be:

- `IActionResult`, `ActionResult<T>`, `Ok()`, `BadRequest()`, `NoContent()`, etc.
    
- String or POCO (`public string Hello()` → `200 OK` with body `Hello`)
    

---

## 🔹 6. **Filters (Very Important)**

Filters are **hook points** into the pipeline:

|Filter Type|Runs...|Use Case|
|---|---|---|
|`Authorization`|Before anything else|Access control, policies|
|`Resource`|Around model binding & validation|Caching, connection checks|
|`Action`|Before/after action execution|Logging, input/output manipulation|
|`Exception`|On unhandled exception|Global exception handling|
|`Result`|Before/after result execution|Modify final result or wrap responses|

Example: A custom action filter

```csharp
public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Before controller action
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        // After controller action
    }
}
```

---

## 🔹 7. **Result Execution**

After the controller returns a result:

- The result is **executed** to write to the HTTP response.
    
- `ActionResult` is turned into a response (JSON, XML, etc.) based on content negotiation.
    
- If returning `ObjectResult`, status code and serialization are handled.
    
- If returning `ViewResult`, the Razor view engine renders HTML.
    

---

# ✅ Summary Flow

```csharp
[HttpPost("login")]
public async Task<ActionResult<LoginResponse>> Login([FromBody] LoginDto input)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var result = await _authService.LoginAsync(input);
    return Ok(result);
}
```

Here's how this flows:

1. Request matches route → `/api/account/login`
    
2. `AccountController` is created
    
3. `LoginDto` is bound from JSON body
    
4. `[ApiController]` validates model
    
5. Action runs and calls `_authService`
    
6. `ActionResult` (`Ok(result)`) is returned
    
7. Framework serializes `LoginResponse` to JSON
    
8. Writes `200 OK` with body
    

---

## 🧩 What Is NOT Part of the MVC Pipeline?

- **Middleware** (e.g., `UseAuthentication`, `UseCors`) runs **before** MVC.
    
- Minimal APIs have their **own pipeline**, and don’t support filters by default.
    
- `IResult` executes directly on `HttpContext`, **bypassing** the MVC pipeline.
    

---

# 🛠 Pro Tip: Customizing the Pipeline

You can:

- Register **filters globally** (`services.AddControllers(options => { })`)
    
- Create custom model binders
    
- Write `ActionResult` subclasses
    
- Use `IApplicationModelConvention` for custom controller logic
    

---

## ✅ Summary Table

|Stage|Responsibility|
|---|---|
|Middleware|Cross-cutting concerns (auth, logging)|
|Routing|Map request to controller & action|
|Model Binding|Populate method parameters|
|Model Validation|Validate input models|
|Action Execution|Execute controller method|
|Filters|Inject behavior before/after actions|
|Result Execution|Write response to client|
