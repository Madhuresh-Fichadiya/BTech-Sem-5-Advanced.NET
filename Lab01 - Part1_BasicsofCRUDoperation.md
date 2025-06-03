# City Table CRUD Operations in ASP.NET Core MVC

This project demonstrates how to perform CRUD operations for the `City` table with cascading dropdown functionality in ASP.NET Core MVC.

## Table Schema

### Country Table

```sql
CREATE TABLE Country (
    CountryID INT PRIMARY KEY IDENTITY(1,1),
    CountryName NVARCHAR(100) NOT NULL,
    CountryCode NVARCHAR(10) NOT NULL,
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL
);
```

### State Table

```sql
CREATE TABLE State (
    StateID INT PRIMARY KEY IDENTITY(1,1),
    CountryID INT NOT NULL,
    StateName NVARCHAR(100) NOT NULL,
    StateCode NVARCHAR(10),
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL,
    FOREIGN KEY (CountryID) REFERENCES Country(CountryID)
);
```

### City Table

```sql
CREATE TABLE City (
    CityID INT PRIMARY KEY IDENTITY(1,1),
    StateID INT NOT NULL,
    CountryID INT NOT NULL,
    CityName NVARCHAR(100) NOT NULL,
    CityCode NVARCHAR(10),
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL,
    FOREIGN KEY (StateID) REFERENCES State(StateID),
    FOREIGN KEY (CountryID) REFERENCES Country(CountryID)
);
```

---

## Data

Here is the sample data to insert into the `Country`, `State`, and `City` tables after recreating them.

---

### Insert Data for `Country`
```sql
INSERT INTO Country (CountryName, CountryCode, CreatedDate) VALUES
('United States', 'US', GETDATE()),
('India', 'IN', GETDATE()),
('Australia', 'AU', GETDATE()),
('Canada', 'CA', GETDATE()),
('United Kingdom', 'UK', GETDATE()),
('Germany', 'DE', GETDATE()),
('France', 'FR', GETDATE()),
('Japan', 'JP', GETDATE()),
('China', 'CN', GETDATE()),
('Brazil', 'BR', GETDATE());
```

---

### Insert Data for `State`
```sql
INSERT INTO State (StateName, StateCode, CountryID, CreatedDate) VALUES
('California', 'CA', 1, GETDATE()),
('Texas', 'TX', 1, GETDATE()),
('Gujarat', 'GJ', 2, GETDATE()),
('Maharashtra', 'MH', 2, GETDATE()),
('New South Wales', 'NSW', 3, GETDATE()),
('Victoria', 'VIC', 3, GETDATE()),
('Ontario', 'ON', 4, GETDATE()),
('Quebec', 'QC', 4, GETDATE()),
('England', 'ENG', 5, GETDATE()),
('Scotland', 'SCT', 5, GETDATE());
```

---

### Insert Data for `City`
```sql
INSERT INTO City (CityName, CityCode, StateID, CountryID, CreatedDate) VALUES
('Los Angeles', 'LA', 1, 1, GETDATE()),
('Houston', 'HOU', 2, 1, GETDATE()),
('Ahmedabad', 'AMD', 3, 2, GETDATE()),
('Mumbai', 'MUM', 4, 2, GETDATE()),
('Sydney', 'SYD', 5, 3, GETDATE()),
('Melbourne', 'MEL', 6, 3, GETDATE()),
('Toronto', 'TOR', 7, 4, GETDATE()),
('Montreal', 'MTL', 8, 4, GETDATE()),
('London', 'LDN', 9, 5, GETDATE()),
('Edinburgh', 'EDI', 10, 5, GETDATE());
```

---
## Stored Procedures

### Get All Cities

```sql
CREATE PROCEDURE [dbo].[PR_LOC_City_SelectAll]
AS 
SELECT
		[dbo].[City].[CityID],
		[dbo].[City].[StateID],
		[dbo].[Country].CountryID,
		[dbo].[Country].[CountryName],
		[dbo].[State].[StateName],
		[dbo].[State].[StateCode],
		[dbo].[City].[CreatedDate],
		[dbo].[City].[ModifiedDate],
		[dbo].[City].[CityName],
		[dbo].[City].[CityCode]
		
FROM [dbo].[City]
LEFT OUTER JOIN [dbo].[State]
ON [dbo].[State].[StateID] = [dbo].[City].[StateID]
LEFT OUTER JOIN [dbo].[Country]
ON [dbo].[Country].[CountryID] = [dbo].[State].[CountryID]
```

