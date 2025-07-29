# MVC Project
## Step 1: Model Class
```csharp
public class JobApplication
{
    public int JobId { get; set; }
    public string PersonName { get; set; } = string.Empty;
    public string? Description { get; set; }
    public string? ResumePath { get; set; }
    public IFormFile? Resume { get; set; }
}
```
---
## Step 2: Controller class
```csharp
public class HomeController : Controller
{
    
    private readonly HttpClient _httpClient;
    public HomeController(HttpClient httpClient)
    {
        _httpClient = httpClient;
        _httpClient.BaseAddress = new System.Uri("https://localhost:7282/api/");
    }

    public async Task<IActionResult> Index()
    {
        var request = new HttpRequestMessage(HttpMethod.Get, $"Home/Index");
     
        var response = await _httpClient.SendAsync(request);
        if (response.IsSuccessStatusCode)
        {
            var result = response.Content.ReadAsStringAsync().Result;

            var jobs = JsonConvert.DeserializeObject<List<JobApplication>>(result);

            return View("Index", jobs);
        }
        else
        {
            TempData["ErrorMessage"] = "Unable to fetch data. Please try again later.";
        }

        return View("Index");
    }

    [HttpGet]
    public async Task<IActionResult> JobAddEdit(int? JobID)
    {
        if (JobID != null && JobID > 0)
        {
            JobApplication job = new JobApplication();
            var request = new HttpRequestMessage(HttpMethod.Get, $"Home/GetJobApplicationByID?jobID={JobID}");

            var response = await _httpClient.SendAsync(request);
            if (response.IsSuccessStatusCode)
            {
                var result = response.Content.ReadAsStringAsync().Result;

                job = JsonConvert.DeserializeObject<JobApplication>(result);

                return View("JobAddEdit", job);
            }
        }

        // If new student or fetch failed, return empty form
        return View("JobAddEdit");
    }

    // Deletes a student by ID
    [HttpPost]
    public async Task<IActionResult> Delete(int JobId)
    {
        if (JobId > 0)
        {
            var request = new HttpRequestMessage(HttpMethod.Delete, $"Home/DeleteByJobID?jobID={JobId}");

            var response = await _httpClient.SendAsync(request);
            if (response.IsSuccessStatusCode)
            {
                TempData["SuccessMessage"] = "Record Deleted";
            }
            else
            {
                TempData["ErrorMessage"] = "Error Occured";
            }
        }

        return RedirectToAction("Index");
    }

    [HttpPost]
    public async Task<IActionResult> Save(JobApplication job)
    {
        var formData = ConvertToMultipartFormData(job);

        var request = new HttpRequestMessage(HttpMethod.Post, $"Home/Save");
        request.Content = formData;

        var response = await _httpClient.SendAsync(request);
        Console.Write(response);

        if (response.IsSuccessStatusCode)
        {
            if (job.JobId > 0)
            {
                TempData["SuccessMessage"] = "Record Updated Successfully";
                return RedirectToAction("Index");
            }
            else
            {
                TempData["SuccessMessage"] = "Record Saved Successfully";
                return RedirectToAction("JobAddEdit");
            }
        }
        else
        {
            TempData["ErrorMessage"] = "Error Occured";

            return View("JobAddEdit", job);
        }
    }

    public MultipartFormDataContent ConvertToMultipartFormData(JobApplication jobApplication)
    {
        var formData = new MultipartFormDataContent();

        // Add basic properties to form-data
        formData.Add(new StringContent(jobApplication.JobId.ToString() ?? ""), "JobId");
        formData.Add(new StringContent(jobApplication.PersonName ?? ""), "PersonName");
        formData.Add(new StringContent(jobApplication.Description ?? ""), "Description");
        formData.Add(new StringContent(jobApplication.PersonName ?? ""), "PersonName");
        formData.Add(new StringContent(jobApplication.ResumePath ?? ""), "ResumePath");

        // Add uploaded file if available
        if (jobApplication.Resume != null && jobApplication.Resume.Length > 0)
        {
            var streamContent = new StreamContent(jobApplication.Resume.OpenReadStream());
            streamContent.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue(jobApplication.Resume.ContentType);

            formData.Add(streamContent, "Resume", jobApplication.Resume.FileName);
        }

        return formData;
    }
}
```
---
## Step 3: Add/Update form
```csharp
@using MVC_FileUpload.Controllers
@model JobApplication
@if (TempData["ErrorMessage"] != null)
{
    <div class="alert alert-danger">@TempData["ErrorMessage"]</div>
}

@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success">@TempData["SuccessMessage"]</div>
}

<form asp-action="Save" asp-controller="Home" enctype="multipart/form-data">
    <input type="hidden" asp-for="ResumePath" />
    <div class="form-group">
        <label for="exampleInputEmail1">Job ID</label>
        <input type="text" class="form-control" asp-for="JobId" placeholder="Enter Job ID">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">Person Name</label>
        <input type="text" class="form-control" asp-for="PersonName" placeholder="Person Name">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">Description</label>
        <input type="text" class="form-control" asp-for="Description" placeholder="Description">
    </div>
    <br />
    <div class="form-group">
        <label for="exampleFormControlFile1">Resume</label>
        <input type="file" class="form-control-file" asp-for="Resume">
    </div>
    @if (Model != null && Model.ResumePath != null)
    {
        <div class="form-group">
            <img src="https://localhost:7282/@Model.ResumePath" alt="Resume Image" class="img-thumbnail" style="max-height: 100px;" />
        </div>
    }
    <br />
    <button type="submit" class="btn btn-primary">Save</button>
    <a asp-action="Index" asp-controller="Home" class="btn btn-danger">Cancle</a>
</form>
```
---
## Step 4: Index (List) page
```csharp
@using MVC_FileUpload.Controllers
@{
    ViewData["Title"] = "Home Page";
}
@model List<JobApplication>

<div class="text-center">
    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger">@TempData["ErrorMessage"]</div>
    }

    @if (TempData["SuccessMessage"] != null)
    {
        <div class="alert alert-success">@TempData["SuccessMessage"]</div>
    }
    <a asp-action="JobAddEdit" class="btn btn-success">Add Job Application</a>
    <div class="container mt-4">
        <h3 class="mb-3">Job Application List</h3>
        <table class="table table-bordered table-hover align-middle">
            <thead class="table-dark">
                <tr>
                    <th scope="col">@nameof(JobApplication.JobId)</th>
                    <th scope="col">@nameof(JobApplication.PersonName)</th>
                    <th scope="col">@nameof(JobApplication.Description)</th>
                    <th > @nameof(JobApplication.ResumePath)</th>
                    <th colspan="2">Action</th>
                </tr>
            </thead>
            <tbody>
                @if (Model.Count > 0)
                    @foreach (var job in Model)
                    {
                        <tr>
                            <td>@job.JobId</td>
                            <td>@job.PersonName</td>
                            <td>@job.Description</td>
                            <td>
                                <img src="https://localhost:7282/@job.ResumePath" alt="@job.ResumePath" class="img-thumbnail" style="max-height: 100px;">
                            </td>
                            <td><a asp-action="JobAddEdit" class="btn btn-primary" asp-route-JobId="@job.JobId">Edit</a> </td>
                                <td><form asp-action="Delete" method="post">
                                    <input type="hidden" name="JobId" value="@job.JobId" />
                                    <input type="submit" class="btn btn-danger" value="Delete" />
                                </form>
                            </td>
                        </tr>
                    }
            </tbody>
        </table>
    </div>
</div>
```
