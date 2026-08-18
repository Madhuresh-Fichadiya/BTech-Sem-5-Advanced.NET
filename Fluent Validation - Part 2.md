# Fluent Validation Part 2

## Consider below schema for validation
```cshap
public class SingleDTO
{
    public int StudentId { get; set; }

    public string EnrollmentNo { get; set; } = string.Empty;

    public string StudentName { get; set; } = string.Empty;

    public int Age { get; set; }

    public Gender Gender { get; set; }

    public int DepartmentId { get; set; }

    public int Semester { get; set; }

    public bool IsHosteller { get; set; }

    public string? HostelName { get; set; }

    public decimal Fees { get; set; }

    public List<string> Courses { get; set; } = new();
}
```
Also, We do have Enum as below

```csharp
public enum Gender
{
    Male = 1,
    Female = 2,
    Other = 3
}
```

--- 

## 1. IsInEnum() - Used when the property is an enum.
```cshap
RuleFor(x => x.Gender)
    .IsInEnum();
```

## 2. When() - When() applies a rule only when a condition is true.
```cshap
RuleFor(x => x.HostelName)
    .NotEmpty()
    .When(x => x.IsHosteller);
```

## 3. Unless() - Unless() is basically the opposite condition.
```cshap
RuleFor(x => x.HostelName)
    .Empty()
    .Unless(x => x.IsHosteller);
```

## 4. Must() - Must() is extremely useful for custom business rules.
```csharp
RuleFor(x => x.Fees)
    .Must(fees => fees >= 10000 && fees <= 200000)
    .WithMessage("Fees must be between ₹10,000 and ₹2,00,000.");
```

## 5. RuleForEach() - Used when the DTO contains a collection.
```csharp
RuleForEach(x => x.Courses)
    .NotEmpty()
    .MaximumLength(50);
```

## 6. SetValidator() - SetValidator() becomes especially useful when each collection item is a complex object. mainly useful for nested/complex DTO validation.
```csharp
public class CourseDTO
{
    public int CourseId { get; set; }

    public string CourseName { get; set; } = string.Empty;

    public int Credits { get; set; }
}

//For example, if we changed:
public List<string> Courses

//To,
public List<CourseDTO> Courses

//Create: a CourseDTO validator
public class CourseDTOValidator : AbstractValidator<CourseDTO>
{
    public CourseDTOValidator()
    {
        RuleFor(x => x.CourseId)
            .GreaterThan(0);

        RuleFor(x => x.CourseName)
            .NotEmpty()
            .MaximumLength(100);

        RuleFor(x => x.Credits)
            .InclusiveBetween(1, 6);
    }
}

//Then, In StudentDTO Validator,

RuleForEach(x => x.Courses)
    .SetValidator(new CourseDTOValidator());
```

## 7. DependentRules() - DependentRules() is useful when one validation should execute only if the previous validation succeeds.
Is StudentName empty?
       │
       ├── YES → Stop dependent validation
       │
       └── NO
            ↓
      Check numbers

```csharp
RuleFor(x => x.StudentName)
    .NotEmpty()
    .DependentRules(() =>
    {
        RuleFor(x => x.StudentName)
            .Must(name => !name.Any(char.IsDigit))
            .WithMessage("Student name cannot contain numbers.");
    });
```

## 8. MustAsync() — Database Validation

Suppose we want to check: Does this enrollment number already exist?

```csharp
RuleFor(x => x.EnrollmentNo)
    .MustAsync(async (dto, enrollmentNo, cancellation) =>
    {
        return !await _context.Students
            .AnyAsync(x =>
                x.EnrollmentNo == enrollmentNo &&
                x.StudentId != dto.StudentId,
                cancellation);
    })
    .WithMessage("Enrollment number already exists.");
```

## 9. Data Exists Validation

Suppose the request contains DepartmentId 10, 
Before inserting the student, we need to verify does Department 10 actually exist?

```csharp
RuleFor(x => x.DepartmentId)
    .MustAsync(async (departmentId, cancellation) =>
    {
        return await _context.Departments
            .AnyAsync(x => x.DepartmentId == departmentId,
                cancellation);
    })
    .WithMessage("Selected department does not exist.");
```
