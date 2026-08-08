# LINQ Joins

## Types of Joins
- Inner Join
- Left Join

Consider below model classes
```csharp
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }

    public string EnrollmentNumber { get; set; }

    [ForeignKey(nameof(DepartmentId))]
    public int DepartmentId { get; set; }

    //Navigation
    public Department Department { get; set; }

    public DateTime Created { get; set; }

    public DateTime LastUpdated { get; set; }
}

public class Department
{
    public int DepartmentId { get; set; }

    public string DepartmentName { get; set; }
    public string? DepartmentCode { get; set; }

    public ICollection<Student> Students { get; set; }
}
```

## Inner Join

When we want to do apply Inner Join we could write:

```csharp
var result = await _context.Students

    .Join(

        _context.Departments,               // Right table

        student => student.DepartmentId,    // Student FK

        department => department.DepartmentId, // Department PK

        (student, department) => new        // Result
        {
            StudentId = student.StudentId,
            StudentName = student.Name,
            EnrollmentNumber = student.EnrollmentNumber,

            DepartmentId = department.DepartmentId,
            DepartmentName = department.DepartmentName
        })

    .ToListAsync();
```

Which will generate Below SQL:
```sql
SELECT
    s.StudentId,
    s.Name,
    s.EnrollmentNumber,
    d.DepartmentId,
    d.DepartmentName
FROM Students s
INNER JOIN Departments d
ON s.DepartmentId = d.DepartmentId
```
---

## Now, Observe our Model Classes, Do we need to do apply join Manually?

## The answer is No, Because we already have navigation properties.

For example, Student already has:
```csharp
public Department Department { get; set; }
```

### We can simply use `Include()` method, instead of writing Joins manually as per below example:
```csharp
[HttpGet]
public async Task<IActionResult> GetStudents()
{
    var students = await _context.Students
    .Include(student => student.Department)
    .Select(student => new
    {
        StudentId = student.StudentId,
        StudentName = student.Name,
        EnrollmentNumber = student.EnrollmentNumber,
        DepartmentName = student.Department.DepartmentName
    })
    .ToListAsync();

    return Ok(students);
}
```

**Why?**
- EF Core automatically generates the JOIN.
- Write less code and it's easier to maintain.

---

# Left Outer Join
Suppose someday DepartmentId becomes nullable:
```csharp
public int? DepartmentId { get; set; }
```

Then: 
```csharp
[HttpGet]
public async Task<IActionResult> GetStudents()
{
    var result = await _context.Students

    .GroupJoin(
      _context.Departments,               // Right table
      student => student.DepartmentId,    // Student FK
      department => department.DepartmentId, // Department PK
      (student, departmentGroup) => new
      {
          Student = student,
          Departments = departmentGroup
      })
      .SelectMany(
        x => x.Departments.DefaultIfEmpty(),
        (x, department) => new        // Result
        {
            StudentName = x.Student.Name,
            DepartmentName = department != null? department.DepartmentName : "No Department"
        })
      .ToListAsync();
}
```
