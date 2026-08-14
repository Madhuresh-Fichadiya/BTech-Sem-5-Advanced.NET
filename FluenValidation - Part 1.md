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
 
