Here is a **properly formatted Markdown file** with necessary comments for better understanding:

# 🔐 Authentication & Role-Based Access Implementation in ASP.NET Core MVC

This example demonstrates how to **authenticate users** and **store their JWT token and role in session**, which can then be used to authorize access to different parts of the application.

---

## 📌 **1️⃣ Register Services in `Program.cs`**

```csharp
// ✅ Register HttpClient, IHttpContextAccessor, and AuthService
builder.Services.AddHttpClient();
builder.Services.AddHttpContextAccessor();
builder.Services.AddScoped<AuthService>();

// ✅ Enable Session for JWT storage
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30); // Session expiry
});

var app = builder.Build();

// ✅ Enable Session Middleware
app.UseSession();
```

---


## 📌 **2️⃣ Login View (`Login.cshtml`)**

```html
<form method="post" class="col-4">
    <!-- Username -->
    <div class="form-group">
        <label>Username</label>
        <input type="text" class="form-control" name="Username" required />
    </div>

    <!-- Password -->
    <div class="form-group">
        <label>Password</label>
        <input type="password" class="form-control" name="Password" required />
    </div>

    <!-- Role Dropdown -->
    <div class="form-group">
        <label>Role</label>
        <select class="form-control" name="Role" required>
            <option value="Admin">Admin</option>
            <option value="Manager">Manager</option>
            <option value="User">User</option>
        </select>
    </div>

    <br />
    <button type="submit" class="btn btn-primary">Login</button>
</form>
```
---
## 📌 **3️⃣ Login Controller**

```csharp
[HttpPost]
public async Task<IActionResult> Login(string Username, string Password, string Role)
{
    // ✅ Call AuthService for authentication
    var jsonData = await _authService.AuthenticateUserAsync(Username, Password, Role);

    // ❌ If authentication fails
    if (jsonData == null)
    {
        ViewBag.Error = "Invalid credentials.";
        return View();
    }

    // ✅ Deserialize JSON response
    var data = JsonConvert.DeserializeObject<Dictionary<string, dynamic>>(jsonData);
    string token = data["token"];

    // ❌ If token is empty, login failed
    if (string.IsNullOrEmpty(token))
    {
        ViewBag.Error = "Invalid credentials.";
        return View();
    }

    // ✅ Store token & role in session
    _httpContextAccessor.HttpContext.Session.SetString("JWTToken", token);
    _httpContextAccessor.HttpContext.Session.SetString("UserRole", Role);

    // ✅ Redirect to home/dashboard
    return RedirectToAction("Index", "Home");
}
```

---

## 📌 **4️⃣ AuthService.cs**

```csharp
// ✅ AuthService is responsible for making API calls for authentication
//    It sends Username, Password, and Role to the API and returns the JSON response.
public async Task<string?> AuthenticateUserAsync(string username, string password, string? role = null)
{
    // ✅ Prepare request body
    var requestData = new 
    { 
        UserName = username, 
        Password = password, 
        Role = role // ✅ Sending role if required
    };

    // ✅ Convert request data to JSON
    var content = new StringContent(JsonConvert.SerializeObject(requestData), Encoding.UTF8, "application/json");

    // ✅ Call API endpoint
    HttpResponseMessage response = await _httpClient.PostAsync("https://localhost:7093/api/User/login", content);

    // ✅ If success, return JSON response
    if (response.IsSuccessStatusCode)
    {
        return await response.Content.ReadAsStringAsync();
    }

    // ❌ Return null if login fails
    return null;
}
````

---

## 📌 ** 5️⃣ UserController.cs**

```csharp
using Address_Consume.Models;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;
using System.Text;

namespace Address_Consume.Controllers
{
    public class UserController : Controller
    {
        private readonly HttpClient _client;
        private readonly IHttpContextAccessor _httpContextAccessor;

        // ✅ Constructor injection for HttpClient and IHttpContextAccessor
        public UserController(IHttpClientFactory httpClientFactory, IHttpContextAccessor httpContextAccessor)
        {
            _client = httpClientFactory.CreateClient();
            _client.BaseAddress = new Uri("https://localhost:7093/api/");
            _httpContextAccessor = httpContextAccessor;
        }

        // ✅ Adds JWT token from session to request headers
        private void AddJwtToken()
        {
            var token = _httpContextAccessor.HttpContext.Session.GetString("JWTToken");
            if (!string.IsNullOrEmpty(token))
            {
                _client.DefaultRequestHeaders.Authorization =
                    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
            }
        }

        // ✅ Displays list of users (only if token is valid)
        public async Task<IActionResult> Index()
        {
            try
            {
                AddJwtToken(); // 🔑 Add token before calling API

                var response = await _client.GetAsync("User");

                // 🔒 Redirect to login if unauthorized
                if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
                {
                    return RedirectToAction("Login", "Login");
                }

                // ✅ Deserialize response into user list
                response.EnsureSuccessStatusCode();
                var json = await response.Content.ReadAsStringAsync();
                var list = JsonConvert.DeserializeObject<List<UserModel>>(json);

                return View(list);
            }
            catch
            {
                TempData["Error"] = "Unable to load users.";
                return View(new List<UserModel>());
            }
        }

        // ⚠️ Similarly, AddEdit and Delete methods should call AddJwtToken() before API requests
    }
}
```


---

## 📌 ** 6️⃣Role-Based UI Display (Example)**

```csharp
// ✅ Get user role from session
var role = _httpContextAccessor.HttpContext.Session.GetString("UserRole");

if (role == "Admin")
{
    // Show admin dashboard
}
else if (role == "User")
{
    // Show user dashboard
}
```

### ✅ Razor View Example (Role-Based Navigation)

```csharp
@if (Context.Session.GetString("UserRole") == "Admin")
{
    <a href="/User/Index">Manage Users</a>
}

@if (Context.Session.GetString("UserRole") == "User")
{
    <a href="/Profile">My Profile</a>
}
```

---
