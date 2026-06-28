# Model Classes and Migrations

![Image](https://images.openai.com/static-rsc-4/YxHYh3DASAzrr11vAmTvhu7brcN3Vf6ojICJVlf2DadOwqOMkAg411od2SvBUeGgyu3IrQqbLOx0a87JBox6OrHfO6drQfy26GVISJ5Ry3Lp89lWrk-IuRi_kAJgYMOVNfM8pdm0jtaatLM79kSopcbRrlzadqRMHmRq4wOCMuO8x853aFTUq6uk3BoqRTpz?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/hFJFC4dFnO2sV9Uc4IQa1CLK0pMzbwhjgUO7M-TbXr92onKKmHf2vtPyhAwOXVlYrqCkWBY_6puKsdkVZ_CZrLY2PehMZVoLn-Ebv10xSE1p7tlEa48CS2JtvXYATOW4L8UqWmzBvL4t3_1UMeCb2J3N5QSXP3902sK7tgeyLxYo-2Z3atQqQ6Fe0L3e5zDZ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ykewsy_epnMCp1zT9-FRKV1769wuGTDjRrJRwCyHVt9EZ2-8NAWYUWGddcW9JlQqtRLJ2036h84fpb0ZBcWrYt7ZaoE7AX9ITGJ55G-YcdUpd-7R-sy6rz_MTrbZV_77-zqkoYVlyy347E6csEVkS_UuutF9dfV97EVcig4f6ZVOCmraXwwPDbsuf5rLsdXZ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/-FBVVU2B-yGJnlZpygZ3qPH831lD2aiNdlLVn-2apwKgfaJcExshTn0ESN31H25kOtMeqLv25o7rEAEpmlxkaRLphUQad1nY_HMXAyhgHz4vBld0tYKz1i9tAS85mqqpdjnCw-4Pm5qTwcqvXjgHbWI7fNLsvczJnx09lYo2uh4Jz5_s_qFGAP2wpKsmq5qh?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/8kgbbaFCcbl4wNMF_v6u64p6gOwRIXK2FrfwlhE50iHMi8MoocsFO1FRogPBaxJU4MfwEO8cEvpxTiVNpkTVbFwjqePyt-tQeLW0qAKRRp346E6__wPLpM448QnonAJZqJObrEkKhcB_jWOUih3UxjMCmxT0wLs5WtS3hOK8V8RFs08Az7qcADzpWr-Ln1HQ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/-hKfng8RlCfUUED3ZlteL0-G95WksXzi0Dpj88zNNAoaZ-_Og84hEZWEE0L7mtsWrVWkYK03D2RfBam5V-T8Yd0VKZlRMkJ3flnWu-Ih5x94sKp2d5sQuinFaY4ntT7Do66Z0Y1Csp9GCXIwl7mwvrTlFJomEPu5rPzASAq5lFdSmOWy0vYkQxCtAEPShljj?purpose=fullsize)

If you're teaching students about **Migrations in .NET Core Web API**, the best approach is:

1. Start with the problem.
2. Explain why migrations exist.
3. Introduce EF Core Migration concepts.
4. Demonstrate implementation.
5. Cover real-world scenarios and best practices.

---

# Suggested Teaching Flow (2-3 Hours Session)

```text
Databases Without Migration
        ↓
Problem Statement
        ↓
What is EF Core Migration?
        ↓
Why Migration is Required?
        ↓
Migration Architecture
        ↓
Create First Migration
        ↓
Update Database
        ↓
Add/Modify/Delete Columns
        ↓
Relationships
        ↓
Rollback Migration
        ↓
Production Scenarios
        ↓
Best Practices
```

---

# Part 1: Start with a Problem Statement

## Scenario 1

Suppose we have a Student table.

### Initial Table

| Id | Name  |
| -- | ----- |
| 1  | John  |
| 2  | David |

Now business says:

> Add Email column.

Without migration:

```sql
ALTER TABLE Students
ADD Email NVARCHAR(100)
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

Or simply:

> Migration is version control for your database.

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
V3 -> Add Phone
V4 -> Add Department table
```

---

# Why Migrations are Required

## 1. Database Versioning

Track every database change.

---

## 2. Team Collaboration

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

---

## 3. Production Deployment

No need to manually write SQL.

---

## 4. Database Synchronization

Development
Testing
Staging
Production

All environments remain consistent.

---

## 5. Rollback Support

If something goes wrong:

```bash
dotnet ef database update PreviousMigration
```

---

# Migration Architecture

![Image](https://images.openai.com/static-rsc-4/ykewsy_epnMCp1zT9-FRKV1769wuGTDjRrJRwCyHVt9EZ2-8NAWYUWGddcW9JlQqtRLJ2036h84fpb0ZBcWrYt7ZaoE7AX9ITGJ55G-YcdUpd-7R-sy6rz_MTrbZV_77-zqkoYVlyy347E6csEVkS_UuutF9dfV97EVcig4f6ZVOCmraXwwPDbsuf5rLsdXZ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/w7so3m_HnSjScGFkrdIxbknVj7o0pSwrjJGDuHgYDYlHZB8xl5dt-nnlwvp9ffBhA27TkiQvpvZWM_XXKKeQx7W-PkOGF0_oQaoDLtvsdXCYHkOu2dT1AyVWCaSIOdCtCD-GCdpjtASi6pEDct9gWR6lzWOIODlsQyrM1QSvwMwlaTGpVene_MlJ1Q2CRQUC?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/pPWBdiDHoemAJ5XNiQCx_imyV26-Wd_086PRIwtAjFAys9xmDQ9-Nr8zURkOBHwerZFJrkGZxG3Eb_aRAZOxSx55Bo_8zPCsRS0Ge4x2LWeoU687xAqUbbX_Pe6gFXHNHbR-IUS6oUMqCQ-LlQQktHJCyE7zxVwWxoFg7LX7Q5ZHcy-6rL7Kd0djx0UzAq8C?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/-hKfng8RlCfUUED3ZlteL0-G95WksXzi0Dpj88zNNAoaZ-_Og84hEZWEE0L7mtsWrVWkYK03D2RfBam5V-T8Yd0VKZlRMkJ3flnWu-Ih5x94sKp2d5sQuinFaY4ntT7Do66Z0Y1Csp9GCXIwl7mwvrTlFJomEPu5rPzASAq5lFdSmOWy0vYkQxCtAEPShljj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/o6SOkN-PkdJQi9ouD7PvjtTG_PP2FpGdxOLHELVb6hJMROgcr-QPne0Ypooi2xbogCQ_wamJxDDDTkuP0si0gclK2TVS4Q5TJ7l5Z3eNeyFQK5OazlGTfh7R30XK8elU1qrim88fLm7c5IYqK9Ip4LnOCE39a0X3VRPYFKmuTlRZUkMXgAVFH15cQye2Eort?purpose=fullsize)

```text
Entity Classes
      ↓
DbContext
      ↓
Add-Migration
      ↓
Migration Files
      ↓
Update-Database
      ↓
SQL Generated
      ↓
Database Updated
```

---

# Part 2: Understanding Code First Approach

## Entity

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

---

## DbContext

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Student> Students { get; set; }
}
```

---

# Install Packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

---

# Configure Connection String

```json
"ConnectionStrings": {
  "DefaultConnection":
  "Server=.;Database=StudentDB;Trusted_Connection=True;TrustServerCertificate=True"
}
```

---

# Register DbContext

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"));
});
```

---

# Part 3: Create First Migration

## Command

```bash
dotnet ef migrations add InitialCreate
```

or

```powershell
Add-Migration InitialCreate
```

---

# What Happens?

EF compares:

```text
Current Model
vs
Database Snapshot
```

Generates migration file.

---

# Migration Folder Structure

```text
Migrations
│
├── 20250628120000_InitialCreate.cs
├── 20250628120000_InitialCreate.Designer.cs
└── ApplicationDbContextModelSnapshot.cs
```

---

# Generated Migration

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
}
```

---

# Down Method

```csharp
protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropTable(
        name: "Students");
}
```

---

# Explain Carefully

## Up()

Applied when migration runs.

## Down()

Applied when migration rolls back.

---

# Update Database

```bash
dotnet ef database update
```

or

```powershell
Update-Database
```

---

# Internal Process

```text
Migration File
      ↓
