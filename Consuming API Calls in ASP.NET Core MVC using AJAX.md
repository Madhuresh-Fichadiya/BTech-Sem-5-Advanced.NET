# Consuming API Calls in ASP.NET Core MVC using AJAX

how to consume **Web API calls** in an ASP.NET Core MVC project using **AJAX (jQuery)**.  
We’ll build a simple **State Management** module with CRUD (Create, Read, Update, Delete) functionality.

---

## 1. Enable CORS in Web API

In `Program.cs`, add CORS configuration:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

app.UseCors("AllowAll");
```

This ensures the MVC project can call the API without CORS errors.

---

## 2. Create MVC Controller

In the **MVC project**, add a controller to serve the view:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace Address_Consume.Controllers
{
    public class StatesController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
    }
}
```

---

## 3. Create the View (Index.cshtml)

The view contains a form for creating/updating states and a table to display all states.  
AJAX is used to make API calls for CRUD operations.

```html
@{
    ViewData["Title"] = "States Management";
}

<div class="container mt-4">
    <h2 class="mb-4">Manage States (AJAX + API)</h2>

    <form id="stateForm" class="row g-3" enctype="multipart/form-data">
        <input type="hidden" id="StateId" name="StateId" />
        <div class="col-md-4">
            <label for="StateName" class="form-label">State Name</label>
            <input type="text" id="StateName" name="StateName" class="form-control" required />
        </div>
        <div class="col-md-4">
            <label for="StateCode" class="form-label">State Code</label>
            <input type="text" id="StateCode" name="StateCode" class="form-control" required />
        </div>
        <div class="col-md-4">
            <label for="CountryId" class="form-label">Country</label>
            <select id="CountryId" name="CountryId" class="form-select"></select>
        </div>
        <div class="col-md-6">
            <label for="File" class="form-label">Upload Image</label>
            <input type="file" id="File" name="File" class="form-control" accept="image/*" onchange="previewImage(event)" />
        </div>
        <div class="col-md-6">
            <label class="form-label">Current Image</label><br />
            <img id="StateImagePreview" src="" alt="State Image" class="img-thumbnail" style="max-height:150px; display:none;" />
        </div>
        <div class="col-12">
            <button type="submit" class="btn btn-primary">Save</button>
            <button type="button" class="btn btn-secondary" onclick="resetForm()">Reset</button>
        </div>
    </form>

    <hr class="my-4" />

    <table class="table table-bordered table-striped" id="stateTable">
        <thead class="table-dark">
            <tr>
                <th>ID</th>
                <th>State Name</th>
                <th>Code</th>
                <th>Country</th>
                <th>Image</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>
</div>
```

---

## 4. AJAX Script for CRUD Operations

Add jQuery and AJAX script inside `Index.cshtml`:

```html
@section Scripts {
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
        const stateApiUrl = "https://localhost:7093/api/State";
        const countryApiUrl = "https://localhost:7093/api/State/dropdown";

        $(document).ready(function () {
            loadStates();
            loadCountries();

            // Insert / Update
            $("#stateForm").submit(function (e) {
                e.preventDefault();

                let formData = new FormData(this);
                let stateId = $("#StateId").val();

                if (stateId) {
                    // Update
                    $.ajax({
                        url: stateApiUrl + "/" + stateId,
                        type: "PUT",
                        data: formData,
                        processData: false,
                        contentType: false,
                        success: function () {
                            alert("State updated!");
                            resetForm();
                            loadStates();
                        },
                        error: function (xhr) {
                            console.error("Update error:", xhr.responseText);
                            alert("Error updating state!");
                        }
                    });
                } else {
                    // Insert
                    $.ajax({
                        url: stateApiUrl,
                        type: "POST",
                        data: formData,
                        processData: false,
                        contentType: false,
                        success: function () {
                            alert("State added!");
                            resetForm();
                            loadStates();
                        },
                        error: function (xhr) {
                            console.error("Insert error:", xhr.responseText);
                            alert("Error adding state!");
                        }
                    });
                }
            });
        });

        // Load all states
        function loadStates() {
            $.get(stateApiUrl, function (data) {
                let rows = "";
                $.each(data, function (i, item) {
                    rows += `
                        <tr>
                            <td>${item.stateId}</td>
                            <td>${item.stateName}</td>
                            <td>${item.stateCode}</td>
                            <td>${item.countryName}</td>
                            <td>
                                ${item.filePath ? `<img src="https://localhost:7093/${item.filePath}" width="50"/>` : "No Image"}
                            </td>
                            <td>
                                <button class="btn btn-sm btn-warning" onclick="editState(${item.stateId})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deleteState(${item.stateId})">Delete</button>
                            </td>
                        </tr>`;
                });
                $("#stateTable tbody").html(rows);
            }).fail(function (xhr) {
                console.error("Load states error:", xhr.responseText);
                alert("Failed to load states.");
            });
        }

        // Load countries dropdown
        function loadCountries() {
            $.ajax({
                url: countryApiUrl,
                type: "GET",
                success: function (data) {
                    let options = "<option value=''>-- Select Country --</option>";
                    $.each(data, function (i, country) {
                        options += `<option value="${country.countryId}">${country.countryName}</option>`;
                    });
                    $("#CountryId").html(options);
                },
                error: function (xhr) {
                    console.error("Country load error:", xhr.responseText);
                    alert("Failed to load countries.");
                }
            });
        }

        // Edit state
        function editState(id) {
            $.get(stateApiUrl + "/" + id, function (data) {
                $("#StateId").val(data.stateId);
                $("#StateName").val(data.stateName);
                $("#StateCode").val(data.stateCode);
                $("#CountryId").val(data.countryId);

                if (data.filePath) {
                    $("#StateImagePreview").attr("src", "https://localhost:7093/" + data.filePath).show();
                } else {
                    $("#StateImagePreview").hide();
                }
            }).fail(function (xhr) {
                console.error("Edit error:", xhr.responseText);
            });
        }

        // Delete state
        function deleteState(id) {
            if (confirm("Are you sure?")) {
                $.ajax({
                    url: stateApiUrl + "/" + id,
                    type: "DELETE",
                    success: function () {
                        alert("Deleted successfully!");
                        loadStates();
                    },
                    error: function (xhr) {
                        console.error("Delete error:", xhr.responseText);
                        alert("Error deleting state!");
                    }
                });
            }
        }

        // Reset form
        function resetForm() {
            $("#StateId").val("");
            $("#stateForm")[0].reset();
            $("#StateImagePreview").hide().attr("src", "");
        }

        // Preview uploaded image instantly
        function previewImage(event) {
            const reader = new FileReader();
            reader.onload = function () {
                const output = document.getElementById('StateImagePreview');
                output.src = reader.result;
                output.style.display = "block";
            };
            reader.readAsDataURL(event.target.files[0]);
        }
    </script>
}
```

---

## ✅ Summary

- **API is hosted on `https://localhost:7093/`**
- **CORS enabled** for MVC project access
- **AJAX used for CRUD operations** (GET, POST, PUT, DELETE)
- **Image upload with preview support**
- **Dynamic country dropdown population**

This provides a working **AJAX + Web API integration** inside an ASP.NET Core MVC project.
