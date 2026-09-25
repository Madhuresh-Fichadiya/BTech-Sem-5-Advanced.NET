### Repository interface
```csharp
public interface IStudentRepository
{
    Task<Student> CreateAsync(Student student);

    Task<Student?> GetByIdAsync(int id);

    Task DeleteAsync(int id);
}
```

## 1) Service Layer Code — Create & Delete

```csharp
using StudentWebAPI.Models;
using StudentWebAPI.Repositories;

namespace StudentWebAPI.Services;

public class StudentService
{
    private readonly IStudentRepository _repository;

    public StudentService(IStudentRepository repository)
    {
        _repository = repository;
    }

    // POST / Create Student
    public async Task<Student> CreateStudent(Student student)
    {
        var result = await _repository.CreateAsync(student);
        return result;
    }


    // DELETE Student
    public async Task<bool> DeleteStudent(int id)
    {
        var student = await _repository.GetByIdAsync(id);

        if (student == null)
        {
            return false;
        }

        await _repository.DeleteAsync(id);

        return true;
    }
}
```

---

# 2) Moq Testing Code

```csharp
using Moq;
using StudentWebAPI.Models;
using StudentWebAPI.Repositories;
using StudentWebAPI.Services;
using Xunit;

namespace StudentWebAPI.Tests;

public class StudentServiceTests
{
    [Fact]
    public async Task CreateStudent_WhenValidStudent_ReturnsCreatedStudent()
    {
        // Arrange

        var student = new Student
        {
            Id = 1,
            Name = "Asha",
            EnrollmentNumber = "23CS001",
            DepartmentId = 10
        };

        // Create Mock Repository
        var mockRepository = new Mock<IStudentRepository>();

        // Tell Mock: When CreateAsync() is called, return the student
        mockRepository
            .Setup(x => x.CreateAsync(student))
            .ReturnsAsync(student);

        // Inject Mock Repository into Service
        var service = new StudentService(
            mockRepository.Object
        );


        // Act

        var result = await service.CreateStudent(student);


        // Assert

        Assert.NotNull(result);
        Assert.Equal(1, result.Id);
        Assert.Equal("Asha", result.Name);

        // Verify that CreateAsync() was called exactly once
        mockRepository.Verify(
            x => x.CreateAsync(student),
            Times.Once
        );
    }


    // ============================================================
    // CREATE STUDENT - TEST CASE 2
    // Verify repository CreateAsync() is called once
    // ============================================================

    [Fact]
    public async Task CreateStudent_ShouldCallRepositoryOnce()
    {
        // Arrange

        var student = new Student
        {
            Id = 2,
            Name = "Ravi",
            EnrollmentNumber = "23CS002",
            DepartmentId = 10
        };

        var mockRepository = new Mock<IStudentRepository>();

        mockRepository
            .Setup(x => x.CreateAsync(student))
            .ReturnsAsync(student);

        var service = new StudentService(
            mockRepository.Object
        );


        // Act

        await service.CreateStudent(student);


        // Assert

        // Make sure repository was called exactly once
        mockRepository.Verify(
            x => x.CreateAsync(student),
            Times.Once
        );
    }

    [Fact]
    public async Task DeleteStudent_WhenStudentExists_ReturnsTrue()
    {
        // Arrange

        var student = new Student
        {
            Id = 1,
            Name = "Asha",
            EnrollmentNumber = "23CS001",
            DepartmentId = 10
        };

        var mockRepository = new Mock<IStudentRepository>();

        // Tell Mock:When GetByIdAsync(1) is called, return the student
        mockRepository
            .Setup(x => x.GetByIdAsync(1))
            .ReturnsAsync(student);

        // DeleteAsync() doesn't return a value,
        // so return a completed Task
        mockRepository
            .Setup(x => x.DeleteAsync(1))
            .Returns(Task.CompletedTask);

        var service = new StudentService(
            mockRepository.Object
        );


        // Act

        var result = await service.DeleteStudent(1);


        // Assert

        Assert.True(result);

        // Verify that student was searched
        mockRepository.Verify(
            x => x.GetByIdAsync(1),
            Times.Once
        );

        // Verify that DeleteAsync() was called
        mockRepository.Verify(
            x => x.DeleteAsync(1),
            Times.Once
        );
    }

    [Fact]
    public async Task DeleteStudent_WhenStudentDoesNotExist_ReturnsFalse()
    {
        // Arrange

        var mockRepository = new Mock<IStudentRepository>();

        // Tell Mock: Student with ID 1 does not exist
        mockRepository
            .Setup(x => x.GetByIdAsync(1))
            .ReturnsAsync((Student?)null);

        var service = new StudentService(
            mockRepository.Object
        );


        // Act

        var result = await service.DeleteStudent(1);


        // Assert

        Assert.False(result);

        // Student should be checked once
        mockRepository.Verify(
            x => x.GetByIdAsync(1),
            Times.Once
        );

        mockRepository.Verify(
            x => x.DeleteAsync(1),
            Times.Never
        );
    }

    [Fact]
    public async Task DeleteStudent_WhenStudentExists_CallsDeleteOnce()
    {
        // Arrange

        var student = new Student
        {
            Id = 5,
            Name = "Meera",
            EnrollmentNumber = "23CS005",
            DepartmentId = 20
        };

        var mockRepository = new Mock<IStudentRepository>();

        mockRepository
            .Setup(x => x.GetByIdAsync(5))
            .ReturnsAsync(student);

        mockRepository
            .Setup(x => x.DeleteAsync(5))
            .Returns(Task.CompletedTask);

        var service = new StudentService(
            mockRepository.Object
        );


        // Act

        await service.DeleteStudent(5);

        // Assert

        // Verify that DeleteAsync(5) was called exactly once
        mockRepository.Verify(
            x => x.DeleteAsync(5),
            Times.Once
        );
    }
}
```

**Key Moq concepts demonstrated:** `Mock<T>` → `Setup()` → `ReturnsAsync()` → `mock.Object` → `Verify()` → `Times.Once` / `Times.Never`.
