
## Part 1: xUnit Testing Fundamentals
**xUnit.net** is the leading open-source unit testing tool for .NET.

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
