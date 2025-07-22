## 📘 FluentValidation Integration in ASP.NET Core Web API

---

### ✅ Step 1: Install Required NuGet Packages

In the terminal, run:

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
```

> ✅ Do **not** install `FluentValidation.AspNetCore` (it's deprecated).

---

### ✅ Step 2: Create Your Model

```csharp
// Models/Country.cs
namespace Address_API.Models
{
    public class Country
    {
        public int CountryId { get; set; }
        public string CountryName { get; set; } = string.Empty;
        public string CountryCode { get; set; } = string.Empty;
        public DateTime CreatedDate { get; set; }
        public DateTime ModifiedDate { get; set; }
    }
}
```

---

### ✅ Step 3: Create a Validator Class

```csharp
// ValidationClass/CountryValidator.cs
using FluentValidation;
using Address_API.Models;

namespace Address_API.ValidationClass
{
    public class CountryValidator : AbstractValidator<Country>
    {
        public CountryValidator()
        {
            RuleFor(c => c.CountryName)
                .NotNull().WithMessage("Country name must not be empty.")
                .Length(3, 20).WithMessage("Country name must be between 3 and 20 characters.")
                .Matches("^[A-Za-z ]*$").WithMessage("Country name must contain only letters and spaces.");

            RuleFor(c => c.CountryCode)
                .NotNull().WithMessage("Country code must not be empty.")
                .MaximumLength(4).WithMessage("Country code must not exceed 4 characters.")
                .Matches("^[A-Za-z]{2,4}$").WithMessage("Country code must contain only letters and be between 2 to 4 characters.");
        }
    }
}
```

---

### ✅ Step 4: Register Validator in `Program.cs`

```csharp
using FluentValidation;
using Address_API.Models;
using Address_API.ValidationClass;
using System.Reflection;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Register all validators from this assembly
builder.Services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
builder.Services.AddScoped<IValidator<Country>, CountryValidator>();

builder.Services.AddDbContext<AddressDemoDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

### ✅ Step 5: Update the Controller for Manual Validation

```csharp
// Controllers/CountryController.cs
using Microsoft.AspNetCore.Mvc;
using Address_API.Models;
using FluentValidation;

namespace Address_API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class CountryController : ControllerBase
    {
        private readonly AddressDemoDbContext _context;
        private readonly IValidator<Country> _validator;

        public CountryController(AddressDemoDbContext context, IValidator<Country> validator)
        {
            _context = context;
            _validator = validator;
        }

        [HttpPost]
        public async Task<IActionResult> PostCountry([FromBody] Country country)
        {
            var validationResult = await _validator.ValidateAsync(country);

            if (!validationResult.IsValid)
            {
                return BadRequest(validationResult.Errors.Select(e => new
                {
                    Property = e.PropertyName,
                    Error = e.ErrorMessage
                }));
            }

            _context.Countries.Add(country);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetCountry), new { id = country.CountryId }, country);
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<Country>> GetCountry(int id)
        {
            var country = await _context.Countries.FindAsync(id);
            if (country == null)
                return NotFound();

            return country;
        }
    }
}
```

---

### ✅ Step 6: Test with Invalid JSON

Try posting this in Swagger or Postman:

```json
{
  "countryName": "x",
  "countryCode": "123456",
  "createdDate": "2025-07-21T00:00:00",
  "modifiedDate": "2025-07-21T00:00:00"
}
```

✅ You’ll receive:

```json
[
  {
    "property": "CountryName",
    "error": "Country name must be between 3 and 20 characters."
  },
  {
    "property": "CountryCode",
    "error": "Country code must not exceed 4 characters."
  },
  {
    "property": "CountryCode",
    "error": "Country code must contain only letters and be between 2 to 4 characters."
  }
]
```

