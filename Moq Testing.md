### What is Moq Testing?

**Moq** (pronounced "Mock-doc" or "Mock") is a popular mocking library for .NET used during unit testing.

When you test a component (like an API Controller), that component often depends on external classes or services (like database repositories, third-party APIs, or email services). If your unit test hits a real database or calls a real external service, it is **not a unit test**—it becomes an integration test, which is slow, unreliable, and prone to side effects.

**Moq** solves this problem by allowing you to create fake (mocked) implementations of interfaces in memory. You can program these mock objects to:

1. **Return predefined data** when specific methods are called (`Setup`).
2. **Throw specific exceptions** to test error handling logic.
3. **Verify** that specific methods were actually invoked by your code (`Verify`).

---

### Key Concepts in Moq

* `Mock<T>`: Creates a mock instance of an interface `T`.
* `_mock.Setup(x => x.Method()).ReturnsAsync(value)`: Configures the mock to return a specific `value` when `Method()` is called.
* `_mock.Object`: Extracts the underlying mocked object instance so you can pass it into your controller's constructor.
* `It.IsAny<T>()`: A parameter wildcard matching helper (e.g., "accept any integer or string argument").
* `_mock.Verify(x => x.Method(), Times.Once)`: Asserts that a method on the mock interface was called a specific number of times.

---

### Simple Implementation Example: Controller + Service + Repository

Here is a clean end-to-end example using a standard 3-layer architecture (`Controller` $\rightarrow$ `Service` $\rightarrow$ `Repository`).

#### 1. Models & Interfaces

```csharp
namespace App.Core;

public record UserDto(int Id, string Name);

public interface IUserRepository
{
    Task<UserDto?> GetByIdAsync(int id);
}

public interface IUserService
{
    Task<UserDto?> GetUserAsync(int id);
}

```

#### 2. Service Layer

```csharp
namespace App.Services;

using App.Core;

public class UserService : IUserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }

    public async Task<UserDto?> GetUserAsync(int id)
    {
        if (id <= 0) return null;
        return await _repository.GetByIdAsync(id);
    }
}

```

#### 3. Controller Layer (Unit Under Test)

```csharp
namespace App.Controllers;

using App.Core;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;

    public UserController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id)
    {
        var user = await _userService.GetUserAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        return Ok(user);
    }
}

```

---

### 4. xUnit + Moq Unit Tests

Below are unit tests for the **Controller Layer** (mocking the `IUserService`) and the **Service Layer** (mocking the `IUserRepository`).

```csharp
namespace App.Tests;

using App.Controllers;
using App.Core;
using App.Services;
using Microsoft.AspNetCore.Mvc;
using Moq;
using Xunit;

public class UserControllerTests
{
    private readonly Mock<IUserService> _mockUserService;
    private readonly UserController _controller;

    public UserControllerTests()
    {
        // 1. Create Mock instance of IUserService
        _mockUserService = new Mock<IUserService>();

        // 2. Inject Mock object into Controller
        _controller = new UserController(_mockUserService.Object);
    }

    [Fact]
    public async Task GetUser_WhenUserExists_ReturnsOkResultWithUser()
    {
        // ARRANGE
        int userId = 1;
        var fakeUser = new UserDto(userId, "John Doe");

        // Program Moq: When GetUserAsync(1) is called, return fakeUser
        _mockUserService
            .Setup(service => service.GetUserAsync(userId))
            .ReturnsAsync(fakeUser);

        // ACT
        var result = await _controller.GetUser(userId);

        // ASSERT
        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnedUser = Assert.IsType<UserDto>(okResult.Value);
        Assert.Equal("John Doe", returnedUser.Name);

        // VERIFY: Ensure service method was called exactly once
        _mockUserService.Verify(service => service.GetUserAsync(userId), Times.Once);
    }

    [Fact]
    public async Task GetUser_WhenUserDoesNotExist_ReturnsNotFound()
    {
        // ARRANGE
        int userId = 99;

        // Program Moq: Return null for missing user
        _mockUserService
            .Setup(service => service.GetUserAsync(It.IsAny<int>()))
            .ReturnsAsync((UserDto?)null);

        // ACT
        var result = await _controller.GetUser(userId);

        // ASSERT
        Assert.IsType<NotFoundResult>(result);
    }
}

public class UserServiceTests
{
    [Fact]
    public async Task GetUserAsync_WhenInvalidId_ReturnsNullWithoutCallingRepo()
    {
        // ARRANGE
        var mockRepo = new Mock<IUserRepository>();
        var service = new UserService(mockRepo.Object);

        // ACT
        var result = await service.GetUserAsync(-1);

        // ASSERT
        Assert.Null(result);

        // VERIFY: Verify repo was NEVER called because validation failed early
        mockRepo.Verify(repo => repo.GetByIdAsync(It.IsAny<int>()), Times.Never);
    }
}

```
