# Complete Endpoint Specifications: Original vs Redux API

**Generated**: 2025-11-11
**Purpose**: Detailed comparison of all 9 remaining endpoints across both API versions
**Status**: Ready for Post

man collection generation

---

## Table of Contents

1. [ComtracController (2 endpoints)](#comtraccontroller)
2. [MatchTracksIntegrationController (2 endpoints)](#matchtracksintegrationcontroller)
3. [HelmIntegrationController (4 endpoints)](#helmintegrationcontroller)
4. [Breaking Changes Summary](#breaking-changes-summary)

---

## ComtracController

### 1. GET GetNewComtracSchedules

#### Original API
```
URL: GET /api/ComTrac/GetNewComtracSchedules?clientId={string}
Authorization: Basic Auth, Role: "ComTrac"
```

**Request**:
- Query Parameter: `clientId` (string, required)

**Response** (200 OK):
```json
[
  {
    "BargeTripID": 12345,
    "BargeNum": "ABC123",
    "LoadLocation": "Location Name",
    "UnloadLocation": "Destination",
    "Commodity": "Coal",
    ...
  }
]
```
Type: `ComTracScheduleInfo[]`

**Error Responses**:
- 400: Bad Request
- 401: Unauthorized
- 500: Internal Server Error

#### Redux API
```
URL: GET /api/comtrac/GetNewComtracSchedules?clientId={string}
Authorization: Basic Auth, Role: "ComTrac"
```

**Request**:
- Query Parameter: `clientId` (string, required)

**Response** (200 OK):
```json
[
  {
    "BargeTripID": 12345,
    "BargeNum": "ABC123",
    "LoadLocation": "Location Name",
    "UnloadLocation": "Destination",
    "Commodity": "Coal",
    ...
  }
]
```
Type: `ComTracScheduleInfo[]` (same as Original)

**Error Responses**:
- 400: Bad Request
- 401: Unauthorized
- 500: Internal Server Error (exception thrown, not wrapped)

#### ⚠️ Breaking Changes
1. **URL Case Change**: `/ComTrac/` → `/comtrac/` (lowercase)
2. **Error Handling**: Original returns wrapped ApiResponse, Redux throws exceptions (handled by middleware)

---

### 2. POST SubmitComtracBargeUnloadData

#### Original API
```
URL: POST /api/ComTrac/SubmitComtracBargeUnloadData
Authorization: Basic Auth, Role: "ComTrac"
Content-Type: application/json
```

**Request Body**:
```json
{
  "BargeTripID": 12345,
  "UnloadDateTime": "2025-11-10T14:30:00Z",
  "UnloadTons": 1500.75
}
```

**Validation**:
- `BargeTripID`: Required, must be > 0
- `UnloadDateTime`: Required, valid DateTime
- `UnloadTons`: Required, must be > 0

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Unload data submitted successfully."
}
```

**Error Responses**:
- 400: Bad Request (validation failed or SqlException not 51000)
- 401: Unauthorized
- 404: Not Found (SqlException 51000 - custom error from stored procedure)
- 500: Internal Server Error

#### Redux API
```
URL: POST /api/comtrac/SubmitComtracBargeUnloadData
Authorization: Basic Auth, Role: "ComTrac"
Content-Type: application/json
```

**Request Body**: Same as Original

**Response** (200 OK):
```
Empty OkResult (no body)
```

**Error Responses**:
- 400: Bad Request (ValidationProblem with ModelState)
- 401: Unauthorized
- 500: Internal Server Error (all SqlExceptions thrown, handled by middleware)

#### ⚠️ Breaking Changes
1. **URL Case Change**: `/ComTrac/` → `/comtrac/`
2. **Response Body**: Original returns `{Success, Message}`, Redux returns empty `OkResult`
3. **Error Handling**: Redux throws exceptions instead of returning 404 for SqlException 51000
4. **Validation Errors**: Original returns custom error message, Redux returns ValidationProblem format

---

## MatchTracksIntegrationController

### 3. GET GetBargeTripList

#### Original API
```
URL: GET /api/MatchTracksIntegration/GetBargeTripList
Authorization: Basic Auth, Role: "MatchTracks"
```

**Request**:
- NO query parameters
- Reads `CustomerID` from Claims in JWT token

**Implementation**:
```csharp
var identity = this.User.Identity as ClaimsIdentity;
var customerIDClaim = identity.Claims.Where(x => x.Type == "CustomerID").FirstOrDefault();
```

**Response** (200 OK):
```json
[
  {
    "BargeNum": "ABC123",
    "TicketID": 54321,
    "Commodity": "Coal",
    "CommodityID": 10,
    "CommodityGroup": "Dry Bulk",
    "LoadLocation": "Port A",
    "LoadLocationID": 100,
    "UnloadLocation": "Port B",
    "UnloadLocationID": 200,
    "LoadPlacementDateTime": "2025-11-10T08:00:00Z",
    "LoadReleasedDateTime": "2025-11-10T10:00:00Z",
    "LoadQuantity": 1500.50,
    "LoadUnits": "tons",
    "LoadDepartureDateTime": "2025-11-10T11:00:00Z",
    "CurrentPositionRiver": "Mississippi",
    "CurrentPositionMile": "Mile 235.6",
    "CurrentPositionLastUpdatedDateTime": "2025-11-10T14:30:00Z",
    "CurrentLocationID": 150,
    "CurrentLocation": "Memphis",
    "UnloadPlacementDateTime": null,
    "PONumber": "PO-12345",
    "POVendor": "Vendor Inc",
    "DestinationLocation": "Final Destination",
    "DestinationLocationID": 300,
    "DestinationScheduledDateTime": "2025-11-12T09:00:00Z"
  }
]
```
Type: `IList<BargeTripListModel>` (35+ fields)

**Error Responses**:
- 400: Bad Request (if CustomerID claim missing or invalid)
- 401: Unauthorized
- 500: Internal Server Error

#### Redux API
```
URL: GET /api/matchtracksintegration/GetBargeTripList?customerID={int}
Authorization: Basic Auth, Role: "MatchTracks"
```

**Request**:
- Query Parameter: `customerID` (int, required)

**Implementation**:
```csharp
public async Task<IActionResult> GetBargeTripList([FromQuery] int customerID)
```

**Response** (200 OK): Same structure as Original

**Error Responses**:
- 400: Bad Request
- 401: Unauthorized
- 500: Internal Server Error (exception thrown)

#### ⚠️ Breaking Changes
1. **🚨 CRITICAL**: Parameter Source Changed
   - Original: Reads `CustomerID` from Claims (no parameter)
   - Redux: Requires `customerID` query parameter
2. **URL Case Change**: `/MatchTracksIntegration/` → `/matchtracksintegration/`
3. **Error Handling**: Original returns wrapped errors, Redux throws exceptions

**Migration Impact**: Clients MUST add `?customerID={value}` to URL

---

### 4. POST SubmitBargeLoadData

#### Original API
```
URL: POST /api/MatchTracksIntegration/SubmitBargeLoadData
Authorization: Basic Auth, Role: "MatchTracks"
Content-Type: application/json
```

**Request Body**:
```json
{
  "TicketID": 54321,
  "StartLoadingDateTime": "2025-11-10T08:00:00Z",
  "CompleteLoadingDateTime": "2025-11-10T10:00:00Z",
  "ReleaseDateTime": "2025-11-10T11:00:00Z",
  "LoadQuantity": 1500.75
}
```

**Validation**:
- All fields required
- DateTime fields must be valid ISO 8601
- LoadQuantity must be positive decimal
- Validates ticket ownership via CustomerID claim

**Response** (200 OK):
```json
{
  "Success": true
}
```

**Error Responses**:
- 400: Bad Request (validation failed, CustomerID claim missing, or ticket ownership mismatch)
- 401: Unauthorized
- 500: Internal Server Error

#### Redux API
```
URL: POST /api/matchtracksintegration/SubmitBargeLoadData
Authorization: Basic Auth, Role: "MatchTracks"
Content-Type: application/json
```

**Request Body**: Same as Original

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Barge load data submitted successfully."
}
```

**Response** (400 Bad Request):
```json
{
  "Success": false,
  "Message": "Failed to submit barge load data." | "Invalid model data."
}
```

**Error Responses**:
- 400: Bad Request (validation or business logic failure)
- 401: Unauthorized
- 500: Internal Server Error (Database error or general exception)

#### ⚠️ Breaking Changes
1. **URL Case Change**: `/MatchTracksIntegration/` → `/matchtracksintegration/`
2. **Response Format**: Redux adds `Message` field to success response
3. **Error Messages**: Different error handling patterns
4. **Ticket Ownership Validation**: Original validates via Claims, Redux may handle differently

---

## HelmIntegrationController

### 5. POST BargeDamageRepairUpdate

#### Original API
```
URL: POST /api/HelmIntegration/BargeDamageRepairUpdate
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body** (raw JSON validated against config):
```json
{
  "bargenum": "ABC123",
  "uscgnum": "1234567",
  "repairstatus": "In Progress",
  "IsCargoDamaged": true,
  "IsDryDocked": false,
  "IsLeaker": false,
  "IsRepairScheduled": true,
  "DamageLevel": "*",
  "IsDamaged": true,
  "DamageNote": "Minor damage to hull"
}
```

**Special Validation**:
- Reads required parameter list from `ConfigurationManager.AppSettings["BargeDamageRepairUpdateParameterList"]`
- Validates each parameter exists in JSON (case-insensitive)
- Returns 400 if any parameters missing
- Creates alert in BargeOps system if validation fails

**Response** (200 OK):
```json
{
  "Success": true
}
```

**Error Responses**:
- 400: Bad Request (missing required parameters)
- 401: Unauthorized
- 500: Internal Server Error

#### Redux API
```
URL: POST /api/helmintegration/BargeDamageRepairUpdate
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body** (typed DTO):
```json
{
  "BargeNum": "ABC123",
  "UscgNum": "1234567",
  "RepairStatus": "In Progress",
  "IsCargoDamaged": true,
  "IsDryDocked": false,
  "IsLeaker": false,
  "IsRepairScheduled": true,
  "DamageLevel": "*",
  "IsDamaged": true,
  "DamageNote": "Minor damage to hull"
}
```

**Validation**:
- ModelState validation via attributes on `BargeDamageRepairUpdateDto`
- No configuration-based parameter validation

**Response** (200 OK):
```json
{
  "Success": true
}
```

**Error Responses**:
- 400: Bad Request (ModelState validation)
- 401: Unauthorized
- 500: Internal Server Error

#### ⚠️ Breaking Changes
1. **🚨 CRITICAL**: Field Name Casing Changed
   - Original: `lowercase` fields (`bargenum`, `uscgnum`, `repairstatus`)
   - Redux: `PascalCase` fields (`BargeNum`, `UscgNum`, `RepairStatus`)
2. **URL Case Change**: `/HelmIntegration/` → `/helmintegration/`
3. **Validation Approach**: Configuration-based → Model-based
4. **No Alert Creation**: Redux doesn't create BargeOps alerts on validation failure

**Migration Impact**: ALL field names must be updated in client code

---

### 6. POST BargeDamageUpdate

#### Original API
```
URL: POST /api/HelmIntegration/BargeDamageUpdate
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body**:
```json
{
  "BargeNum": "ABC123",
  "BargeUscgNum": "1234567",
  "DamageLevel": "**",
  "DamageNote": "Moderate hull damage",
  "IsCargoDamaged": false,
  "IsLeaker": false,
  "IsDryDocked": false,
  "IsRepairScheduled": false
}
```

**Validation**:
- `DamageLevel`: Must be one of: `""`, `"*"`, `"**"`, `"***"`, `"DNRL"` (ValidValues attribute)
- All boolean fields default to false if not provided

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Update successful."
}
```

**Error Responses**:
- 400: Bad Request (validation or SqlException not 51000)
- 401: Unauthorized
- 404: Not Found (SqlException 51000)
- 500: Internal Server Error

#### Redux API
```
URL: POST /api/helmintegration/BargeDamageUpdate
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body**: Same as Original

**Response** (200 OK):
```json
{
  "Success": true
}
```

**Error Responses**:
- 400: Bad Request
- 401: Unauthorized
- 500: Internal Server Error (all exceptions thrown)

#### ⚠️ Breaking Changes
1. **URL Case Change**: `/HelmIntegration/` → `/helmintegration/`
2. **Response Message**: Original includes "Update successful" message, Redux does not
3. **Error Handling**: Redux doesn't return 404 for SqlException 51000, throws 500 instead

---

### 7. POST FormComplete

#### Original API
```
URL: POST /api/HelmIntegration/FormComplete
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body**:
```json
{
  "FormType": "Drill",
  "VesselUscgNum": "1234567",
  "StartDateTime": "2025-11-10T08:00:00Z",
  "CompletedDateTime": "2025-11-10T09:00:00Z",
  "PersonnelNames": "John Doe,Jane Smith,Bob Johnson",
  "InstructorName": "Captain Smith",
  "FormNumber": "FORM-001"
}
```

**Validation**:
- `FormType`: Must be `"Drill"` or `"SafetyMeeting"` (ValidValues attribute)
- `PersonnelNames`: Comma-separated list of names
- All DateTime fields must be valid ISO 8601

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Update successful."
}
```

**Error Responses**:
- 400: Bad Request (validation)
- 401: Unauthorized
- 404: Not Found (NotFoundException)
- 500: Internal Server Error

#### Redux API
```
URL: POST /api/helmintegration/FormComplete
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body**: Same as Original

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Update successful."
}
```

**Error Responses**:
- 400: Bad Request
- 401: Unauthorized
- 500: Internal Server Error (NotFoundException throws 500, not 404)

#### ⚠️ Breaking Changes
1. **URL Case Change**: `/HelmIntegration/` → `/helmintegration/`
2. **Error Handling**: Redux throws 500 for NotFoundException instead of 404

---

### 8. POST InventoryReadingAdd

#### Original API
```
URL: POST /api/HelmIntegration/InventoryReadingAdd
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body**:
```json
{
  "ReadingType": "Fuel",
  "VesselUscgNum": "1234567",
  "ReadingDateTime": "2025-11-10T14:30:00Z",
  "ReadingUnit": "liters",
  "HelmReadingId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "ReadingValue": 1234.56
}
```

**Validation**:
- `ReadingType`: Must be one of 8 values: `"Fuel"`, `"Lube"`, `"Slop"`, `"WaterPercent"`, `"GearPercent"`, `"Gas"`, `"HydraulicOil"`, `"Coolant"`
- `VesselUscgNum`: StringLength(10)
- `ReadingUnit`: StringLength(10)
- `ReadingValue`: DecimalPrecision(7, 2) - max 7 total digits, 2 after decimal
- `HelmReadingId`: Valid GUID

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Update successful."
}
```

**Error Responses**:
- 400: Bad Request (validation)
- 401: Unauthorized
- 404: Not Found (NotFoundException)
- 500: Internal Server Error

#### Redux API
```
URL: POST /api/helmintegration/InventoryReadingAdd
Authorization: Basic Auth, Role: "Helm"
Content-Type: application/json
```

**Request Body**: Same as Original

**Response** (200 OK):
```json
{
  "Success": true,
  "Message": "Update successful."
}
```

**Error Responses**:
- 400: Bad Request
- 401: Unauthorized
- 500: Internal Server Error (NotFoundException throws 500, not 404)

#### ⚠️ Breaking Changes
1. **URL Case Change**: `/HelmIntegration/` → `/helmintegration/`
2. **Error Handling**: Redux throws 500 for NotFoundException instead of 404
3. **Complex Validation**: DecimalPrecision and StringLength may behave differently between frameworks

---

## Breaking Changes Summary

### Critical (Requires Code Changes)

| # | Endpoint | Change | Impact |
|---|----------|--------|--------|
| 1 | **GetBargeTripList** | CustomerID from Claims → Query Parameter | ⚠️ **HIGH** - Clients must add `?customerID={value}` |
| 2 | **BargeDamageRepairUpdate** | Field casing: `lowercase` → `PascalCase` | ⚠️ **HIGH** - All field names must change |
| 3 | **SubmitComtracBargeUnloadData** | Response body: `{Success, Message}` → Empty | ⚠️ **MEDIUM** - Clients parsing response will break |

### Major (Configuration/Behavior Changes)

| # | Endpoint | Change | Impact |
|---|----------|--------|--------|
| 4 | **All HelmIntegration** | 404 errors → 500 errors | ⚠️ **MEDIUM** - Error handling logic needs update |
| 5 | **SubmitComtracBargeUnloadData** | SqlException 51000: 404 → 500 | ⚠️ **MEDIUM** - Lost specific error code handling |
| 6 | **BargeDamageRepairUpdate** | Config-based validation → Model validation | ⚠️ **MEDIUM** - Different validation behavior |

### Minor (Cosmetic/Non-Breaking)

| # | Change | Impact |
|---|--------|--------|
| 7 | URL casing: `/ComTrac/` → `/comtrac/` | ⚠️ **LOW** - May affect case-sensitive clients |
| 8 | URL casing: `/HelmIntegration/` → `/helmintegration/` | ⚠️ **LOW** - May affect case-sensitive clients |
| 9 | URL casing: `/MatchTracksIntegration/` → `/matchtracksintegration/` | ⚠️ **LOW** - May affect case-sensitive clients |
| 10 | Error messages format changed | ⚠️ **LOW** - Error parsing may need updates |

---

## Test Data Recommendations

### ComtracController
- **Valid BargeTripID**: 12345 (active trip)
- **Invalid BargeTripID**: 99999 (should return 404/500)
- **clientId**: "TEST_CLIENT" or read from environment

### MatchTracksIntegrationController
- **Valid customerID**: 999 (from InMemoryApiKeyStore claims)
- **Valid TicketID**: Must exist for customerID in database
- **DateTime format**: ISO 8601 with timezone (e.g., `2025-11-10T14:30:00Z`)

### HelmIntegrationController
- **Valid BargeNum**: "TEST001"
- **Valid VesselUscgNum**: "1234567" (7 digits)
- **DamageLevel values**: `""`, `"*"`, `"**"`, `"***"`, `"DNRL"`
- **FormType values**: `"Drill"`, `"SafetyMeeting"`
- **ReadingType values**: `"Fuel"`, `"Lube"`, `"Slop"`, `"WaterPercent"`, `"GearPercent"`, `"Gas"`, `"HydraulicOil"`, `"Coolant"`
- **ReadingValue**: Max 12345.67 (7 digits total, 2 decimal)

---

## Next Steps

1. Generate Postman collections for each controller
2. Create test requests with valid sample data
3. Add comprehensive test assertions
4. Execute against Redux API (running on localhost:5001)
5. Document actual responses
6. Compare with Original API (when available)

---

**Document Status**: ✅ Complete - Ready for Postman Generation
**Breaking Changes Identified**: 10 total (3 Critical, 3 Major, 4 Minor)
**Test Coverage Target**: 9/9 endpoints (100%)