EF Generates SQL
      ↓
Executes SQL
      ↓
Updates Database
      ↓
Stores Migration History
```

---

# Migration History Table

```text
__EFMigrationsHistory
```

Stores:

```text
MigrationId
ProductVersion
```

---

# Part 4: Different Scenarios

# Scenario 1: Add Column

Old Model

```csharp
public string Name { get; set; }
```

New Model

```csharp
public string Email { get; set; }
```

Create migration:

```bash
dotnet ef migrations add AddEmail
```

Generated SQL:

```sql
ALTER TABLE Students
ADD Email nvarchar(max)
```

---

# Scenario 2: Remove Column

```csharp
Remove PhoneNumber property
```

Migration:

```bash
dotnet ef migrations add RemovePhone
```

Generated SQL:

```sql
ALTER TABLE Students
DROP COLUMN PhoneNumber
```

---

# Scenario 3: Rename Column

Wrong approach:

```csharp
Name -> FullName
```

EF thinks:

```text
Drop Name
Create FullName
```

Data loss!

Correct:

```csharp
migrationBuilder.RenameColumn(
    name: "Name",
    table: "Students",
    newName: "FullName");
```

---

# Scenario 4: Add New Table

```csharp
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

Migration:

```bash
dotnet ef migrations add AddDepartment
```

---

# Scenario 5: One-to-Many Relationship

```csharp
public class Department
{
    public int Id { get; set; }

    public ICollection<Student> Students { get; set; }
}
```

```csharp
public class Student
{
    public int Id { get; set; }

    public int DepartmentId { get; set; }

    public Department Department { get; set; }
}
```

Migration creates:

```text
Departments Table
Students Table
Foreign Key
```

---

# Scenario 6: Many-to-Many

```csharp
public class Student
{
    public ICollection<Course> Courses { get; set; }
}

public class Course
{
    public ICollection<Student> Students { get; set; }
}
```

EF automatically creates:

```text
StudentCourse
```

junction table.

---

# Scenario 7: Change Data Type

```csharp
int Age
```

to

```csharp
string Age
```

Migration generated:

```sql
ALTER TABLE
ALTER COLUMN
```

Sometimes it fails if data conversion is impossible.

Always backup production data.

---

# Scenario 8: Seed Data

```csharp
modelBuilder.Entity<Department>().HasData(
new Department
{
    Id = 1,
    Name = "IT"
});
```

Migration inserts:

```sql
INSERT INTO Departments
```

---

# Scenario 9: Rollback Migration

Remove latest migration:

```bash
dotnet ef migrations remove
```

Rollback database:

```bash
dotnet ef database update InitialCreate
```

---

# Scenario 10: Generate SQL Script

Very important in production.

```bash
dotnet ef migrations script
```

Specific migrations:

```bash
dotnet ef migrations script InitialCreate AddEmail
```

---

# Production Flow

```text
Developer
    ↓
Create Migration
    ↓
Commit Code
    ↓
Generate SQL Script
    ↓
DBA Review
    ↓
Production Database
```

---

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
