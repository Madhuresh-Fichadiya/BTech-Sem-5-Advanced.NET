# Complete CRUD with Jwt authentication and File upload

**Prerequisite**: Basic knowledge of .NET Core and WebAPI
# Part 1: Web API project

## Step 1: Add following packages
While installing packages make sure your laptop/pc .net framwork matches with packages
 - Microsoft.IdentityModel.Tokens;
 - Microsoft.AspNetCore.Authentication.JwtBearer
 - Newtonsoft.Json

## Step 2: Add following code in appsettings.json file 
JWT configuration settings for token generation and validation
```csharp
"Jwt": 
 {
  // The secret key used to sign and verify the JWT token
  // Must be kept secure and not exposed in source control
  "Key": "ThisIsASecretKeyForJwtTokenGeneration",

  // The issuer of the token, identifying the server that generated it
  // Typically the URL of the application (e.g., your API's domain)
  "Issuer": "https://localhost:5001",

  // The audience of the token, identifying the intended recipients
  // Usually matches the issuer or specifies authorized clients
  "Audience": "https://localhost:5001"
}
```

## Step 3: Add following code in Program.cs file
Configures JWT authentication for the application
```csharp
// Configures JWT authentication for the application
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme) // Sets JWT Bearer as the default authentication scheme
    .AddJwtBearer(options => // Configures JWT Bearer authentication options
    {
        // Disables the requirement for HTTPS metadata (useful for development; enable in production)
        options.RequireHttpsMetadata = false;
        // Saves the JWT token in the request context for later use
        options.SaveToken = true;
        // Defines how the JWT token should be validated
        options.TokenValidationParameters = new TokenValidationParameters
        {
            // Ensures the token's issuer matches the expected issuer
            ValidateIssuer = true,
            // Ensures the token's audience matches the expected audience
            ValidateAudience = true,
            // Ensures the token has not expired
            ValidateLifetime = true,
            // Ensures the token is signed with the correct key
            ValidateIssuerSigningKey = true,
            // Specifies the expected issuer (e.g., your app's domain)
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            // Specifies the expected audience (e.g., your app's clients)
            ValidAudience = builder.Configuration["Jwt:Audience"],
            // Specifies the key used to verify the token's signature
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

builder.Services.AddAuthorization();
```
Add following code after   var app = builder.Build();
```csharp
//Enable static file middleware that will be helpful to access `wwwroot` folder.
app.UseStaticFiles();
```

## Step 4: Add model classes as required
```csharp
public class UserModel
{
    public string? Username { get; set; }
    public string? Password { get; set; }
}

// Represents the data model for a student
public class Student
{
    public int StudentID { get; set; }
    public string Name { get; set; }
    public string? FilePath { get; set; } // URL to the student's image
    public IFormFile? File { get; set; } // Uploaded image file
}
```
## Step 5: Add `ImageHelper` utility class for handling image file operations
```csharp
public static class ImageHelper
{
    // Saves an uploaded image file to the specified directory and returns its relative path
    public static string SaveImageToFile(IFormFile imageFile, string directoryPath)
    {
        // Constructs the full directory path under the wwwroot folder
        string directory = $"wwwroot/{directoryPath}";

        // Validates that the image file is not null and has content
        if (imageFile == null || imageFile.Length == 0)
            return null;

        // Ensures the directory exists, creating it if necessary
        if (!Directory.Exists(directory))
        {
            Directory.CreateDirectory(directory);
        }

        // Gets the file extension from the uploaded file (e.g., .jpg, .png)
        string fileExtension = Path.GetExtension(imageFile.FileName);
        // Uses a default extension (.jpeg) if none is provided
        if (string.IsNullOrWhiteSpace(fileExtension))
        {
            fileExtension = ".jpeg";
        }

        // Generates a unique file name using a GUID to avoid conflicts
        string fileName = $"{Guid.NewGuid()}{fileExtension}";
        
        // Constructs the relative path for returning to the client
        string fullPath = $"{directoryPath}/{fileName}";
        
        // Constructs the absolute path for saving the file
        string fullPathToWrite = $"{directory}/{fileName}";

        // Saves the image file to the specified path
        using (var stream = new FileStream(fullPathToWrite, FileMode.Create))
        {
            imageFile.CopyTo(stream); // Copies the file content to the stream
        }

        // Returns the relative path for accessing the saved image
        return fullPath;
    }

    // Reads the contents of a file as a byte array
    public static byte[] ReadFileBytes(string filePath)
    {
        // Validates that the file path is not empty and the file exists
        if (string.IsNullOrWhiteSpace(filePath) || !File.Exists(filePath))
            return null;

        // Reads and returns the file's contents as a byte array
        return File.ReadAllBytes(filePath);
    }

    // Deletes a file based on its URL and returns a status message
    public static string DeleteFileFromUrl(string fileUrl)
    {
        // Validates that the file URL is not empty
        if (string.IsNullOrWhiteSpace(fileUrl))
            return "File URL is required.";

        // Parses the URL to extract the file name (e.g., 391fc38d-a144-4817-9731-7da9a87c5f27.jpg)
        var uri = new Uri(fileUrl);
        var fileName = Path.GetFileName(uri.LocalPath);

        // Constructs the physical path to the file in the wwwroot/Images directory
        var filePath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", "Images", fileName);

        // Checks if the file exists
        if (!System.IO.File.Exists(filePath))
            return "File not found.";

        try
        {
            // Deletes the file from the file system
            System.IO.File.Delete(filePath);
            return "File deleted successfully.";
        }
        catch (Exception ex)
        {
            // Rethrows the exception to be handled by the caller
            throw;
        }
    }
}
```

