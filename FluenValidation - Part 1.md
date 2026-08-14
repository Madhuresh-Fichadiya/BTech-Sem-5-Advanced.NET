# What Problems Do We Face Without API input restrictions?

Suppose we have the following DTO:
```csharp
public class StudentDTO
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    public string EnrollmentNumber { get; set; }
    public int DepartmentId { get; set; }
    public string? DepartmentName { get; set; }
}
```

<img width="1741" height="715" alt="FV-Image1" src="https://github.com/user-attachments/assets/e4bb8484-07db-4b94-a816-5330dff923b8" />


<img width="1738" height="716" alt="FV-Image2" src="https://github.com/user-attachments/assets/67d1731b-0e89-45e6-b292-8e73d3e84ab1" />


<img width="1736" height="713" alt="FV-Image3" src="https://github.com/user-attachments/assets/ea28dacf-5f97-4b58-a283-265a9de2a149" />

# What if we want to apply few restrictions on input in API?
For ex:
- Student Name: Compulsory, Not contain whitespaces, Maximum Length is 100 characters
- EnrollmentNo: Compulsory, Not contain whitespaces, exact 11 characters
- DepartmentId: Must be > 0

## To implement such kind of restrictions on API There are two techniques:
1. Data Annotations - Less Flexible
2. Fluent Validation - More Flexible and Widely used in API development.

---

# Steps for Implementation 

### Step 1: Install Required Packages
- FluentValidation
- FluentValidation.DependencyInjectionExtensions

### Step 2: Create Validator Class
Create a Folder Named Validators, Inside Validator Folder create all validator classes (Validator/StudentValidator.cs)
```csharp
//Validator/StudentValidator.cs
using FluentValidation;

public class StudentValidator : AbstractValidator<StudentDTO>
{
    public StudentValidator()
    {
        RuleFor(x => x.Name)

            // Name is mandatory
            .NotEmpty()
            .WithMessage("Student Name is required")

            // Name should not contain only whitespace characters
            .Must(name => !string.IsNullOrWhiteSpace(name))
            .WithMessage("Student Name cannot be empty or whitespace")

            //Name cannot contain numbers
            .Must(name => !name.Any(char.IsDigit))
            .WithMessage("Student Name cannot contain Digits")

            // Name length must not exceed 100 characters
            .MaximumLength(100)
            .WithMessage("Student Name cannot exceed 100 characters");

        RuleFor(x => x.EnrollmentNumber)

            // Enrollment Number is mandatory
            .NotEmpty()
            .WithMessage("Enrollment Number is required")

            // Enrollment Number should not contain only whitespace characters
            .Must(enrollment => !string.IsNullOrWhiteSpace(enrollment))
            .WithMessage("Enrollment Number cannot be empty or whitespace")

            // Enrollment Number must contain exactly 11 characters
            .Length(11)
            .WithMessage("Enrollment Number must be exactly 11 characters");

        RuleFor(x => x.DepartmentId)

            // DepartmentId must be greater than 0
            .GreaterThan(0)
            .WithMessage("Please select a valid Department");
    }
}
```
### Step 3: Register validator in Program.cs

```csharp
using FluentValidation;

var builder = WebApplication.CreateBuilder(args);

...
//This line automatically scans all validators:
builder.Services.AddValidatorsFromAssemblyContaining<StudentValidator>();

var app = builder.Build();

...
app.MapControllers();

app.Run();
```

### Step 4: Call Validator Manually in Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class StudentController : ControllerBase
{
    private readonly IValidator<StudentDTO> _validator;

    public StudentController(IValidator<StudentDTO> validator)
    {
        _validator = validator;
    }

    [HttpPost]
    public async Task<IActionResult> Create(StudentDTO dto)
    {
        var result = await _validator.ValidateAsync(dto);

        if (!result.IsValid)
        {
            return BadRequest(result.Errors.Select(x=>x.ErrorMessage));
        }
        //Logic to Add Record in DB

        return Ok("Student Created Successfully");
    }
}
```
---

<img width="1744" height="718" alt="FV-Image4" src="https://github.com/user-attachments/assets/6e782fe4-ac67-41aa-9a6e-010d19b04e4c" />

---
# Common Validators
| Validator Name             | Task of Validator                                           | Code Snippet                                                | Example Output                                          |
| -------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------- |
| **WithName()**             | Changes property name displayed in validation message       | `RuleFor(x => x.Name).NotEmpty().WithName("Student Name");` | `"Student Name must not be empty."`                     |
| **NotNull()**              | Ensures value is not `null`                                 | `RuleFor(x => x.Name).NotNull();`                           | `"Name must not be empty."`                             |
| **NotEmpty()**             | Ensures value is not `null`, empty string, or default value | `RuleFor(x => x.Name).NotEmpty();`                          | `"Name must not be empty."`                             |
| **Equal()**                | Value must match specified value                            | `RuleFor(x => x.DepartmentId).Equal(1);`                    | `"Department Id must be equal to '1'."`                 |
| **NotEqual()**             | Value must not match specified value                        | `RuleFor(x => x.DepartmentId).NotEqual(0);`                 | `"Department Id should not be equal to '0'."`           |
| **Length()**               | Value must have exact or specified range of characters      | `RuleFor(x => x.EnrollmentNumber).Length(11);`              | `"Enrollment Number must be 11 characters in length."`  |
| **MinimumLength()**        | Value must contain at least specified characters            | `RuleFor(x => x.Name).MinimumLength(3);`                    | `"Name must be at least 3 characters."`                 |
| **MaximumLength()**        | Value cannot exceed specified characters                    | `RuleFor(x => x.Name).MaximumLength(100);`                  | `"Name must be 100 characters or fewer."`               |
| **LessThan()**             | Value must be less than specified value                     | `RuleFor(x => x.StudentId).LessThan(1000);`                 | `"Student Id must be less than '1000'."`                |
| **LessThanOrEqualTo()**    | Value must be less than or equal to specified value         | `RuleFor(x => x.StudentId).LessThanOrEqualTo(1000);`        | `"Student Id must be less than or equal to '1000'."`    |
| **GreaterThan()**          | Value must be greater than specified value                  | `RuleFor(x => x.DepartmentId).GreaterThan(0);`              | `"Department Id must be greater than '0'."`             |
| **GreaterThanOrEqualTo()** | Value must be greater than or equal to specified value      | `RuleFor(x => x.DepartmentId).GreaterThanOrEqualTo(1);`     | `"Department Id must be greater than or equal to '1'."` |

