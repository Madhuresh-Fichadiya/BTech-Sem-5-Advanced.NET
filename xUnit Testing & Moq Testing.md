
## Part 1: xUnit Testing Fundamentals
**xUnit.net** is the leading open-source unit testing tool for .NET.

- Testing means checking that your code works correctly before it goes live.
- Unit Testing = testing the smallest part of code (a method, service, or controller action) alone, without other parts.
- Why? Finds bugs early, keeps code quality high, makes refactoring safe.

### Key Attributes & Concepts

* **`[Fact]`**: Identifies a test method that requires no parameters and tests a fixed condition.
* **`[Theory]`**: Identifies a test method that accepts arguments to test the same logic across multiple data variations.
* **`[InlineData]`**: Passes primitive data arguments to a `[Theory]` method.
* **AAA Pattern (Arrange, Act, Assert)**:
  1. **Arrange**: Initialize target objects, setup input data.
  2. **Act**: Execute the method under test.
  3. **Assert**: Verify that outcomes match expected criteria using `Assert` class methods.
---
- When testing a controller or service with no external dependencies, xUnit acts directly upon the target instance:

```mermaid
sequenceDiagram
    autonumber
    participant Runner as xUnit Test Runner
    participant Controller as OrderController (Target)
    
    Runner->>Controller: 1. Instantiate OrderController()
    Runner->>Controller: 2. Call CalculateTax(100.00m)
    Note over Controller: Executes tax calculation logic
    Controller-->>Runner: 3. Return OkObjectResult(10.00m)
    Note over Runner: 4. Assert response type and calculated value
```

---

### Production Code Example: Controller Without Dependencies

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyWebAPI.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    [HttpGet("calculate-tax")]
    public IActionResult CalculateTax(decimal orderTotal)
    {
        if (orderTotal <= 0)
        {
            return BadRequest("Order total must be greater than zero.");
        }

        // Standard 10% tax calculation
        decimal tax = orderTotal * 0.10m;
        return Ok(tax);
    }
}
```

---
### xUnit Test Suite

```csharp
using Microsoft.AspNetCore.Mvc;
using MyWebAPI.Controllers;
using Xunit;

namespace MyWebAPI.Tests;

public class OrderControllerTests
{
    [Fact]
    public void CalculateTax_WhenOrderTotalIsPositive_ReturnsOkWithCalculatedTax()
    {
        // 1. ARRANGE
        var controller = new OrderController();
        decimal inputTotal = 100.00m;
        decimal expectedTax = 10.00m;

        // 2. ACT
        var result = controller.CalculateTax(inputTotal);

        // 3. ASSERT
        var okResult = Assert.IsType<OkObjectResult>(result); 
        var actualTax = Assert.IsType<decimal>(okResult.Value); 
        Assert.Equal(expectedTax, actualTax);
    }

    [Theory]
    [InlineData(0)]
    [InlineData(-10.50)]
    [InlineData(-100)]
    public void CalculateTax_WhenOrderTotalIsZeroOrNegative_ReturnsBadRequest(decimal invalidTotal)
    {
        // ARRANGE
        var controller = new OrderController();

        // ACT
        var result = controller.CalculateTax(invalidTotal);

        // ASSERT
        var badRequestResult = Assert.IsType<BadRequestObjectResult>(result);
        Assert.Equal("Order total must be greater than zero.", badRequestResult.Value);
    }
}
```
---
## 3. Part 2: Moq Testing (Dependency Mocking)

**Moq** allows developers to synthesize fake implementations of interfaces to isolate unit tests from external infrastructure like database adapters, external HTTP services, or file systems.

### Core Moq Concepts

* **`new Mock<IInterface>()`**: Constructs a mock container wrapper for the target interface.
* **`.Setup(...)`**: Establishes execution expectations and configuration rules for a given call signature.
* **`.ReturnsAsync(...)` / `.Returns(...)`**: Configures return payload values.
* **`It.IsAny<T>()`**: Serves as a parameter wildcard matcher.
* **`.Verify(...)`**: Asserts that an interface method was invoked according to specified rules (e.g., `Times.Once`).

---

### Production Code Example: Controller With Dependencies

```csharp
// Models & Interfaces
namespace MyWebAPI.Models;

public class UserProfile
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

public interface IUserRepository
{
    Task<UserProfile?> GetByIdAsync(int id);
    Task<bool> CreateUserAsync(UserProfile user);
}

// Controller Implementation
using Microsoft.AspNetCore.Mvc;
using MyWebAPI.Models;

namespace MyWebAPI.Controllers;

[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IUserRepository _userRepository;

    public UserController(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetUserById(int id)
    {
        var user = await _userRepository.GetByIdAsync(id);

        if (user == null)
        {
            return NotFound($"User with ID {id} was not found.");
        }

        return Ok(user);
    }
}
```

---
---
