# 🔐 Role-Based JWT Authentication in ASP.NET Core Web API
---
<img width="1100" height="507" alt="image" src="https://github.com/user-attachments/assets/ac61f173-487d-4989-a601-8584c07466cd" />
----

## Add following packages

- Microsoft.AspNetCore.Authentication.JwtBearer
- Microsoft.IdentityModel.Tokens
- System.IdentityModel.Tokens.Jwt

## ✅ 1️⃣ Configure `appsettings.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "server=SEJAL\\SQLEXPRESS; database=AddressDemoDB; trusted_connection=true; TrustServerCertificate=True;"
  },
  "Jwt": {
    "Key": "ThisIsASecretKeyForJwtTokenGeneration",
    "Issuer": "https://localhost:7093",
    "Audience": "*",
    "TokenExpiryMinutes": 60
  }
}
```

👉 **`TokenExpiryMinutes`** controls how long the token is valid (here, 60 minutes).

---

## ✅ 2️⃣ Configure JWT in `Program.cs`

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddDbContext<AddressDemoDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

builder.Services.AddAuthorization();

// ✅ Add Swagger JWT Support
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Enter 'Bearer {your JWT token}'"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new string[] {}
        }
    });
});

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.UseSwagger();
app.UseSwaggerUI();

app.MapControllers();
app.Run();
```

---

## ✅ 3️⃣ Update `UserController` with Role-Based Login

```csharp
using Address_API.Models;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

namespace Address_API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UserController : ControllerBase
    {
        private readonly AddressDemoDbContext _context;
        private readonly IConfiguration _configuration;

        public UserController(AddressDemoDbContext context, IConfiguration configuration)
        {
            _context = context;
            _configuration = configuration;
        }

        // 🔑 Generate Token with Role & Expiry from appsettings.json
        private string GenerateJwtToken(User user)
        {
            var jwtSettings = _configuration.GetSection("Jwt");
            var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings["Key"]));
            var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

            var claims = new[]
            {
                new Claim(ClaimTypes.Name, user.UserName),
                new Claim(ClaimTypes.Role, user.Role)
            };

            var expiryMinutes = Convert.ToDouble(jwtSettings["TokenExpiryMinutes"]);

            var token = new JwtSecurityToken(
                issuer: jwtSettings["Issuer"],
                audience: jwtSettings["Audience"],
                claims: claims,
                expires: DateTime.Now.AddMinutes(expiryMinutes),
                signingCredentials: creds
            );

            return new JwtSecurityTokenHandler().WriteToken(token);
        }

        // ✅ LOGIN API
        [HttpPost("login")]
        public async Task<IActionResult> Login([FromBody] User loginUser)
        {
            var user = await _context.Users
                .FirstOrDefaultAsync(u => u.UserName == loginUser.UserName && u.Password == loginUser.Password);

            if (user == null)
                return Unauthorized(new { message = "Invalid username or password" });

            var token = GenerateJwtToken(user);

            return Ok(new
            {
                token,
                user = new { user.UserId, user.UserName, user.Email, user.Role }
            });
        }
    }
}
```

You can apply `[Authorize]` **at both the controller level and the method level.**

---

## ✅ 1️⃣ **Controller-Level `[Authorize]`**

When applied at the **controller level**, **all actions inside that controller require authentication** unless overridden.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Address_API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    [Authorize] // ✅ Applied to the entire controller
    public class UserController : ControllerBase
    {
        private readonly AddressDemoDbContext _context;

        public UserController(AddressDemoDbContext context)
        {
            _context = context;
        }

        // 🛡️ This will require any authenticated user (no role restriction)
        [HttpGet("profile")]
        public IActionResult Profile()
        {
            return Ok(new { message = "Welcome Authenticated User!" });
        }

        // 🛡️ Override with specific roles
        [Authorize(Roles = "Admin")]
        [HttpGet("all-users")]
        public async Task<IActionResult> GetAllUsers()
        {
            return Ok(await _context.Users.ToListAsync());
        }

        // 🛡️ Allow anonymous access (overrides controller-level authorize)
        [AllowAnonymous]
        [HttpGet("public-info")]
        public IActionResult PublicInfo()
        {
            return Ok(new { message = "This is public info, no login required." });
        }
    }
}
```