## Step 6: Add Authentication Controller
Here we have added Authentication controller for Login and generate Json Web Token.
```csharp
// Defines the route for this controller as "api/auth/[action]" (e.g., api/auth/login)
[Route("api/[controller]/[action]")]
// Marks this class as an API controller, enabling API-specific behaviors
[ApiController]
public class AuthController : ControllerBase
{
    // Stores configuration settings (e.g., JWT key, issuer, audience)
    private readonly IConfiguration _config;

    // Constructor that injects the configuration dependency
    public AuthController(IConfiguration config)
    {
        _config = config; // Assigns the configuration to the private field
    }

    // Handles HTTP POST requests to the Login action (e.g., api/auth/login)
    [HttpPost]
    public IActionResult Login([FromBody] UserModel user)
    {
        // Checks if the provided username and password match the expected values
        if (user.Username == "admin" && user.Password == "password")
        {
            // Generates a JWT token for the user
            var token = GenerateJwtToken(user.Username);
            // Returns a 200 OK response with the token in JSON format
            return Ok(new { token });
        }
        // Returns a 401 Unauthorized response if credentials are invalid
        return Unauthorized();
    }

    // Generates a JWT token for the authenticated user
    private string GenerateJwtToken(string username)
    {
        // Creates a security key using the secret key from configuration
        var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
        // Creates signing credentials using the security key and HMAC-SHA256 algorithm
        var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

        // Defines the claims (user data) to include in the token
        var claims = new[]
        {
        // Subject claim (identifies the user)
        new Claim(JwtRegisteredClaimNames.Sub, username),
        // Unique token identifier
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
    };

        // Creates the JWT token with specified properties
        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"], // Token issuer (e.g., your app's domain)
            audience: _config["Jwt:Audience"], // Intended audience (e.g., your app's clients)
            claims: claims, // User claims defined above
            expires: DateTime.UtcNow.AddHours(1), // Token expires in 1 hour
            signingCredentials: credentials // Signs the token with the credentials
        );

        // Converts the token to a string format and returns it
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```
## Step 7: Add Student Controller to create endpoints for student CRUD

```csharp
// Defines the route for this controller as "api/home/[action]" (e.g., api/home/index)
[Route("api/[controller]/[action]")]
// Marks this class as an API controller, enabling API-specific behaviors
[ApiController]
// Requires authentication for all actions in this controller (JWT token must be provided)
[Authorize]
public class HomeController : ControllerBase
{
    // Defines the directory where images will be stored
    private readonly string _imageDirectory = "Images";
    // In-memory list to store student data (simulates a database)
    static List<Student> students = new List<Student>();

    // GET: api/home/index
    // Retrieves the list of all students
    [HttpGet]
    public List<Student> Index()
    {
        // Returns the entire list of students
        return students;
    }

    // GET: api/home/getstudentbyid?studentID={id}
    // Retrieves a single student by their ID
    [HttpGet]
    public Student GetStudentByID(int studentID)
    {
        // Finds and returns the student with the matching ID
        // Throws an exception if no student is found
        return students.Where(s => s.StudentID == studentID).Single();
    }

    // POST: api/home/save
    // Saves or updates a student record, including an optional image file
    [HttpPost]
    public IActionResult Save([FromForm] Student student)
    {
        string savedPath = String.Empty;

        // Check if an image file is included in the request
        if (student.File != null)
        {
            // If the student already has an associated file, attempt to delete it
            if (student.FilePath != null)
            {
                try
                {
                    // Deletes the existing file using a helper method
                    string message = ImageHelper.DeleteFileFromUrl(student.FilePath);
                }
                catch (Exception ex)
                {
                    // Returns 404 if the file to delete doesn't exist
                    return NotFound("File does not exists");
                }
            }

            // Saves the new image to the specified directory and gets the saved path
            savedPath = ImageHelper.SaveImageToFile(student.File, _imageDirectory);
            // Constructs the full URL for accessing the saved image
            student.FilePath = $"{Request.Scheme}://{Request.Host}/{savedPath}";

            // Returns 400 if the image upload fails
            if (string.IsNullOrEmpty(savedPath))
                return BadRequest("Failed to upload image.");
        }

        // Check if a student with the given ID exists
        if (students.Find(x => x.StudentID == student.StudentID) == null)
        {
            // If no student exists with this ID, add the new student
            students.Add(student);
        }
        else
        {
            // If a student exists, update their record
            var std = students.Find(s => s.StudentID == student.StudentID);
            // Remove the old student record
            students.Remove(std);
            // Add the updated student record
            students.Add(student);
        }
       
        // Returns 200 OK with the saved/updated student data
        return Ok(student);
    }

    // GET: api/home/get/{fileName}
    // Retrieves an image file by its name
    [HttpGet("get/{fileName}")]
    public IActionResult GetImage(string fileName)
    {
        // Constructs the full path to the image file
        string filePath = Path.Combine(_imageDirectory, fileName);
        // Reads the image file as a byte array
        byte[] imageData = ImageHelper.ReadFileBytes(filePath);

        // Returns 404 if the image is not found
        if (imageData == null)
            return NotFound("Image not found.");

        // Returns the image as a file response with JPEG content type
        return File(imageData, "image/jpeg");
    }

    // DELETE: api/home/deletebyid?studentID={id}
    // Deletes a student by their ID
    [HttpDelete]
    public IActionResult DeleteByStudentID(int studentID)
    {
        // Finds the student with the matching ID
        var std = students.Find(x => x.StudentID == studentID);

        // Returns 400 if no student is found
        if (std == null)
        {
            return BadRequest(new { Message = "Invalid Student" });
        }

        // Removes the student from the list
        students.Remove(std);
        // Returns 200 OK with the deleted student data
        return Ok(std);
    }
}
```
