# LINQ Lab-2: Ordering, Grouping, Joins, Aggregation

> Note: Reuse the `students` list defined in `LINQ Lab-1.md` sampleData for these examples.

https://github.com/sejalgupta001/Building-REST-APIs-using-ASP.NET-Core/blob/main/Lab11%20%7C%20LINQ%20Part%201.md

---

## 1. Aggregation Operators

```csharp
var students = new List<Student>
{
    new Student { Id = 1, Name = "Amit", Marks = 85 },
    new Student { Id = 2, Name = "Neha", Marks = 92 },
    new Student { Id = 3, Name = "Ravi", Marks = 78 },
    new Student { Id = 4, Name = "Priya", Marks = 95 }
};
```

| Operator      | Description                                                                        | Return Type         | Example                           | Output |
| ------------- | ---------------------------------------------------------------------------------- | ------------------- | --------------------------------- |--------|
| `Count()`     | Returns the total number of elements.                                              | `int`               | `students.Count()`                | 4      |
| `Sum()`       | Calculates the sum of numeric values.                                              | Numeric Type        | `students.Sum(s => s.Marks)`      | 350    |
| `Average()`   | Calculates the average of numeric values.                                          | Numeric Type        | `students.Average(s => s.Marks)`  | 87.5   |
| `Min()`       | Returns the smallest value.                                                        | Same as source type | `students.Min(s => s.Marks)`      | 78     |
| `Max()`       | Returns the largest value.                                                         | Same as source type | `students.Max(s => s.Marks)`      | 95     |
| `Aggregate()` | Applies a custom accumulation function to the collection.                          | Custom              | `students.Select(s => s.Name).Aggregate((x,y) => x + ", " + y)` | Amit, Neha, Ravi, Priya   |


1. Total Number of Students
```csharp
var totalStudents = students.Count();

Console.WriteLine($"Total Students: {totalStudents}");
```
2. Average CPI of All Students
```csharp
var avgCPI = students.Average(s => s.CPI);
Console.WriteLine($"Average CPI: {avgCPI}");
```
3. Highest CPI
```csharp
var highestCPI = students.Max(s => s.CPI);
Console.WriteLine($"Highest CPI: {highestCPI}");
```
4. Lowest CPI
```csharp
var lowestCPI = students.Min(s => s.CPI);
Console.WriteLine($"Lowest CPI: {lowestCPI}");
```
5. Total Semester Sum
```csharp
var totalSem = students.Sum(s => s.Sem);
Console.WriteLine($"Total Semesters Sum: {totalSem}");
```

6. Total Students Having CPI Above 8
```csharp
var topperCount = students.Count(s => s.CPI > 8);

Console.WriteLine($"Students with CPI > 8 : {topperCount}");
```

7. Total Courses Enrolled By All Students
```csharp
var totalCourses = students.Sum(s => s.Courses.Count);

Console.WriteLine($"Total Course Registrations: {totalCourses}");


## 2. Group By Operator

GroupBy() is used to group records based on a common key. It is similar to SQL's GROUP BY clause.

Here we want to Group data based on Department, So In SQL:
```sql
SELECT DepartmentId, COUNT(*)
FROM Students
GROUP BY DepartmentId;
```

Same query for LINQ
```csharp
var result = students.GroupBy(s => s.DepartmentId);
```

### Why Use GroupBy?

Suppose we have a list of students:
```csharp
var students = new List<Student>
{
    new Student { Id = 1, Name = "Amit", Department = "MCA", Marks = 85 },
    new Student { Id = 2, Name = "Neha", Department = "MCA", Marks = 92 },
    new Student { Id = 3, Name = "Ravi", Department = "MBA", Marks = 78 },
    new Student { Id = 4, Name = "Priya", Department = "MBA", Marks = 88 },
    new Student { Id = 5, Name = "Kiran", Department = "BCA", Marks = 95 }
};
```

We want to group data based on Department we can Group it using Group By, for ex:
```javascript
MCA
 ├─ Amit
 └─ Neha

MBA
 ├─ Ravi
 └─ Priya

BCA
 └─ Kiran
```
Basic Group By:
```csharp
var groups = students.GroupBy(s => s.Department);

foreach (var group in groups) // To Get Individual Key (Department Details)
{
    Console.WriteLine($"Department: {group.Key}");

    foreach (var student in group) //To Iterate over Elements of that Particular group key
    {
        Console.WriteLine(student.Name);
    }
}

```

Output:

<img width="212" height="201" alt="image" src="https://github.com/user-attachments/assets/73553644-89a7-4115-8c3d-2db223a92ad9" />

| Property | Description                      |
| -------- | -------------------------------- |
| Key      | Group value (MCA, MBA, BCA)      |
| Elements | Students belonging to that group |
