# Model Classes and Migrations
# Part 1: Start with a Problem Statement

## Scenario: Suppose we have a Student table.

### Initial Table

| Id | Name  |
| -- | ----- |
| 1  | John  |
| 2  | David |

Now business says:
> Add Email column.

Without migration:

```sql
ALTER TABLE Students ADD Email NVARCHAR(100)
```

Then:
> Add Phone Number.

Again:
```sql
ALTER TABLE Students
ADD PhoneNumber NVARCHAR(20)
```

Then:
> Remove Name and split into FirstName and LastName.

Again more SQL.
---

### Problems

* Manual SQL scripts
* Different developers have different databases
* No version control
* Difficult deployment
* Errors in production
* No history of schema changes
---

# Introduce Migration

## Definition

> Migration is a mechanism provided by Entity Framework Core to keep the database schema synchronized with the application's model classes.
---

# Real Life Analogy

Think of migrations like:

```text
Git → Source Code Versioning
Migration → Database Versioning
```

Every schema change becomes a version.

Example:

```text
V1 -> Create Students table
V2 -> Add Email
V3 -> PhoneNumber
V4 -> Remove Name and split into FirstName and LastName
```
---

# Why Migrations are Required

### 1. Database Versioning
Track every database change.

### 2. Team Collaboration
Developer A:
```text
Add Employee table
```
Developer B:
```text
git pull
update-database
```
Database becomes same.

### 3. Production Deployment
No need to manually write SQL.

### 4. Database Synchronization
Development, Testing, Production
All environments remain consistent.

### 5. Rollback Support

If something goes wrong then revert to required migration

# Migration Architecture

![Image](https://images.openai.com/static-rsc-4/ykewsy_epnMCp1zT9-FRKV1769wuGTDjRrJRwCyHVt9EZ2-8NAWYUWGddcW9JlQqtRLJ2036h84fpb0ZBcWrYt7ZaoE7AX9ITGJ55G-YcdUpd-7R-sy6rz_MTrbZV_77-zqkoYVlyy347E6csEVkS_UuutF9dfV97EVcig4f6ZVOCmraXwwPDbsuf5rLsdXZ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/w7so3m_HnSjScGFkrdIxbknVj7o0pSwrjJGDuHgYDYlHZB8xl5dt-nnlwvp9ffBhA27TkiQvpvZWM_XXKKeQx7W-PkOGF0_oQaoDLtvsdXCYHkOu2dT1AyVWCaSIOdCtCD-GCdpjtASi6pEDct9gWR6lzWOIODlsQyrM1QSvwMwlaTGpVene_MlJ1Q2CRQUC?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/pPWBdiDHoemAJ5XNiQCx_imyV26-Wd_086PRIwtAjFAys9xmDQ9-Nr8zURkOBHwerZFJrkGZxG3Eb_aRAZOxSx55Bo_8zPCsRS0Ge4x2LWeoU687xAqUbbX_Pe6gFXHNHbR-IUS6oUMqCQ-LlQQktHJCyE7zxVwWxoFg7LX7Q5ZHcy-6rL7Kd0djx0UzAq8C?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/-hKfng8RlCfUUED3ZlteL0-G95WksXzi0Dpj88zNNAoaZ-_Og84hEZWEE0L7mtsWrVWkYK03D2RfBam5V-T8Yd0VKZlRMkJ3flnWu-Ih5x94sKp2d5sQuinFaY4ntT7Do66Z0Y1Csp9GCXIwl7mwvrTlFJomEPu5rPzASAq5lFdSmOWy0vYkQxCtAEPShljj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/o6SOkN-PkdJQi9ouD7PvjtTG_PP2FpGdxOLHELVb6hJMROgcr-QPne0Ypooi2xbogCQ_wamJxDDDTkuP0si0gclK2TVS4Q5TJ7l5Z3eNeyFQK5OazlGTfh7R30XK8elU1qrim88fLm7c5IYqK9Ip4LnOCE39a0X3VRPYFKmuTlRZUkMXgAVFH15cQye2Eort?purpose=fullsize)

# Part 2: Understanding Code First Approach

### Entity

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```
### DbContext

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options): base(options)
    {
    }

    public DbSet<Student> Students { get; set; }
}
```

#### Install Packages

```bash
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools
Microsoft.EntityFrameworkCore.Design
```
### Configure Connection String

```json
"ConnectionStrings": {
  "DefaultConnection":   "Server=.;Database=StudentDB;Trusted_Connection=True;TrustServerCertificate=True"
}
```
# Register DbContext

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"));
});
```
Here is the breakdown of exactly what each part does:

#### 1. `builder.Services`
> This accesses the application's **Dependency Injection (DI) container**. By adding services here, you are telling the application how to create and provide specific objects whenever different parts of your app (like Controllers or Services) ask for them.

#### 2. `.AddDbContext<ApplicationDbContext>(options => { ... })`
> * **`AddDbContext`**: This tells ASP.NET Core to register your database context (`ApplicationDbContext`) into the DI container. This allows you to easily inject your database context into your controllers using constructor injection.
> * **`<ApplicationDbContext>`**: This is your custom class that inherits from EF Core's `DbContext`. It acts as the bridge between your C# code and your actual database.
> * **`options => { ... }`**: This is an action delegate (a lambda expression) used to configure the database options, such as which database provider to use and where to find it.

#### 3. `options.UseSqlServer(...)`

> This specifies that you are using **Microsoft SQL Server** as your relational database management system. It tells EF Core to translate your C# LINQ queries into SQL Server-compatible SQL syntax.

#### 4. `builder.Configuration.GetConnectionString("DefaultConnection")`

> * **`builder.Configuration`**: This looks into your application's configuration sources (most commonly the `appsettings.json` file).
> * **`GetConnectionString("DefaultConnection")`**: This is a helper method that specifically looks for a connection string named `"DefaultConnection"` inside the `ConnectionStrings` section of your configuration.

---

### Summary: How it connects to your `appsettings.json`

For this code to work without throwing an error, you typically have a matching block in your `appsettings.json` file that looks like this:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER;Database=YOUR_DB;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}

```
---

