# City Add/Edit Functionality in ASP.NET Core MVC

## Steps to Implement

### 1. **Setup Stored Procedures**

- Create the following stored procedures in your database:
  1.  `PR_LOC_City_SelectByPK`: Fetches city details by `CityID`.
  2.  `PR_LOC_City_Insert`: Inserts a new city record.
  3.  `PR_LOC_City_Update`: Updates an existing city record.
  4.  `PR_LOC_Country_SelectComboBox`: Fetches a list of countries.
  5.  `PR_LOC_State_SelectComboBoxByCountryID`: Fetches states based on the selected country.

---

### 2. Models: `CountryDropDownModel` and `StateDropDownModel`

```csharp
public class CountryDropDownModel
{
    public int CountryID { get; set; }
    public string CountryName { get; set; }
}

public class StateDropDownModel
{
    public int StateID { get; set; }
    public string StateName { get; set; }
}
```

### 3. **City Controller**

```csharp
#region Add
// This action displays the City Add/Edit form
public IActionResult CityAddEdit(int? CityID)
{
    // Load the dropdown list of countries
    LoadCountryList();

    // Check if an edit operation is requested
    if (CityID.HasValue)
    {
        string connectionstr = _configuration.GetConnectionString("ConnectionString");
        DataTable dt = new DataTable();

        // Fetch city details by ID
        using (SqlConnection conn = new SqlConnection(connectionstr))
        {
            conn.Open();
            using (SqlCommand objCmd = conn.CreateCommand())
            {
                objCmd.CommandType = CommandType.StoredProcedure;
                objCmd.CommandText = "PR_LOC_City_SelectByPK";
                objCmd.Parameters.Add("@CityID", SqlDbType.Int).Value = CityID;

                using (SqlDataReader objSDR = objCmd.ExecuteReader())
                {
                    dt.Load(objSDR); // Load data into DataTable
                }
            }
        }

        if (dt.Rows.Count > 0)
        {
            // Map data to CityModel
            CityModel model = new CityModel();
            foreach (DataRow dr in dt.Rows)
            {
                model.CityID = Convert.ToInt32(dr["CityID"]);
                model.CityName = dr["CityName"].ToString();
                model.StateID = Convert.ToInt32(dr["StateID"]);
                model.CountryID = Convert.ToInt32(dr["CountryID"]);
                model.CityCode = dr["CityCode"].ToString();
                ViewBag.StateList = GetStateByCountryID(model.CountryID); // Load states for selected country
            }
             GetStatesByCountry(model.CountryID);
            return View("CityAddEdit", model); // Return populated model to view
        }
    }

    return View("CityAddEdit"); // For adding a new city
}
#endregion

#region Save
// Save action handles both insert and update operations
[HttpPost]
public IActionResult Save(CityModel modelCity)
{
    if (ModelState.IsValid)
    {
        string connectionstr = _configuration.GetConnectionString("ConnectionString");
        using (SqlConnection conn = new SqlConnection(connectionstr))
        {
            conn.Open();
            using (SqlCommand objCmd = conn.CreateCommand())
            {
                objCmd.CommandType = CommandType.StoredProcedure;

                // Choose procedure based on operation (insert or update)
                if (modelCity.CityID == null)
                {
                    objCmd.CommandText = "PR_LOC_City_Insert";
                }
                else
                {
                    objCmd.CommandText = "PR_LOC_City_Update";
                    objCmd.Parameters.Add("@CityID", SqlDbType.Int).Value = modelCity.CityID;
                }

                // Pass parameters
                objCmd.Parameters.Add("@CityName", SqlDbType.VarChar).Value = modelCity.CityName;
                objCmd.Parameters.Add("@CityCode", SqlDbType.VarChar).Value = modelCity.CityCode;
                objCmd.Parameters.Add("@StateID", SqlDbType.Int).Value = modelCity.StateID;
                objCmd.Parameters.Add("@CountryID", SqlDbType.Int).Value = modelCity.CountryID;

                objCmd.ExecuteNonQuery(); // Execute the query
            }
        }

        TempData["CityInsertMsg"] = "Record Saved Successfully"; // Success message
        return RedirectToAction("Index"); // Redirect to city listing
    }

    LoadCountryList(); // Reload dropdowns if validation fails
    return View("CityAddEdit", modelCity);
}
#endregion
```

---

### 4. **Dropdown Load Methods**

```csharp
#region LoadCountryList
// Load the dropdown list of countries
private void LoadCountryList()
{
    string connectionstr = _configuration.GetConnectionString("ConnectionString");
    DataTable dt = new DataTable();
    using (SqlConnection conn = new SqlConnection(connectionstr))
    {
        conn.Open();
        using (SqlCommand objCmd = conn.CreateCommand())
        {
            objCmd.CommandType = CommandType.StoredProcedure;
            objCmd.CommandText = "PR_LOC_Country_SelectComboBox";

            using (SqlDataReader objSDR = objCmd.ExecuteReader())
            {
                dt.Load(objSDR); // Load data into DataTable
            }
        }
    }

    // Map data to list
    List<CountryDropDownModel> countryList = new List<CountryDropDownModel>();
    foreach (DataRow dr in dt.Rows)
    {
        countryList.Add(new CountryDropDownModel
        {
            CountryID = Convert.ToInt32(dr["CountryID"]),
            CountryName = dr["CountryName"].ToString()
        });
    }
    ViewBag.CountryList = countryList; // Pass list to view
}
#endregion

#region GetStatesByCountry
// AJAX handler for loading states dynamically
[HttpPost]
public JsonResult GetStatesByCountry(int CountryID)
{
    List<StateDropDownModel> loc_State = GetStateByCountryID(CountryID); // Fetch states
    return Json(loc_State); // Return JSON response
}
#endregion

#region GetStateByCountryID
// Helper method to fetch states by country ID
public List<StateDropDownModel> GetStateByCountryID(int CountryID)
{
    string connectionstr = _configuration.GetConnectionString("ConnectionString");
    List<StateDropDownModel> loc_State = new List<StateDropDownModel>();

    using (SqlConnection conn = new SqlConnection(connectionstr))
    {
        conn.Open();
        using (SqlCommand objCmd = conn.CreateCommand())
        {
            objCmd.CommandType = CommandType.StoredProcedure;
            objCmd.CommandText = "PR_LOC_State_SelectComboBoxByCountryID";
            objCmd.Parameters.AddWithValue("@CountryID", CountryID);

            using (SqlDataReader objSDR = objCmd.ExecuteReader())
            {
                if (objSDR.HasRows)
                {
                    while (objSDR.Read())
                    {
                        loc_State.Add(new StateDropDownModel
                        {
                            StateID = Convert.ToInt32(objSDR["StateID"]),
                            StateName = objSDR["StateName"].ToString()
                        });
                    }
                }
            }
        }
    }

    return loc_State;
}
#endregion
```

