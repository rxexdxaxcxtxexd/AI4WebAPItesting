# Migration Guide: Original API to Redux API

**Version:** 1.0
**Date:** 2025-11-14
**Target Audience:** Client Application Developers
**Prerequisite:** Read BREAKING-CHANGES-REPORT.md first

---

## Table of Contents

1. [Pre-Migration Checklist](#pre-migration-checklist)
2. [Step-by-Step Migration Instructions](#step-by-step-migration-instructions)
3. [Code Update Examples](#code-update-examples)
4. [Configuration Changes](#configuration-changes)
5. [Testing Procedures](#testing-procedures)
6. [Rollback Strategy](#rollback-strategy)
7. [Common Issues & Solutions](#common-issues--solutions)

---

## Pre-Migration Checklist

### Before You Begin

⚠️ **CRITICAL:** Do NOT attempt migration until ALL pre-migration requirements are met.

### Pre-Migration Requirements

| Requirement | Status | Verification | Blocker? |
|-------------|--------|--------------|----------|
| Redux API at 100% functionality | ❌ **BLOCKED** | BusinessLogic.dll must be resolved | **YES** |
| HTTPS configured on Original API | ⚠️ Optional | For baseline comparison testing | NO |
| SSL certificates obtained | 🔲 Pending | Required for HTTPS clients | **YES** |
| Development environment ready | 🔲 Pending | Redux API running on test server | **YES** |
| Comprehensive test suite | 🔲 Pending | All 17 endpoints covered | **YES** |
| Rollback plan documented | 🔲 Pending | See Rollback Strategy section | **YES** |
| Client stakeholders notified | 🔲 Pending | 2-week advance notice | NO |
| Database backup created | 🔲 Pending | Before any changes | **YES** |

### Current Blocker Status

**🔴 MIGRATION BLOCKED - BusinessLogic.dll Incompatibility**

| Component | Status | Impact |
|-----------|--------|--------|
| BusinessLogic.dll | ❌ Not Compatible | 6/17 endpoints unavailable |
| Redux API Functionality | 65% (11/17) | 35% functionality loss |
| Migration Readiness | **NOT READY** | Blocker must be resolved first |

**Resolution Required:** Recompile Business Logic.dll to .NET Standard 2.0 or .NET 9.0

**Estimated Time:** 16-24 hours (API team responsibility)

⚠️ **DO NOT PROCEED** until Redux API reaches 100% functionality (17/17 endpoints operational).

---

## Migration Timeline

### Recommended Phases

| Phase | Duration | Prerequisites | Deliverables |
|-------|----------|---------------|--------------|
| **Phase 0: Blocker Resolution** | 2-3 weeks | BusinessLogic.dll recompilation | Redux API 100% functional |
| **Phase 1: Preparation** | 1-2 weeks | Blocker resolved | Updated code, configuration |
| **Phase 2: Development Testing** | 1-2 weeks | Phase 1 complete | Dev environment validated |
| **Phase 3: Staging Testing** | 2-3 weeks | Phase 2 complete | Staging environment validated |
| **Phase 4: Production Rollout** | 2-4 weeks | Phase 3 complete | 100% traffic on Redux API |
| **Phase 5: Original API Deprecation** | 30-90 days | Phase 4 complete | Original API decommissioned |

**Total Timeline:** 10-16 weeks

---

## Step-by-Step Migration Instructions

### Step 1: Update Base URL Configuration

**Change:** Protocol (HTTP/HTTPS) and Port (51306 → 5001)

#### For .NET/C# Applications

**Before (Original API):**
```json
// appsettings.json
{
  "BargeOpsApi": {
    "BaseUrl": "http://localhost:51306",
    "ApiKey": "your-original-api-key"
  }
}
```

**After (Redux API):**
```json
// appsettings.json
{
  "BargeOpsApi": {
    "BaseUrl": "https://localhost:5001",
    "ApiKey": "your-redux-api-key"
  }
}
```

#### For JavaScript/Node.js Applications

**Before (Original API):**
```javascript
// config.js
module.exports = {
  apiBaseUrl: 'http://localhost:51306',
  apiKey: 'your-original-api-key'
};
```

**After (Redux API):**
```javascript
// config.js
module.exports = {
  apiBaseUrl: 'https://localhost:5001',
  apiKey: 'your-redux-api-key'
};
```

#### For Environment Variables

**Before:**
```bash
export API_BASE_URL="http://localhost:51306"
export API_KEY="your-original-api-key"
```

**After:**
```bash
export API_BASE_URL="https://localhost:5001"
export API_KEY="your-redux-api-key"
```

---

### Step 2: Update Endpoint URLs (Controller Names)

**Change:** PascalCase → lowercase controller names

#### Find and Replace Pattern

| Original URL Pattern | Redux URL Pattern |
|---------------------|-------------------|
| `/api/ComTrac/` | `/api/comtrac/` |
| `/api/HelmIntegration/` | `/api/helmintegration/` |
| `/api/MatchTracksIntegration/` | `/api/matchtracksintegration/` |
| `/api/HealthCheck/` | `/api/healthcheck/` |

#### Example Code Updates

**C# Example:**
```csharp
// BEFORE
private const string GetSchedulesEndpoint = "/api/ComTrac/GetNewComtracSchedules";

// AFTER
private const string GetSchedulesEndpoint = "/api/comtrac/GetNewComtracSchedules";
```

**JavaScript Example:**
```javascript
// BEFORE
const endpoint = '/api/ComTrac/GetNewComtracSchedules';

// AFTER
const endpoint = '/api/comtrac/GetNewComtracSchedules';
```

#### Automated Find/Replace

Use your IDE's find/replace feature:

1. Find: `/api/ComTrac/`
   Replace: `/api/comtrac/`

2. Find: `/api/HelmIntegration/`
   Replace: `/api/helmintegration/`

3. Find: `/api/MatchTracksIntegration/`
   Replace: `/api/matchtracksintegration/`

4. Find: `/api/HealthCheck/`
   Replace: `/api/healthcheck/`

---

### Step 3: Update JSON Deserialization Settings

**Change:** PascalCase → camelCase response fields

#### For C# Applications (.NET Framework/Core)

**Option 1: Global JsonSerializerSettings (Recommended)**

```csharp
// Startup.cs or Program.cs

// BEFORE (Original API - PascalCase)
services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Default settings (PascalCase)
    });

// AFTER (Redux API - camelCase)
services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    });
```

**Option 2: Per-Request Deserialization (Newtonsoft.Json)**

```csharp
// BEFORE (Original API)
var response = JsonConvert.DeserializeObject<ResponseModel>(jsonString);

// AFTER (Redux API)
var settings = new JsonSerializerSettings
{
    ContractResolver = new CamelCasePropertyNamesContractResolver()
};
var response = JsonConvert.DeserializeObject<ResponseModel>(jsonString, settings);
```

**Option 3: Per-Request Deserialization (System.Text.Json)**

```csharp
// BEFORE (Original API)
var response = JsonSerializer.Deserialize<ResponseModel>(jsonString);

// AFTER (Redux API)
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};
var response = JsonSerializer.Deserialize<ResponseModel>(jsonString, options);
```

#### For JavaScript Applications

**JavaScript naturally uses camelCase**, so NO CODE CHANGE needed for field access.

**However**, if you're using TypeScript interfaces matching Original API:

```typescript
// BEFORE (Original API - PascalCase interface)
interface ComtracResponse {
  Success: boolean;
  Message: string;
  BargeID: string;
  TransactionID: string;
}

// AFTER (Redux API - camelCase interface)
interface ComtracResponse {
  success: boolean;
  message: string;
  bargeId: string;
  transactionId: string;
}
```

---

### Step 4: Update Response Model Classes

**Change:** All property names from PascalCase → camelCase

#### C# Response Models

**Example 1: ComTrac Submit Response**

```csharp
// BEFORE (Original API)
public class ComtracSubmitResponse
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public string BargeID { get; set; }
    public string TransactionID { get; set; }
}

// AFTER (Redux API) - Property names match JSON response
public class ComtracSubmitResponse
{
    public bool Success { get; set; }  // C# property remains PascalCase
    public string Message { get; set; }
    public string BargeId { get; set; }
    public string TransactionId { get; set; }
}

// Then use camelCase deserializer settings (Step 3)
```

**Example 2: Health Check Response (Schema Changed)**

```csharp
// BEFORE (Original API)
public class HealthCheckResponse
{
    public string ApplicationStatus { get; set; }
    public string DatabaseConnectionStatus { get; set; }
}

// AFTER (Redux API) - Complete rewrite required
public class HealthCheckResponse
{
    public string Status { get; set; }
    public string TotalDuration { get; set; }
    public Dictionary<string, HealthCheckEntry> Entries { get; set; }
}

public class HealthCheckEntry
{
    public string Status { get; set; }
    public string Description { get; set; }
    public string Duration { get; set; }
}
```

#### TypeScript Response Models

```typescript
// BEFORE (Original API)
interface HealthCheckResponse {
  ApplicationStatus: string;
  DatabaseConnectionStatus: string;
}

// AFTER (Redux API)
interface HealthCheckResponse {
  status: string;
  totalDuration: string;
  entries: {
    [key: string]: {
      status: string;
      description: string;
      duration: string;
    };
  };
}
```

---

### Step 5: Update Health Check Integration

**Change:** Complete schema rewrite (Breaking Change #3)

#### Health Check Monitoring Dashboard

**Before (Original API):**
```csharp
public async Task<bool> IsApiHealthy()
{
    var response = await _httpClient.GetAsync("/api/HealthCheck/CheckHealth");
    var healthData = await response.Content.ReadAsAsync<HealthCheckResponse>();

    return healthData.ApplicationStatus == "Healthy" &&
           healthData.DatabaseConnectionStatus == "Connected";
}
```

**After (Redux API):**
```csharp
public async Task<bool> IsApiHealthy()
{
    var response = await _httpClient.GetAsync("/api/healthcheck/CheckHealth");
    var healthData = await response.Content.ReadAsAsync<HealthCheckResponse>();

    // Check overall status
    if (healthData.Status != "Healthy")
        return false;

    // Check individual component health
    foreach (var entry in healthData.Entries.Values)
    {
        if (entry.Status != "Healthy")
            return false;
    }

    return true;
}
```

#### Alerting Rules Update

**Before (Original API):**
```yaml
# Prometheus alert rule
- alert: ApiUnhealthy
  expr: bargeops_application_status != "Healthy"
  annotations:
    summary: "Application status is unhealthy"
```

**After (Redux API):**
```yaml
# Prometheus alert rule - updated for nested structure
- alert: ApiUnhealthy
  expr: bargeops_health_status != "Healthy"
  annotations:
    summary: "Overall health status is unhealthy"

- alert: DatabaseUnhealthy
  expr: bargeops_health_entries{component="database"} != "Healthy"
  annotations:
    summary: "Database health check failed"
```

---

### Step 6: Configure HTTPS Certificate Handling

**Change:** HTTP → HTTPS (if Original API requires HTTPS)

#### For .NET/C# Applications

**Option 1: Trust Development Certificates (Development Only)**

```bash
# Trust ASP.NET Core development certificate
dotnet dev-certs https --trust
```

**Option 2: Configure HttpClient to Accept Self-Signed Certificates (Development Only)**

```csharp
// DEVELOPMENT ONLY - DO NOT USE IN PRODUCTION
var handler = new HttpClientHandler
{
    ServerCertificateCustomValidationCallback =
        HttpClientHandler.DangerousAcceptAnyServerCertificateValidator
};
var httpClient = new HttpClient(handler);
```

**Option 3: Production Certificate Configuration**

```csharp
// Production - validate real certificates
var httpClient = new HttpClient();
// No custom validation - uses system certificate store
```

#### For Node.js Applications

**Development Only:**
```javascript
// DEVELOPMENT ONLY - DO NOT USE IN PRODUCTION
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

// Or with axios:
const axios = require('axios');
const https = require('https');

const httpsAgent = new https.Agent({
  rejectUnauthorized: false
});

axios.get('https://localhost:5001/api/healthcheck/ping', { httpsAgent });
```

**Production:**
```javascript
// Production - no special configuration needed
const axios = require('axios');
axios.get('https://api.production.com/api/healthcheck/ping');
```

---

### Step 7: Update API Key Authentication

**Change:** Regenerate API keys for Redux API

#### Obtain New API Key

Contact your API administrator to generate a new Redux API key.

**Original API Key Format:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9... (JWT)
```

**Redux API Key Format:**
```
bJ8yw22VTxRDq6WJKZTV6c (Simple key)
```

#### Update Authentication Headers

**C# Example:**
```csharp
// BEFORE (Original API)
_httpClient.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", originalApiKey);

// AFTER (Redux API) - Basic Authentication
var credentials = Convert.ToBase64String(
    Encoding.ASCII.GetBytes($"localdevonly:{reduxApiKey}")
);
_httpClient.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Basic", credentials);
```

**JavaScript/Axios Example:**
```javascript
// BEFORE (Original API)
axios.get('/api/ComTrac/GetNewComtracSchedules', {
  headers: {
    'Authorization': `Bearer ${originalApiKey}`
  }
});

// AFTER (Redux API) - Basic Authentication
const credentials = Buffer.from(`localdevonly:${reduxApiKey}`).toString('base64');
axios.get('/api/comtrac/GetNewComtracSchedules', {
  headers: {
    'Authorization': `Basic ${credentials}`
  }
});
```

---

## Configuration Changes

### Complete Configuration File Examples

#### .NET appsettings.json

**Before (Original API):**
```json
{
  "BargeOpsApi": {
    "BaseUrl": "http://localhost:51306",
    "ApiKey": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "Timeout": 30,
    "EnableHttps": false
  },
  "Endpoints": {
    "ComTrac": {
      "GetSchedules": "/api/ComTrac/GetNewComtracSchedules",
      "SubmitData": "/api/ComTrac/SubmitComtracBargeUnloadData"
    }
  }
}
```

**After (Redux API):**
```json
{
  "BargeOpsApi": {
    "BaseUrl": "https://localhost:5001",
    "ApiKey": "bJ8yw22VTxRDq6WJKZTV6c",
    "Timeout": 30,
    "EnableHttps": true,
    "ValidateCertificate": true
  },
  "Endpoints": {
    "ComTrac": {
      "GetSchedules": "/api/comtrac/GetNewComtracSchedules",
      "SubmitData": "/api/comtrac/SubmitComtracBargeUnloadData"
    }
  }
}
```

#### Node.js .env File

**Before (Original API):**
```bash
API_BASE_URL=http://localhost:51306
API_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
API_TIMEOUT=30000
ENABLE_HTTPS=false
```

**After (Redux API):**
```bash
API_BASE_URL=https://localhost:5001
API_KEY=bJ8yw22VTxRDq6WJKZTV6c
API_TIMEOUT=30000
ENABLE_HTTPS=true
VALIDATE_CERTIFICATE=true
```

---

## Testing Procedures

### Pre-Deployment Testing

#### 1. Unit Tests Update

**Update all unit tests to use Redux API response format:**

```csharp
// BEFORE (Original API test)
[Fact]
public void DeserializeComtracResponse_ShouldParsePascalCase()
{
    var json = @"{""Success"":true,""Message"":""OK"",""BargeID"":""12345""}";
    var response = JsonConvert.DeserializeObject<ComtracResponse>(json);
    Assert.True(response.Success);
    Assert.Equal("12345", response.BargeID);
}

// AFTER (Redux API test)
[Fact]
public void DeserializeComtracResponse_ShouldParseCamelCase()
{
    var json = @"{""success"":true,""message"":""OK"",""bargeId"":""12345""}";
    var settings = new JsonSerializerSettings
    {
        ContractResolver = new CamelCasePropertyNamesContractResolver()
    };
    var response = JsonConvert.DeserializeObject<ComtracResponse>(json, settings);
    Assert.True(response.Success);
    Assert.Equal("12345", response.BargeId);
}
```

#### 2. Integration Testing

**Create parallel test suite against both APIs:**

```csharp
[Theory]
[InlineData("http://localhost:51306", "original")]  // Original API
[InlineData("https://localhost:5001", "redux")]     // Redux API
public async Task GetComtracSchedules_ShouldReturnSchedules(string baseUrl, string apiType)
{
    // Arrange
    var client = CreateHttpClient(baseUrl, apiType);

    // Act
    var response = await client.GetAsync(GetSchedulesEndpoint(apiType));

    // Assert
    Assert.Equal(HttpStatusCode.OK, response.StatusCode);

    if (apiType == "original")
        await AssertOriginalApiResponse(response);
    else
        await AssertReduxApiResponse(response);
}
```

#### 3. Performance Testing

**Baseline performance comparison:**

| Test Case | Original API | Redux API | Acceptable? |
|-----------|-------------|-----------|-------------|
| GET /GetNewComtracSchedules | 15ms avg | ? | < 50ms |
| POST /SubmitComtracBargeUnloadData | 25ms avg | ? | < 100ms |
| GET /GetBargeTripList | 20ms avg | ? | < 50ms |
| POST /BargeDamageUpdate | 10ms avg | ? | < 50ms |

**Run load tests:**
```bash
# Use Apache Bench or similar tool
ab -n 1000 -c 10 https://localhost:5001/api/comtrac/GetNewComtracSchedules
```

---

### Deployment Testing Checklist

**Development Environment:**
- [ ] All endpoints return 200 OK
- [ ] Response deserialization works correctly
- [ ] Health check monitoring dashboard updated
- [ ] Alerting rules updated and tested
- [ ] Performance within acceptable range
- [ ] Error scenarios tested (400, 401, 404, 500)

**Staging Environment:**
- [ ] Full regression test suite passed (100%)
- [ ] Load testing completed (1000 req/sec sustained)
- [ ] SSL certificates validated
- [ ] Database connectivity verified
- [ ] All 17 endpoints operational
- [ ] No errors in logs (24-hour observation)

**Production Readiness:**
- [ ] Staging tests passed for 1 week
- [ ] Rollback plan tested
- [ ] Monitoring dashboards configured
- [ ] Alerting thresholds set
- [ ] On-call team briefed
- [ ] Client stakeholders notified

---

## Rollback Strategy

### Pre-Deployment Preparations

**1. Document Current State:**
```bash
# Capture current Original API metrics
curl http://localhost:51306/api/HealthCheck/CheckHealth > original-baseline.json

# Capture configuration
cp appsettings.json appsettings.original.json.backup
```

**2. Create Rollback Configuration:**
```json
// appsettings.rollback.json (Original API config)
{
  "BargeOpsApi": {
    "BaseUrl": "http://localhost:51306",
    "ApiKey": "original-key-here"
  }
}
```

**3. Test Rollback Procedure:**
```bash
# Practice rollback in staging
cp appsettings.rollback.json appsettings.json
dotnet run
# Verify API responses
```

### Rollback Triggers

**Initiate rollback immediately if:**
1. Error rate > 5% for 5 minutes
2. Response time > 2x baseline for 10 minutes
3. Any BLOCKER discovered (e.g., missing endpoint)
4. Client reports critical issues
5. Database connectivity lost
6. SSL certificate validation failures

### Rollback Procedure

**Step 1: Switch Configuration (< 2 minutes)**
```bash
# Stop Redux API client
systemctl stop client-app

# Restore Original API configuration
cp appsettings.original.json.backup appsettings.json

# Start client with Original API config
systemctl start client-app
```

**Step 2: Verify Rollback (< 3 minutes)**
```bash
# Test health check
curl http://localhost:51306/api/HealthCheck/CheckHealth

# Test critical endpoints
curl http://localhost:51306/api/ComTrac/GetNewComtracSchedules
```

**Step 3: Monitor (< 10 minutes)**
```bash
# Watch error logs
tail -f /var/log/client-app/error.log

# Check metrics dashboard
# Ensure error rate returns to normal
```

**Total Rollback Time: < 15 minutes**

---

## Common Issues & Solutions

### Issue 1: SSL Certificate Validation Failure

**Error:**
```
System.Net.Http.HttpRequestException: The SSL connection could not be established
Unable to get local issuer certificate
```

**Solution (Development):**
```bash
# Trust ASP.NET Core dev certificate
dotnet dev-certs https --trust
```

**Solution (Production):**
```csharp
// Ensure proper certificate chain installed
// Contact your infrastructure team
```

---

### Issue 2: JSON Deserialization Failure

**Error:**
```
Newtonsoft.Json.JsonSerializationException: Error converting value "OK" to type 'System.Boolean'
```

**Cause:** Response field name mismatch (PascalCase vs camelCase)

**Solution:**
```csharp
// Add camelCase deserializer settings
var settings = new JsonSerializerSettings
{
    ContractResolver = new CamelCasePropertyNamesContractResolver()
};
var response = JsonConvert.DeserializeObject<ResponseModel>(json, settings);
```

---

### Issue 3: 404 Not Found Errors

**Error:**
```
HTTP 404 Not Found
/api/ComTrac/GetNewComtracSchedules
```

**Cause:** Controller name still using PascalCase

**Solution:**
```csharp
// Change:
const string endpoint = "/api/ComTrac/GetNewComtracSchedules";
// To:
const string endpoint = "/api/comtrac/GetNewComtracSchedules";
```

---

### Issue 4: Health Check Parsing Fails

**Error:**
```
NullReferenceException: Object reference not set to an instance of an object
healthData.DatabaseConnectionStatus
```

**Cause:** Health check schema changed

**Solution:**
```csharp
// Rewrite health check parsing logic
// See Step 5: Update Health Check Integration
```

---

### Issue 5: Authentication Fails (HTTP 401)

**Error:**
```
HTTP 401 Unauthorized
HTTPS is required for authenticated requests
```

**Cause:** Original API requires HTTPS

**Solution:**
```json
// Update configuration to use HTTPS
{
  "BaseUrl": "https://localhost:51306"  // Changed from http
}
```

---

## Migration Support

### Getting Help

**Technical Support:**
- Email: api-support@bargeops.com
- Slack: #api-migration
- Phone: 1-800-BARGEOPS (emergency only)

**Documentation:**
- Breaking Changes Report: `BREAKING-CHANGES-REPORT.md`
- API Comparison Matrix: `API-COMPARISON-MATRIX.md`
- Project Plan: `PROJECT-PLAN.md`

**Office Hours:**
- Migration support calls: Tuesdays & Thursdays, 2-4 PM EST
- Zoom link: https://bargeops.zoom.us/api-migration

---

## Migration Success Metrics

### How to Measure Success

**Metrics to Monitor:**

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| API Availability | > 99.9% | TBD | 🔲 |
| Error Rate | < 0.1% | TBD | 🔲 |
| Response Time (p50) | < 50ms | TBD | 🔲 |
| Response Time (p99) | < 200ms | TBD | 🔲 |
| Client Compatibility | 100% | TBD | 🔲 |

**Success Criteria:**
- ✅ All 17 endpoints operational (100%)
- ✅ No increase in error rate
- ✅ Response times within baseline ±20%
- ✅ All clients migrated successfully
- ✅ Zero production incidents

---

## Conclusion

### Key Takeaways

1. **NOT A SIMPLE UPDATE:** This is a major migration requiring code changes
2. **BLOCKER EXISTS:** BusinessLogic.dll must be resolved first
3. **BREAKING CHANGES:** 5 critical changes impact all clients
4. **TESTING REQUIRED:** Comprehensive testing before production
5. **ROLLBACK READY:** Always have rollback plan tested

### Estimated Effort

**Per Client Application:**
- Code updates: 8-16 hours
- Configuration changes: 1-2 hours
- Testing: 8-16 hours
- Deployment: 2-4 hours
- **Total: 20-40 hours**

**Project-Wide:**
- API blocker resolution: 16-24 hours (API team)
- Multiple client applications: 20-40 hours each
- Integration testing: 40-80 hours
- Staged rollout: 2-4 weeks
- **Total: 10-16 weeks**

### Final Recommendations

1. **DO NOT RUSH:** Wait for blocker resolution
2. **TEST THOROUGHLY:** Use staging environment extensively
3. **MIGRATE GRADUALLY:** Phased rollout by client
4. **MONITOR CLOSELY:** Watch metrics during rollout
5. **COMMUNICATE CLEARLY:** Keep stakeholders informed

---

**Document Version:** 1.0
**Last Updated:** 2025-11-14
**Status:** Stage 6 Complete - Migration Guide Ready
**Next Steps:** Review API Comparison Matrix for endpoint-level details