### Get City by ID

```sql
CREATE PROCEDURE PR_LOC_City_SelectByPK
    @CityID INT
AS
BEGIN
    SELECT CityID, CityName, StateID, CountryID, CityCode
    FROM City
    WHERE CityID = @CityID
END
```

### Insert City

```sql
CREATE PROCEDURE PR_LOC_City_Insert
    @CityName NVARCHAR(100),
    @CityCode NVARCHAR(10),
    @StateID INT,
    @CountryID INT
AS
BEGIN
    INSERT INTO City (CityName, CityCode, StateID, CountryID, CreatedDate)
    VALUES (@CityName, @CityCode, @StateID, @CountryID, GETDATE());
END
```

### Update City

```sql
CREATE PROCEDURE PR_LOC_City_Update
    @CityID INT,
    @CityName NVARCHAR(100),
    @CityCode NVARCHAR(10),
    @StateID INT,
    @CountryID INT
AS
BEGIN
    UPDATE City
    SET CityName = @CityName,
        CityCode = @CityCode,
        StateID = @StateID,
        CountryID = @CountryID,
        ModifiedDate = GETDATE()
    WHERE CityID = @CityID;
END
```

### Delete City

```sql
CREATE PROCEDURE PR_LOC_City_Delete
    @CityID INT
AS
BEGIN
    DELETE FROM City
    WHERE CityID = @CityID
END
```

---
### Get All Countries

```sql
CREATE PROCEDURE [dbo].[PR_LOC_Country_SelectComboBox]
AS 
SELECT
    COUNTRYID,
    COUNTRYNAME
FROM COUNTRY
ORDER BY COUNTRYNAME
```

---

### Get States by Country ID

```sql
CREATE PROCEDURE [dbo].[PR_LOC_State_SelectComboBoxByCountryID]
@CountryID INT
AS 
SELECT
    [dbo].[State].[StateID],
    [dbo].[State].[StateName]
FROM [dbo].[State]
WHERE [dbo].[State].[CountryID] = @CountryID
```

---

### Example of Using the Stored Procedures

- **Get All Countries:**
   To retrieve the list of all countries, you can call the `PR_LOC_Country_SelectComboBox` stored procedure, which will return the `CountryID` and `CountryName` for each country, ordered by `CountryName`.

   ```sql
   EXEC [dbo].[PR_LOC_Country_SelectComboBox]
   ```

- **Get States by Country ID:**
   To retrieve the list of states for a specific country, you can call the `PR_LOC_State_SelectComboBoxByCountryID` stored procedure, passing the `CountryID` as a parameter.

   ```sql
   EXEC [dbo].[PR_LOC_State_SelectComboBoxByCountryID @CountryID = 1]
   ```

---

### Usage Example in a Project

1. **Dropdown Population for Country:**
   When loading a form with country selection, you can use the `PR_LOC_Country_SelectComboBox` stored procedure to populate a country dropdown list.

2. **Dropdown Population for State:**
   When a country is selected, you can use the `PR_LOC_State_SelectComboBoxByCountryID` stored procedure to dynamically load states based on the selected country.

---

## Model Code

### CityModel.cs

```csharp
public class CityModel
{
    public int? CityID { get; set; }
    [Required]
    [DisplayName("City Name")]
    public string CityName { get; set; }
    [Required]
    [DisplayName("Country Name")]
    public int CountryID { get; set; }
    [Required]
    [DisplayName("State Name")]
    public int StateID { get; set; }
    [Required]
    [DisplayName("City Code")]
    public string CityCode { get; set; }
}
```

---
## Configure Database Connection

