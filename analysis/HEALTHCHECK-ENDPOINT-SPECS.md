# HealthCheck Endpoint Specifications Comparison

**Purpose**: Document the HealthCheck endpoints from both APIs for POC testing

---

## Original API (BargeOps.Web.Api_original)

### Endpoint 1: CheckHealth

**HTTP Method**: GET
**Path**: `/api/HealthCheck/CheckHealth`
**Full URL**: `https://localhost:44368/api/HealthCheck/CheckHealth`

**Authentication**:
- Type: API Key + Basic Authentication
- Attribute: `[ApiKeyBasicAuthentication]`
- Role Required: "Helm"

**Request**:
- No parameters
- Headers:
  - `Authorization: Basic {apiKey}`
  - `Content-Type: application/json`

**Response** (200 OK):
```json
{
  "ApplicationStatus": "Healthy",
  "DatabaseConnectionStatus": "Connected"
}
```

**Response Schema**:
- `ApplicationStatus`: string ("Healthy" | "Unhealthy")
- `DatabaseConnectionStatus`: string ("Connected" | "Disconnected")

**Implementation Notes**:
- Synchronous method
- Checks application health (always returns true)
- Checks database connection by opening SQL connection
- Returns anonymous object (not strongly typed response model)

**Source File**: `C:\Users\layden\Codebases\Web API\BargeOps.Web.Api_original\BargeOps.Web.Api_original\Controllers\HealthCheckController.cs:25`

---

## Redux API (BargeOps.Web.API_Redux)

### Endpoint 1: CheckHealth

**HTTP Method**: GET
**Path**: `/api/healthcheck/CheckHealth`
**Full URL**: `https://localhost:5001/api/healthcheck/CheckHealth`

**Authentication**:
- Type: Basic Authentication (ASP.NET Core)
- Scheme: "Basic"
- Role Required: "Helm"

**Request**:
- No parameters
- Headers:
  - `Authorization: Basic {apiKey}`
  - `Content-Type: application/json`

**Response** (200 OK):
```json
{
  "Status": "Healthy",
  "TotalDuration": "00:00:00.1234567",
  "Entries": {
    "database": {
      "Status": "Healthy",
      "Description": "Database is responsive",
      "Duration": "00:00:00.1234567",
      "Data": {}
    }
  }
}
```

**Response Schema**:
- `Status`: string ("Healthy" | "Unhealthy")
- `TotalDuration`: TimeSpan (duration of health checks)
- `Entries`: object (dictionary of health check results)
  - Each entry contains: Status, Description, Duration, Data

**Implementation Notes**:
- Asynchronous method (async/await)
- Uses ASP.NET Core `HealthCheckService` (dependency injection)
- More comprehensive health check with timing and multiple entries
- Modern ASP.NET Core patterns

**Source File**: `C:\Users\layden\Codebases\Web API\BargeOps.Web.API_Redux\BargeOps.Web.API_Redux\src\BargeOps.Web.Api\Controllers\HealthCheckController.cs:29`

---

### Endpoint 2: Ping (NEW in Redux)

**HTTP Method**: GET
**Path**: `/api/healthcheck/ping`
**Full URL**: `https://localhost:5001/api/healthcheck/ping`

**Authentication**:
- Type: Basic Authentication (ASP.NET Core)
- Scheme: "Basic"
- Role Required: "Helm"

**Request**:
- No parameters
- Headers:
  - `Authorization: Basic {apiKey}`
  - `Content-Type: application/json`

**Response** (200 OK):
```json
{
  "Status": "OK",
  "Timestamp": "2025-11-10T12:00:00.000Z"
}
```

**Response Schema**:
- `Status`: string ("OK")
- `Timestamp`: DateTime (UTC timestamp)

**Implementation Notes**:
- Synchronous method
- Simple health check (no database check)
- Returns current UTC timestamp
- Lightweight alternative to full CheckHealth

**Source File**: `C:\Users\layden\Codebases\Web API\BargeOps.Web.API_Redux\BargeOps.Web.API_Redux\src\BargeOps.Web.Api\Controllers\HealthCheckController.cs:49`

---

## Key Differences (Breaking Changes)

### 1. Response Structure Changes ⚠️ BREAKING CHANGE
**Original**:
```json
{
  "ApplicationStatus": "Healthy",
  "DatabaseConnectionStatus": "Connected"
}
```

**Redux**:
```json
{
  "Status": "Healthy",
  "TotalDuration": "00:00:00.1234567",
  "Entries": { ... }
}
```

**Impact**:
- Field names changed: `ApplicationStatus` → `Status`
- `DatabaseConnectionStatus` removed (now in `Entries`)
- New fields added: `TotalDuration`, `Entries`
- **Clients parsing the original response will break!**

---

### 2. URL Case Sensitivity ⚠️ POTENTIAL ISSUE
**Original**: `/api/HealthCheck/CheckHealth` (PascalCase)
**Redux**: `/api/healthcheck/CheckHealth` (lowercase controller)

**Impact**:
- Case-sensitive clients/systems may fail
- URLs are technically different

---

### 3. New Endpoint Added ✅ ENHANCEMENT
**Redux Only**: `/api/healthcheck/ping`

**Impact**:
- New functionality (not breaking)
- Provides lightweight health check option

---

### 4. Implementation Pattern Changes
| Aspect | Original | Redux |
|--------|----------|-------|
| **Async** | No (synchronous) | Yes (async/await) |
| **DI** | No (direct instantiation) | Yes (HealthCheckService) |
| **Framework** | .NET Framework WebAPI | ASP.NET Core |
| **Health Check** | Manual SQL connection | Built-in HealthCheckService |

---

## Test Strategy for POC

### Test Scenarios

#### Scenario 1: Original API CheckHealth
- Call `GET https://localhost:44368/api/HealthCheck/CheckHealth`
- Verify 200 OK response
- Verify response contains `ApplicationStatus` and `DatabaseConnectionStatus`
- Store response for comparison

#### Scenario 2: Redux API CheckHealth
- Call `GET https://localhost:5001/api/healthcheck/CheckHealth`
- Verify 200 OK response
- Verify response contains `Status`, `TotalDuration`, `Entries`
- Store response for comparison

#### Scenario 3: Redux API Ping (New Endpoint)
- Call `GET https://localhost:5001/api/healthcheck/ping`
- Verify 200 OK response
- Verify response contains `Status` and `Timestamp`
- Verify timestamp is recent (within last minute)

#### Scenario 4: Response Comparison
- Compare Original vs Redux CheckHealth responses
- Document schema differences
- Identify breaking changes
- Classify as: Exact Match, Schema Match, Breaking Change

### Expected Results

| Test | Expected Status | Expected Outcome |
|------|-----------------|------------------|
| Original CheckHealth | 200 OK | `{ ApplicationStatus, DatabaseConnectionStatus }` |
| Redux CheckHealth | 200 OK | `{ Status, TotalDuration, Entries }` |
| Redux Ping | 200 OK | `{ Status, Timestamp }` |
| Comparison | N/A | **BREAKING CHANGE DETECTED** |

### Success Criteria for POC

1. ✅ Both APIs start successfully on different ports
2. ✅ Original API responds on port 44368
3. ✅ Redux API responds on port 5001
4. ✅ Both endpoints return 200 OK
5. ⚠️ Response structures are different (document difference)
6. ✅ Redux ping endpoint works (new feature)
7. ✅ Comparison logic successfully identifies breaking changes

---

*This specification will be used to generate POC Postman tests and validate the comparison approach.*