---
✅ Add jQuery Script
To use jQuery across your views (e.g., for AJAX calls or DOM manipulation), include the following script in the _Layout.cshtml file:
```html
<!-- Add this just before the closing </body> tag -->
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.js"></script>
```

### 5. **City Add/Edit View**

```html
@{
    ViewData["Title"] = "City Add/Edit"; Layout = "~/Views/Shared/_Layout.cshtml";
}

@model CoffeeShop.Models.CityModel


<main id="main" class="main">
<div class="container">
    <div class="row">
        <div class="col-md-12">
            <div class="page-header">
                <h1>City Add/Edit</h1>
            </div>
        </div>
    </div>

    <div class="row">
        <div class="col-md-12">
            <div class="panel panel-default">
                <div class="panel-heading">
                    <h3 class="panel-title">City Add/Edit</h3>
                </div>
                <div class="panel-body">
                    <h4 class="text-success">@TempData["CityInsertMessage"]</h4>
                    <form class="form-horizontal"
                          role="form"
                          method="post"
                          asp-controller="City"
                          asp-action="Save">
                        <div asp-validation-summary="ModelOnly" class="text-danger"></div>
                            @Html.HiddenFor(x => x.CityID)

                        <div class="form-group">
                            <label for="CountryID" class="col-md-3 control-label"><span class="text-danger">*</span>Country Name</label>
                            <div class="col-md-9">
                                <select id="CountryID"
                                        name="CountryID"
                                        class="form-control"
                                        asp-for="CountryID">
                                    <option value="">Select Country</option>
                                    @foreach (var country in ViewBag.CountryList)
                                    {
                                        <option value="@country.CountryID">
                                            @country.CountryName
                                        </option>
                                    }
                                </select>
                                <span asp-validation-for="CountryID" class="text-danger"></span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="StateID" class="col-md-3 control-label"><span class="text-danger">*</span>State Name</label>
                            <div class="col-md-9">
                                <select id="StateID"
                                        name="StateID"
                                        class="form-control"
                                        asp-for="StateID">
                                    <option value="">Select State</option>
                                    @if (ViewBag.StateList != null)
                                    {
                                        foreach (var state in
                                                            ViewBag.StateList)
                                        {
                                            if (state.StateID == Model.StateID)
                                            {
                                                <option value="@state.StateID">@state.StateName</option>
                                            }
                                            else
                                            {
                                                <option value="@state.StateID">@state.StateName</option>
                                            }
                                        }
                                    }
                                </select>
                                <span asp-validation-for="StateID" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="form-group">
                            <label for="CityName" class="col-md-3 control-label"><span class="text-danger">*</span>City Name</label>
                            <div class="col-md-9">
                                <input type="text"
                                       id="CityName"
                                       name="CityName"
                                       class="form-control"
                                       asp-for="CityName"
                                       placeholder="Enter City Name" />
                                <span asp-validation-for="CityName" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="form-group">
                            <label for="CityCode" class="col-md-3 control-label"><span class="text-danger">*</span>City Code</label>
                            <div class="col-md-9">
                                <input type="text"
                                       id="CityCode"
                                       name="CityCode"
                                       class="form-control"
                                       asp-for="CityCode"
                                       placeholder="Enter City Code" />
                                <span asp-validation-for="CityCode" class="text-danger"></span>
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-md-offset-3 col-md-9">
                                <input type="submit" class="btn btn-success" value="Save" />
                                <a class="btn btn-default"
                                   asp-controller ="City"
                                   asp-action="Index">Cancel</a>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
</main>
@section Scripts {
    <script>
       $(document).ready(function () {
           $("#CountryID").change(function () {
               var countryId = $(this).val();
               if (countryId) {
                   $.ajax({
                       url: '@Url.Action("GetStatesByCountry", "City")',
                       type: "POST", // Changed to POST
                       data: { CountryID: countryId }, // Use 'CountryID' to match controller
                       success: function (data) {
                           $("#StateID")
                               .empty()
                               .append('<option value="">Select State</option>');
                           $.each(data, function (i, state) {
                               $("#StateID").append(
                                   '<option value="' +
                                   state.stateID +
                                   '">' +
                                   state.stateName +
                                   "</option>"
                               );
                           });
                           console.log(state.stateID);
                       },
                       error: function (xhr, status, error) {
                           console.error(error);
                       },
                   });
               } else {
                   $("#StateID").empty().append('<option value="">Select State</option>');
               }
           });
       });
   </script>
}
```

---
