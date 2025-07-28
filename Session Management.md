# How to Implement Session Management with Login
**Prerequisites:** Basic understanding of session management
### Step 1: Register Session Related services in `Program.cs` file
Register these services before application build
```csharp
builder.Services.AddDistributedMemoryCache();
builder.Services.AddHttpContextAccessor();
builder.Services.AddSession();
```
After application build, add following
```csharp
app.UseSession();
app.UseAuthentication();
```

### Step 2: Create a model class for Login
Here we created `UserLoginModel` model class to manage login information

```csharp
public class UserLoginModel
{
    [Required]
    public string UserName { get; set; }
    [Required]
    public string Password { get; set; }
}
```
### Step 3: Create another class to retrieve information stored in session
Here we have created a `CommonVariables` static class to retrieve session information using its static methods.
```csharp
public static class CommonVariables
{
    //Provides access to the current Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext
    private static IHttpContextAccessor _httpContextAccessor;

    static CommonVariables()
    {
        _httpContextAccessor = new HttpContextAccessor();
    }

    public static int? UserID()
    {
        //Initialize the UserID with null
        int? UserID = null;

        //check if session contains specified key?
        //if it contains then return the value contained by the key.
        if (_httpContextAccessor.HttpContext.Session.GetString("UserID") != null)
        {
            UserID = Convert.ToInt32(_httpContextAccessor.HttpContext.Session.GetString("UserID").ToString());
        }
        return UserID;
    }

    public static string? UserName()
    {
        string? UserName = null;

        if (_httpContextAccessor.HttpContext.Session.GetString("UserName") != null)
        {
            UserName = _httpContextAccessor.HttpContext.Session.GetString("UserName").ToString();
        }
        return UserName;
    }
}
```
### Step 4: Create a custom authorization filter
Here we have created `CheckAccess` authorization filter which inherits ActionFilterAttribute class and IAuthorizationFilter interface
```csharp
public class CheckAccess : ActionFilterAttribute, IAuthorizationFilter
{
    //When User ID is not availale or removed from session,
    // it will redirect to login page
    public void OnAuthorization(AuthorizationFilterContext filterContext)
    {
        if (filterContext.HttpContext.Session.GetString("UserID") == null)
            filterContext.Result = new RedirectResult("~/SEC_User/Index");
    }

    // Once we logout (session is cleared) then we can not go back to the previous screen
    // We must login to proceed further.
    public override void OnResultExecuting(ResultExecutingContext context)
    {
        if (context.HttpContext.Session.GetString("UserID") == null)
        {
            context.HttpContext.Response.Headers["Cache-Control"] = "no-cache, no-store, must-revalidate";
            context.HttpContext.Response.Headers["Expires"] = "-1";
            context.HttpContext.Response.Headers["Pragma"] = "no-cache";
            base.OnResultExecuting(context);
        }
    }
}
```

### Step 5: Design a Login page  
Here only required design is considered for reference
```csharp
@model UserLoginModel

@if(TempData["ErrorMessage"] != null)
{
  <div class="text-danger">@Html.Raw(@Convert.ToString(TempData["ErrorMessage"]))</div>
}

<form class="login100-form validate-form" asp-area="" asp-controller="SEC_User" asp-action="Login">
  <div class="container">
    <span class="form-control">Username</span>
    <input class="form-control" type="text" asp-for="UserName" placeholder="Enter username">
    <span asp-validation-for="UserName" class="focus-input100"></span>
  </div>

  <div class="container">
    <span class="form-control">Password</span>
    <input class="form-control" type="password" asp-for="Password" placeholder="Enter password">
    <span asp-validation-for="Password"></span>
  </div>
    
  <div class="container">
    <button type="submit" class="btn btn-primary">Login</button>
  </div>
</form>
```

### Step 6: Define Login and Logout action methods inside controller
Here we have created Login() and Logout() action methods.
```csharp
public class SEC_UserController : Controller
{
    private IConfiguration Configuration;

    public SEC_UserController(IConfiguration _configuration)
    {
        Configuration = _configuration;
    }

    public IActionResult Index()
    {
        return View("Login");
    }

    [HttpPost]
    public IActionResult Login(UserLoginModel userLoginModel)
    {
        string ErrorMsg = string.Empty;

        if (string.IsNullOrEmpty(userLoginModel.UserName))
        {
            ErrorMsg += "User Name is Required";
        }

        if (string.IsNullOrEmpty(userLoginModel.Password))
        {
            ErrorMsg += "<br/>Password is Required";
        }

        if (ModelState.IsValid)
        {
            SqlConnection conn = new SqlConnection(this.Configuration.GetConnectionString("ABConnectionString"));
            conn.Open();

            SqlCommand objCmd = conn.CreateCommand();

            objCmd.CommandType = System.Data.CommandType.StoredProcedure;
            objCmd.CommandText = "PR_SEC_User_Login";
            objCmd.Parameters.AddWithValue("@UserName", userLoginModel.UserName);
            objCmd.Parameters.AddWithValue("@Password", userLoginModel.Password);

            SqlDataReader objSDR = objCmd.ExecuteReader();

            DataTable dtLogin = new DataTable();

            // Check if Data Reader has Rows or not
            // if row(s) does not exists that means either username or password or both are invalid.
            if (!objSDR.HasRows)
            {
                TempData["ErrorMessage"] = "Invalid User Name or Password";
                return RedirectToAction("Index", "SEC_User");
            }
            else
            {
                dtLogin.Load(objSDR);

                //Load the retrived data to session through HttpContext.
                foreach (DataRow dr in dtLogin.Rows)
                {
                    HttpContext.Session.SetString("UserID", dr["UserID"].ToString());
                    HttpContext.Session.SetString("UserName", dr["UserName"].ToString());
                    HttpContext.Session.SetString("MobileNo", dr["MobileNo"].ToString());
                    HttpContext.Session.SetString("Email", dr["Email"].ToString());
                    HttpContext.Session.SetString("Password", dr["Password"].ToString());
                }
                return RedirectToAction("Index", "Home");
            }
        }
        else
        {
            TempData["ErrorMessage"] = ErrorMsg;
            return RedirectToAction("Index", "SEC_User");
        }
    }

    [HttpPost]
    public IActionResult Logout()
    {
        HttpContext.Session.Clear();
        return RedirectToAction("Index");
    }
}
```

### Step 7: Apply Custom Action Filter attribute over controller
Here we have named over custom action filter attribute as `CheckAccess` so apply it on controllers that requires user authentication. Whenever any action method of the controller is called, at first the custom action filter will be called and executed
> - If you place [CheckAccess] on a controller like HomeController, every action in that controller will run through the CheckAccess authorization logic.
> - If you place [CheckAccess] on individual action methods, only those specific methods will use the custom authorization logic.

```csharp
[CheckAccess]
public class HomeController : Controller
{
  // Rest of code for controller
}
```
