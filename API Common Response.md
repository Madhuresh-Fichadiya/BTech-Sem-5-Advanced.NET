# API Common Response

---

## Problem Without Common Response:

Consider below API Endpoints:

**API 1 - Get Student**

<img width="1922" height="690" alt="GetAll" src="https://github.com/user-attachments/assets/964356f1-87f3-4e83-a00f-7a8feac0ffa7" />


**API 2 - Add Student**

<img width="1922" height="690" alt="Post" src="https://github.com/user-attachments/assets/69762e89-6cb3-4a86-a1e2-15e92ccd2c9f" />


**API 3 - Update Student**

<img width="1922" height="690" alt="Update" src="https://github.com/user-attachments/assets/48e7414a-9ee4-47a3-879e-b29b8a1fbede" />


**API 4 - Delete Student**

<img width="1922" height="690" alt="Delete" src="https://github.com/user-attachments/assets/9c06284e-2680-4fb0-89e3-acd7fdeb26de" />

---

## Issues

**1. Frontend Complexity**

Frontend developers must write different logic for every API.
```jquery
if(response.studentId)
{
   // Handle Student
}

if(response.status)
{
   // Handle Add Student
}

if(response.isDeleted)
{
   // Handle Delete Student
}
```
As APIs grow, this becomes messy.

**2. Inconsistent Error Handling**

Some APIs return:
```jquery
{
  "message":"Not Found"
}
```

Others return:
```json
{
  "error":"Not Found"
}
```

Others:
```jquery
{
  "status":"Failed"
}
```

Frontend has no single way to handle errors.

**3. Difficult Mobile Integration**

Android, iOS, Angular, React developers expect a predictable structure.

Without a standard contract:
- More bugs
- More validation code
- More testing effort

**4. Difficult Logging and Monitoring**

Suppose you want to log all API failures.

Without a common structure:
```jquery
API1 -> error
API2 -> message
API3 -> exception
```

No consistency.

---

## Common Response Structure

```jquery
{
  "success": true,
  "message": "Data Retrieved Successfully",
  "data": {},
  "errors": null
}
```

---

## Implementation Example

Create a Common Response Class as below:

```csharp
public class ApiResponse<T>
{
    // Indicates whether request was successful
    public bool Success { get; set; }

    // Success or failure message
    public string Message { get; set; }

    // Actual response data
    public T? Data { get; set; }

    // Validation or error messages
    public List<string>? Errors { get; set; }
}
```

---

**API 1: Get All Endpoint**
Implement Common Response Class in GetAll() as below:

```csharp
[HttpGet]
public async Task<IActionResult> GetAll()
{
    var result = students.Select(x => new StudentDTO()
    {
        StudentId = x.StudentId,
        Name = x.Name,
        EnrollmentNumber = x.EnrollmentNumber,
        DepartmentId = x.DepartmentId,
        DepartmentName = x.Department.DepartmentName
    }).OrderBy(x => x.DepartmentName).ThenByDescending(x => x.Name);

    return Ok(new ApiResponse<List<StudentDTO>>
    {
        Success = true,
        Message = "Students Retrieved Successfully",
        Data = result,
    });
}
```

Which gives response as below

<img width="1918" height="705" alt="GetAll" src="https://github.com/user-attachments/assets/48ef1a7e-0cd9-4b35-9004-cf61e0e12902" />

---

**API 2: Post Endpoint**

```csharp
[HttpPost]
public async Task<IActionResult> AddStudent(StudentDTO student)
{
    try
    {
        var std = new Student()
        {
            DepartmentId = student.DepartmentId,
            EnrollmentNumber = student.EnrollmentNumber,
            Name = student.Name
        };
        _context.Students.Add(std);
        await _context.SaveChangesAsync();
        return Ok(new ApiResponse<Student>
        {
            Success = true,
            Message = "Student Added Successfully",
            Data = std
        });
    }
    catch (Exception ex)
    {
        return BadRequest(new ApiResponse<object>
        {
            Success = false,
            Message = "Error occurred while adding student",
            Errors = new List<string> { ex.Message }
        });
    }
}
```

<img width="1568" height="725" alt="Post" src="https://github.com/user-attachments/assets/b87cd062-f5a7-45ab-981f-5dfe9e59ec6e" />

---

**API 3: Update Endpoint**

```csharp
[HttpPut]
public IActionResult UpdateStudent(StudentDTO student)
{
    try
    {
        var s = _context.Students.Find(student.StudentId);
        if (s == null)
        {
            return NotFound(new ApiResponse<object>
            {
                Success = false,
                Message = "Student Not Found",
                Errors = new List<string> { $"No student found with Id {student.StudentId}" }
            });
        }
        s.Name = student.Name;
        s.EnrollmentNumber = student.EnrollmentNumber;
        _context.SaveChanges();
        return Ok(new ApiResponse<Student>
        {
            Success = true,
            Message = "Student Updated Successfully",
            Data = s
        });
    }
    catch (Exception ex)
    {
        return BadRequest(new ApiResponse<object>
        {
            Success = false,
            Message = "Error occurred while updating student",
            Errors = new List<string> { ex.Message }
        });
    }
}
```

Which gives response as below
<img width="1562" height="736" alt="Update" src="https://github.com/user-attachments/assets/708a47f7-c61b-401c-8913-910db932f789" />


if no record found then,

<img width="1575" height="722" alt="Updaet 2" src="https://github.com/user-attachments/assets/11a323a0-7f95-4eac-8fe8-3ce9205d277f" />

---

**API 4: Delete Endpoint**
```csharp
[HttpDelete]
public async Task<IActionResult> DeleteById(int StudentId)
{
    try
    {
        var student = _context.Students.Find(StudentId);
        if (student == null)
        {
            return NotFound(new ApiResponse<object>
            {
                Success = false,
                Message = "Student Not Found",
            });
        }
        _context.Students.Remove(student);

        await _context.SaveChangesAsync();

        return Ok(new ApiResponse<object>
        {
            Success = true,
            Message = "Student Deleted Successfully",
            Data = student
        });
    }
    catch (Exception ex)
    {
        return BadRequest(new ApiResponse<object>
        {
            Success = false,
            Message = "Error occurred while deleting student",
            Errors = new List<string> { ex.Message }
        });
    }
}
```

Which gives response as below

<img width="1556" height="729" alt="Delete" src="https://github.com/user-attachments/assets/f94e75cd-bda5-4a3b-8b6d-eef858ab7d54" />


If no record found then,

<img width="1573" height="737" alt="Delete 2" src="https://github.com/user-attachments/assets/5044d056-ba4c-4fd9-bb70-039f1cc3c8cd" />