# Create First Migration

### Command
```powershell
Add-Migration InitialCreate
```

### What Happens?

EF compares:
```text
Current Model
vs
Database Snapshot
```
Generates migration file.

## Migration Folder Structure

```text
Migrations
│
├── 20250628120000_InitialCreate.cs
├── 20250628120000_InitialCreate.Designer.cs
└── ApplicationDbContextModelSnapshot.cs
```
## Generated Migration
> `Up()` Applied when migration runs.
> `Down()` Applied when migration rolls back.

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "Students",
        columns: table => new
        {
            Id = table.Column<int>()
                .Annotation("SqlServer:Identity", "1, 1"),
            Name = table.Column<string>(nullable: false)
        });

 protected override void Down(MigrationBuilder migrationBuilder)
 {
     migrationBuilder.DropTable(
        name: "Students");
 }
```
# Update Database
or

```powershell
Update-Database
```
# Internal Process

```text
Migration File --> EF Generates SQL --> Executes SQL --> Updates Database --> Stores Migration History
```


# Migration History Table
```text
__EFMigrationsHistory
```

Stores:
```text
MigrationId
ProductVersion
```
## Different Scenarios
### Models
```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
}

```

### DbContext Configuration

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

    public DbSet<Student> Students { get; set; }
    public DbSet<Department> Departments { get; set; }
}

```

---

## 🚀 PowerShell Migration Scenarios

### Scenario 1: Add Column

* **C# Change:** Add `public string Email { get; set; }` to `Student`.
* **PowerShell:**
```powershell
Add-Migration AddEmail
Update-Database

```



### Scenario 2: Remove Column

* **C# Change:** Delete `public string PhoneNumber { get; set; }` from `Student`.
* **PowerShell:**
```powershell
Add-Migration RemovePhone
Update-Database

```



### Scenario 3: Rename Column (Data-Safe)

* **C# Change:** Rename `Name` to `FullName` in `Student`.
* **Action Required:** Open the generated migration file and modify the `Up` method to prevent data loss:
```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.RenameColumn(
        name: "Name",
        table: "Students",
        newName: "FullName");
}

```


* **PowerShell:**
```powershell
Add-Migration RenameStudentName
Update-Database

```



