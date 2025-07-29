# Web API project

## Step 1: Add following in Program.cs
app.UseStaticFiles();
---
## Step 2: Add model classes as required
```csharp
public class JobApplication
{
    public int JobId { get; set; }
    public string PersonName { get; set; } = string.Empty;
    public string? Description { get; set; }
    public string? ResumePath { get; set; }
    [JsonIgnore]
    public IFormFile? Resume {  get; set; }
}
```
---
## Step 3: Add ImageHelper static class
```csharp
public static class ImageHelper
{
    public static string directory = "Images";
    public static string SaveImageToFile(IFormFile imageFile)
    {
        if (imageFile == null || imageFile.Length == 0)
            return null;
        if (!Directory.Exists($"wwwroot/{directory}"))
        {
            Directory.CreateDirectory($"wwwroot/{directory}");
        }

        string fullPath = $"{directory}/{Guid.NewGuid()}{Path.GetExtension(imageFile.FileName)}";

        using (var stream = new FileStream($"wwwroot/{fullPath}", FileMode.Create))
        {
            imageFile.CopyTo(stream);
        }

        return fullPath;
    }

    public static string DeleteFile(string filePath)
    {
        var path = $"{Directory.GetCurrentDirectory()}/wwwroot/{filePath}";

        if (!System.IO.File.Exists(path)) return "File not found.";

        try
        {
            System.IO.File.Delete(path);
            return "File deleted successfully.";
        }
        catch (Exception ex)
        {
            throw;
        }
    }
}
```
---
## Step 4: Add Controller to create endpoints for CRUD
```csharp
[Route("api/[controller]/[action]")]
[ApiController]
public class HomeController : ControllerBase
{
    static List<JobApplication> jobApplications = new List<JobApplication>();

    [HttpGet]
    public List<JobApplication> Index()
    {
        return jobApplications;
    }

    [HttpGet]
    public JobApplication GetJobApplicationByID(int jobID)
    {
        return jobApplications.Find(s => s.JobId == jobID);
    }

    [HttpPost]
    public IActionResult Save([FromForm] JobApplication jobApplication)
    {
        string savedPath = String.Empty;

        if (jobApplication.Resume != null)
        {
            if (jobApplication.ResumePath != null)
            {
                try
                {
                    string message = ImageHelper.DeleteFile(jobApplication.ResumePath);
                }
                catch (Exception ex)
                {
                    return NotFound("File does not exists");
                }
            }

            savedPath = ImageHelper.SaveImageToFile(jobApplication.Resume);
            jobApplication.ResumePath = savedPath;

            if (string.IsNullOrEmpty(savedPath))
                return BadRequest("Failed to upload image.");
        }

        if (jobApplications.Find(x => x.JobId == jobApplication.JobId) == null)
        {
            jobApplications.Add(jobApplication);
        }
        else
        {
            var std = jobApplications.Find(s => s.JobId == jobApplication.JobId);
            jobApplications.Remove(std);
            jobApplications.Add(jobApplication);
        }

        return Ok(jobApplication);
    }

    [HttpDelete]
    public IActionResult DeleteByJobID(int jobID)
    {
        var job = jobApplications.Find(x => x.JobId == jobID);

        if (job == null)
        {
            return BadRequest(new { Message = "Invalid Student" });
        }
        string message = ImageHelper.DeleteFile(job.ResumePath);
        jobApplications.Remove(job);

        return Ok(job);
    }
}
```
