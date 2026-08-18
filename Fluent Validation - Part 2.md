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
```cshap

```