Open the `appsettings.json` file and add your SQL Server connection string:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=YOUR_DATABASE_NAME;Trusted_Connection=True;Encrypt=False;"
  }
}
```

Replace `YOUR_SERVER_NAME` and `YOUR_DATABASE_NAME` with your database details.

---

## Controller Code

### CityController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Data.SqlClient;
using System.Data;
using CoffeeShop.Models;

namespace CoffeeShop.Controllers
{
    public class CityController : Controller
    {
        private readonly IConfiguration _configuration;

        #region configuration
        public CityController(IConfiguration configuration)
        {
            _configuration = configuration;
        }
        #endregion

        #region Index
        public IActionResult Index()
        {
            string connectionstr = this._configuration.GetConnectionString("ConnectionString");
            //PrePare a connection
            DataTable dt = new DataTable();
            SqlConnection conn = new SqlConnection(connectionstr);
            conn.Open();

            //Prepare a Command
            SqlCommand objCmd = conn.CreateCommand();
            objCmd.CommandType = CommandType.StoredProcedure;
            objCmd.CommandText = "PR_LOC_City_SelectAll";

            SqlDataReader objSDR = objCmd.ExecuteReader();
            dt.Load(objSDR);
            conn.Close();
            return View("Index", dt);
        }
        #endregion

        #region Delete
        public IActionResult Delete(int CityID)
        {
            string connectionstr = _configuration.GetConnectionString("ConnectionString");
            using (SqlConnection conn = new SqlConnection(connectionstr))
            {
                conn.Open();
                using (SqlCommand sqlCommand = conn.CreateCommand())
                {
                    sqlCommand.CommandType = CommandType.StoredProcedure;
                    sqlCommand.CommandText = "PR_LOC_City_Delete";
                    sqlCommand.Parameters.AddWithValue("@CityID", CityID);
                    sqlCommand.ExecuteNonQuery();
                }
            }
            return RedirectToAction("Index");
        }
        #endregion
    }
}


```

---

## Views

### Index.cshtml

```html
@using System.Data
@{
    ViewData["Title"] = "City List";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<main id="main" class="main">
    <div class="pagetitle">
        <h1>City</h1>
        <nav>
            <ol class="breadcrumb">
                <li class="breadcrumb-item">
                    <a asp-controller="HomeMaster" asp-action="Index">
                        <i class="fa fa-home"></i>
                    </a>
                </li>
                <li class="breadcrumb-item active" aria-current="page">City List</li>
            </ol>
        </nav>
        <div class="d-flex justify-content-end align-items-center">
            <a class="btn btn-outline-primary" asp-controller="City" asp-action="CityAddEdit">
                <i class="bi bi-plus-lg"></i>&nbsp;Add City
            </a>
        </div>
    </div><!-- End Page Title -->

    @if (TempData["CityInsertMsg"] != null)
    {
        <div class="alert alert-success">
            @TempData["CityInsertMsg"]
        </div>
    }

    <div class="mb-3">
        <input type="text" class="form-control" id="citySearch" placeholder="Search Any">
    </div>

    <table class="table table-hover table-header-fixed">
        <thead>
            <tr>
                <th scope="col">City Name</th>
                <th scope="col">State Name</th>
                <th scope="col">Country Name</th>
                <th class="text-center">Actions</th>
            </tr>
        </thead>
        <tbody id="cityTable">
            @foreach (DataRow row in Model.Rows)
            {
                <tr>
                    <td>@row["CityName"]</td>
                    <td>@row["StateName"]</td>
                    <td>@row["CountryName"]</td>
                    <td class="text-center">
                        <a class="btn btn-outline-success btn-xs" asp-controller="City" asp-action="CityAddEdit" asp-route-CityID="@row["CityID"]">
                            <i class="bi bi-pencil-fill"></i>
                        </a>
                        <a class="btn btn-outline-danger btn-xs" asp-controller="City" asp-action="Delete" asp-route-CityID="@row["CityID"]" onclick="return confirm('Are you sure you want to delete this city?');">
                            <i class="bi bi-x"></i>
                        </a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
</main>

@section Scripts {
    <script>
        $(document).ready(function () {
            $("#citySearch").on("keyup", function () {
                var value = $(this).val().toLowerCase();
                $("#cityTable tr").filter(function () {
                    $(this).toggle($(this).text().toLowerCase().indexOf(value) > -1);
                });
            });
        });
    </script>
}

```

## Folder Structure

```
/Models
    CityModel.cs
/Controllers
    CityController.cs
/Views
    /City
        Index.cshtml
        CityAddEdit.cshtml
```