### Scenario 4: Add New Table

* **C# Change:** Add a new class `public class Course { public int Id { get; set; } }` and add `public DbSet<Course> Courses { get; set; }` to `DbContext`.
* **PowerShell:**
```powershell
Add-Migration AddCourseTable
Update-Database

```



### Scenario 5: One-to-Many Relationship

* **C# Change:**
```csharp
// In Department.cs
public ICollection<Student> Students { get; set; }

// In Student.cs
public int DepartmentId { get; set; }
public Department Department { get; set; }

```


* **PowerShell:**
```powershell
Add-Migration AddDepartmentRelationship
Update-Database

```



### Scenario 6: Many-to-Many Relationship

* **C# Change:**
```csharp
// In Student.cs
public ICollection<Course> Courses { get; set; }

// In Course.cs
public ICollection<Student> Students { get; set; }

```


* **PowerShell:** *(EF automatically creates the hidden junction table `CourseStudent`)*
```powershell
Add-Migration AddCourseStudentRelationship
Update-Database

```



### Scenario 7: Change Data Type

* **C# Change:** Modify `public int Age { get; set; }` to `public string Age { get; set; }`.
* **PowerShell:**
```powershell
Add-Migration ChangeAgeDataType
Update-Database

```


> ⚠️ **Warning:** SQL Server will throw an error during `Update-Database` if existing data cannot be implicitly converted.



### Scenario 8: Seed Data

* **C# Change:** Add to `OnModelCreating` in `DbContext`:
```csharp
modelBuilder.Entity<Department>().HasData(new Department { Id = 1, Name = "IT" });

```


* **PowerShell:**
```powershell
Add-Migration SeedInitialDepartments
Update-Database

```



### Scenario 9: Rollback Changes

* **Remove latest migration** *(Only works if it has not been pushed to the database yet)*:
```powershell
Remove-Migration

```


* **Revert Database state** *(Downgrades the database schema back to a specific checkpoint)*:
```powershell
Update-Database -TargetMigration InitialCreate

```



### Scenario 10: Generate SQL Scripts (For Production)

* **Generate full script** from scratch to the latest state:
```powershell
Script-Migration

```


* **Generate a differential script** between two specific migrations:
```powershell
Script-Migration -From InitialCreate -To AddEmail


# Important Commands

| Command          | Purpose               |
| ---------------- | --------------------- |
| Add-Migration    | Create migration      |
| Update-Database  | Apply migration       |
| Remove-Migration | Delete last migration |
| Script-Migration | Generate SQL          |
| Get-Migration    | List migrations       |

---

# Interview Questions

### What is Migration?

Database version control system.

---

### Why is it needed?

To synchronize model and database.

---

### Difference between EnsureCreated and Migration?

| EnsureCreated         | Migration          |
| --------------------- | ------------------ |
| Creates database once | Versioning support |
| No schema tracking    | Schema tracking    |
| No updates            | Supports updates   |
| Not for production    | Production ready   |

---

### What is ModelSnapshot?

Current state of database model used for comparison.

---

### What is `__EFMigrationsHistory`?

Stores applied migrations.

---

# Best Practices

### ✔ One migration per feature

```text
AddStudentModule
AddDepartmentModule
```

---

### ✔ Never edit old migrations in production.

---

### ✔ Review generated migration code.

---

### ✔ Generate SQL script before production deployment.

---

### ✔ Use meaningful migration names.

Bad:

```text
Migration1
Migration2
```

Good:

```text
AddEmployeeTable
AddDepartmentForeignKey
AddEmailToStudents
```

---

# Complete Teaching Flow for Students

```text
1. What problem are we solving?
2. What is migration?
3. Why do we need migration?
4. Code First Architecture
5. Install EF Packages
6. Create DbContext
7. Add First Migration
8. Update Database
9. Understand Migration Files
10. Understand Snapshot
11. Modify Entities
12. Add New Migration
13. Relationships
14. Rollback
15. Production Deployment
16. Best Practices
17. Interview Questions
```

---

# One-Line Summary for Students

> **Migration in Entity Framework Core is a database versioning mechanism that automatically manages schema changes and keeps the database synchronized with your application's model classes.**
