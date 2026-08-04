# LINQ Lab-2: Ordering, Grouping, Joins, Aggregation

> Note: Reuse the `students` list defined in `LINQ Lab-1.md` sampleData for these examples.

https://github.com/sejalgupta001/Building-REST-APIs-using-ASP.NET-Core/blob/main/Lab11%20%7C%20LINQ%20Part%201.md

---

## 1. Group By Operator

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
Baic Group By:
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
