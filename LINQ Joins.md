# LINQ Joins

## Types of Joins
- Inner Join
- Left Outer Join
- Right Outer Join

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
[HttpGet]
public async Task<IActionResult> GetStudents()
{
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
}
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
     var students = await _context.Students
    .LeftJoin(

    _context.Departments,               // Right source

    student => student.DepartmentId,    // Left key

    department => department.DepartmentId, // Right key

    (student, department) => new        // Result selector
    {
        StudentName = student.Name,

        DepartmentName =
            department != null
                ? department.DepartmentName
                : "No Department"
    });
}
```
**We can also perform `Left Join` using `Include()`, As per below sample:**
Suppose Model class has Optional Departments Navigation Properties:
```csharp
public int? DepartmentId { get; set; }

public Department? Department { get; set; }
```
Then EF Core Generates LEFT JOIN Automatically:
```csharp
[HttpGet]
public async Task<IActionResult> GetStudents()
{
var students = await _context.Students
    .Include(s => s.Department)
    .ToListAsync();
}
```

EF Core typically generates a LEFT JOIN:
```sql
SELECT
    s.StudentId,
    s.Name,
    s.DepartmentId,
    d.DepartmentId,
    d.DepartmentName
FROM Students s
LEFT JOIN Departments d
    ON s.DepartmentId = d.DepartmentId
```

---

## Can Include() be used instead of LeftJoin() and RightJoin()?

- Include() is not a join operator; it is an eager-loading operator.
- Include() often results in SQL that behaves like a LEFT JOIN.
- Therefore, LEFT JOIN-like behavior can be achieved using Include().
- Include() does not directly provide RIGHT JOIN semantics.
- To achieve the same result as a RIGHT JOIN, query from the opposite entity (swap the root table) and use Include() or navigation properties.


# Right Outer Join

```csharp
[HttpGet]
public async Task<IActionResult> GetStudents()
{
    var result = _context.Students

    .RightJoin(
    _context.Departments,
    student => student.DepartmentId,
    department => department.DepartmentId,
    (student, department) => new
    {
        DepartmentName = department.DepartmentName,

        StudentName = 
            student != null
                ? student.Name
                : "No Student"
    });

        return result != null ? Ok(result) : NotFound();
    }
}
```
Genereatd SQL:
```sql
SELECT [d].[DepartmentName], CASE
    WHEN [s].[StudentId] IS NOT NULL THEN [s].[Name]
    ELSE N'No Student'
END AS [StudentName]
FROM [Students] AS [s]
RIGHT JOIN [Departments] AS [d] ON [s].[DepartmentId] = [d].[DepartmentId]
```

---

# When to use Include()?
Use Include() when you need the entire related entity.
```csharp
var students = await _context.Students
    .Include(s => s.Department)
    .ToListAsync();
```

We Get:
```cshap
student.Department.DepartmentName
student.Department.DepartmentCode
...
...
```
and all other Department properties.
Generated SQL:
```sql
SELECT [s].[StudentId], [s].[Created], [s].[DepartmentId], [s].[EnrollmentNumber], [s].[LastUpdated], [s].[Name],
    [d].[DepartmentId], [d].[DepartmentCode], [d].[DepartmentName]
FROM [Students] AS [s]
INNER JOIN [Departments] AS [d]
ON [s].[DepartmentId] = [d].[DepartmentId]
```
---

# When to use Include() with Projection (Select() Method)?
If we need a few fields, prefer projection (Select) as below. Because Include() loads all Student columns + all Department columns. So It becomes Less Efficient.

```csharp
var students = await _context.Students
    .Select(s => new
    {
        s.StudentId,
        s.Name,
        DepartmentName = s.Department != null
            ? s.Department.DepartmentName
            : "No Department"
    })
    .ToListAsync();
```
Generate SQL For Above Query:
```sql
SELECT [s].[StudentId], [s].[Name], [d].[DepartmentName]
FROM [Students] AS [s]
INNER JOIN [Departments] AS [d]
ON [s].[DepartmentId] = [d].[DepartmentId]
```
