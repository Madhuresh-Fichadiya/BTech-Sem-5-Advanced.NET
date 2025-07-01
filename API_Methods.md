Here’s a detailed step-by-step explanation of how the `CityController` and the Entity Framework Core (EF Core) setup work together, starting from **database schema** to **controller implementation**.

---

## 🔹 Step-by-Step Explanation of EF Core Setup

---

### **Step 1: Database Tables (SQL Schema)**

Three tables are defined:

#### 1. **Country Table**

```sql
CREATE TABLE Country (
    CountryID INT PRIMARY KEY IDENTITY(1,1),
    CountryName NVARCHAR(100) NOT NULL,
    CountryCode NVARCHAR(10) NOT NULL,
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL
);
```

#### 2. **State Table**

```sql
CREATE TABLE State (
    StateID INT PRIMARY KEY IDENTITY(1,1),
    CountryID INT NOT NULL,
    StateName NVARCHAR(100) NOT NULL,
    StateCode NVARCHAR(10),
    ...
    FOREIGN KEY (CountryID) REFERENCES Country(CountryID)
);
```

#### 3. **City Table**

```sql
CREATE TABLE City (
    CityID INT PRIMARY KEY IDENTITY(1,1),
    StateID INT NOT NULL,
    CountryID INT NOT NULL,
    CityName NVARCHAR(100) NOT NULL,
    ...
    FOREIGN KEY (StateID) REFERENCES State(StateID),
    FOREIGN KEY (CountryID) REFERENCES Country(CountryID)
);
```

---

### **Step 2: Scaffold the Database Using EF Core**

Run this command in the terminal or **Package Manager Console**:

```bash
Scaffold-DbContext "server=SEJAL\SQLEXPRESS; database=AddressDemoDB; trusted_connection=true; TrustServerCertificate=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Force
```

This:

* Connects to your database.
* Reads the table schemas.
* Generates model classes (`City.cs`, `State.cs`, `Country.cs`) in the `Models` folder.
* Also creates a `DbContext` class named `AddressDemoDbContext`.

---

### **Step 3: Auto-Generated Entity Classes**

Each table gets a C# class:

```csharp
public class City
{
    public int CityId { get; set; }
    public string CityName { get; set; }
    public string CityCode { get; set; }
    public int StateId { get; set; }
    public int CountryId { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? ModifiedDate { get; set; }

    public virtual Country Country { get; set; }
    public virtual State State { get; set; }
}
```

EF Core also builds relationships using `virtual` navigation properties.

---

### **Step 4: Register DbContext in `Program.cs`**

```csharp
services.AddDbContext<AddressDemoDbContext>(options =>
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
```

Make sure `DefaultConnection` is defined in `appsettings.json`.

---

## 🔹 Step-by-Step Explanation of Each Controller Method

---
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Address_API.Models;

[Route("api/[controller]")]
[ApiController]
public class CityController : ControllerBase
{
    private readonly AddressDemoDbContext _context;

    public CityController(AddressDemoDbContext context)
    {
        _context = context;
    }

    // Get all cities
    [HttpGet]
    public async Task<ActionResult<IEnumerable<City>>> GetAll()
    {
        return await _context.Cities
            .Include(c => c.State)
            .Include(c => c.Country)
            .ToListAsync();
    }

    // Get top 10 cities
    [HttpGet("Top10")]
    public async Task<ActionResult<IEnumerable<City>>> GetTop10()
    {
        return await _context.Cities
            .Include(c => c.State)
            .Include(c => c.Country)
            .Take(10)
            .ToListAsync();
    }

    // Get single city by ID
    [HttpGet("{id}")]
    public async Task<ActionResult<City>> GetById(int id)
    {
        var city = await _context.Cities
            .Include(c => c.State)
            .Include(c => c.Country)
            .FirstOrDefaultAsync(c => c.CityId == id);

        return city == null ? NotFound() : Ok(city);
    }

    // Get cities by filters (CountryId and StateId)
    [HttpGet("filter")]
    public async Task<ActionResult<IEnumerable<City>>> Filter([FromQuery] int? countryId, [FromQuery] int? stateId)
    {
        var query = _context.Cities.Include(c => c.State).Include(c => c.Country).AsQueryable();

        if (countryId.HasValue)
            query = query.Where(c => c.CountryId == countryId);

        if (stateId.HasValue)
            query = query.Where(c => c.StateId == stateId);

        return await query.ToListAsync();
    }

    // Create city
    [HttpPost]
    public async Task<IActionResult> Create(City city)
    {
        city.CreatedDate = DateTime.Now;
        _context.Cities.Add(city);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetById), new { id = city.CityId }, city);
    }

    // Update city
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, City city)
    {
        if (id != city.CityId) return BadRequest();

        city.ModifiedDate = DateTime.Now;
        _context.Entry(city).State = EntityState.Modified;
        await _context.SaveChangesAsync();
        return NoContent();
    }

    // Delete city
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var city = await _context.Cities.FindAsync(id);
        if (city == null) return NotFound();

        _context.Cities.Remove(city);
        await _context.SaveChangesAsync();
        return NoContent();
    }

    // Get all countries (for dropdown)
    [HttpGet("dropdown/countries")]
    public async Task<ActionResult<IEnumerable<object>>> GetCountries()
    {
        return await _context.Countries
            .Select(c => new { c.CountryId, c.CountryName })
            .ToListAsync();
    }

    // Get states by country (for cascading)
    [HttpGet("dropdown/states/{countryId}")]
    public async Task<ActionResult<IEnumerable<object>>> GetStatesByCountry(int countryId)
    {
        return await _context.States
            .Where(s => s.CountryId == countryId)
            .Select(s => new { s.StateId, s.StateName })
            .ToListAsync();
    }
}
