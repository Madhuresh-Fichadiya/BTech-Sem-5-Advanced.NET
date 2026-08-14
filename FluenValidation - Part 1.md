# What Problems Do We Face Without Fluent Validation?

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
