# Dependency Analysis - BusinessLogic .NET 9.0 Migration

**Date:** 2025-11-21
**Project:** BargeOps.Onshore.BusinessLogic Migration
**Target:** .NET Framework 4.8 → .NET 9.0

---

## Executive Summary

**Csg.* Source Code:** ❌ Not found locally
**Strategy:** Use .NET Framework 4.8 DLLs via compatibility layer
**.NET SDK:** ✅ 9.0.304 installed
**Dependencies:** 5 Csg.* namespaces actually used (out of 7 referenced)

---

## Csg.* Dependencies Analysis

### Actually Used in Code

Based on `Imports` analysis of 1,129 .vb files:

1. **Csg.AppFramework** ✅ CRITICAL
   - Most heavily used
   - Core business logic framework
   - Data access layer

2. **Csg.AppFramework.Data** ✅ CRITICAL
   - Used in: Business Objects
   - Database operations

3. **Csg.Diagnostics.ExceptionManagement** ✅ IMPORTANT
   - Exception handling
   - Error logging

4. **Csg.Security.Cryptography** ✅ IMPORTANT
   - Security operations
   - Encryption/decryption

5. **Csg.Utilities** ✅ MODERATE
   - Helper functions
   - Property utilities

6. **Csg.Windows.Forms.Security** ⚠️ QUESTIONABLE
   - Windows Forms related
   - May not be needed server-side
   - Used in: Specific security classes

### Referenced but Usage Unknown

7. **Csg.AppFramework.Core.BindableBase** - Need to verify usage
8. **Csg.AppFramework.Server.DataPortal** - Server-side operations
9. **Csg.AppFramework.Resources** - Resources/localization

### Csg.* Source Code Search Results

**K Drive:** ❌ Not found in K:/CsgCommon/Development/
**Network Share:** ❌ Not found in //csgsolutions.com/share/Public/
**.NET Standard Versions:** ❌ No alternative framework versions found

**Conclusion:** Must use .NET Framework 4.8 DLLs and rely on .NET compatibility layer

---

## System.* Dependencies Analysis

### .NET 9.0 Compatible (No Issues Expected)

- ✅ System.Collections.Generic
- ✅ System.IO
- ✅ System.Linq
- ✅ System.Text
- ✅ System.Text.RegularExpressions
- ✅ System.Threading
- ✅ System.Threading.Tasks

### Requires NuGet Packages

1. **System.Data.SqlClient** → Package: `System.Data.SqlClient` v4.8.6
2. **System.Drawing** → Package: `System.Drawing.Common` v9.0.0
3. **System.Net.Mail** → Package: `System.Net.Mail` v4.3.0 (if needed)

### Potentially Problematic (.NET Framework → .NET 9.0)

1. **System.Runtime.Remoting.Messaging** ⚠️ DEPRECATED
   - Not supported in .NET 9.0
   - Need to identify usage and create workaround
   - Alternative: Use CallContext or AsyncLocal<T>

2. **System.Runtime.Serialization.Formatters.Binary** ⚠️ OBSOLETE
   - Binary serialization deprecated in .NET 5+
   - Security concerns
   - Alternative: JSON serialization or DataContract

3. **System.Runtime.InteropServices** ⚠️ CHECK USAGE
   - P/Invoke calls may need adjustment
   - Platform-specific code

---

## Windows Forms Dependencies

**Issue:** Project references `System.Windows.Forms`

**Analysis:**
- BusinessLogic is a server-side DLL
- Windows Forms likely used for UI dialogs in client apps
- May not be needed in web API context

**Options:**
1. Enable Windows Forms support: `<UseWindowsForms>true</UseWindowsForms>`
2. Conditional compilation to exclude Forms code
3. Remove if not actually used in API context

---

## Migration Strategy

### Phase 1: Use .NET Framework 4.8 DLLs As-Is

**Approach:**
- Reference existing Csg.* DLLs
- Use .NET compatibility features
- Suppress framework mismatch warnings

**Risks:**
- Runtime compatibility issues
- Performance implications
- Potential type load exceptions

**Mitigation:**
- Extensive testing
- Fallback to multi-targeting if needed

### Phase 2: Address System.* Breaking Changes

**Must Fix:**
1. Replace System.Runtime.Remoting.Messaging usage
2. Replace BinaryFormatter with secure serialization
3. Add required NuGet packages

**Should Fix:**
- Review P/Invoke usage
- Test Windows Forms dependencies

### Phase 3: Testing & Validation

- Unit test BusinessLogic classes
- Integration test with Redux API
- Performance baseline comparison
- Memory leak detection

---

## Required NuGet Packages

```xml
<ItemGroup>
  <!-- Database -->
  <PackageReference Include="System.Data.SqlClient" Version="4.8.6" />

  <!-- Drawing (if needed) -->
  <PackageReference Include="System.Drawing.Common" Version="9.0.0" />

  <!-- Windows Compatibility Pack (includes many .NET Framework APIs) -->
  <PackageReference Include="Microsoft.Windows.Compatibility" Version="9.0.0" />

  <!-- Configuration (if needed) -->
  <PackageReference Include="System.Configuration.ConfigurationManager" Version="9.0.0" />
</ItemGroup>
```

---

## Known Breaking Changes to Address

### 1. System.Runtime.Remoting (CRITICAL)

**Files Affected:** [To be determined during build]

**Fix:**
```vb
' Old (.NET Framework):
Imports System.Runtime.Remoting.Messaging
Dim context As CallContext = CallContext.GetData("key")

' New (.NET 9.0):
Imports System.Threading
Private Shared contextData As AsyncLocal(Of Object) = New AsyncLocal(Of Object)()
Dim context As Object = contextData.Value
```

### 2. BinaryFormatter (SECURITY ISSUE)

**Files Affected:** [To be determined during build]

**Fix:**
```vb
' Old (UNSAFE):
Dim formatter As New BinaryFormatter()
formatter.Serialize(stream, obj)

' New (SAFE):
Imports System.Text.Json
Dim json As String = JsonSerializer.Serialize(obj)
```

---

## Platform Considerations

**Original:** x86 (32-bit)
**Target:** AnyCPU (64-bit capable)

**Implications:**
- May improve performance on 64-bit systems
- Ensure no 32-bit specific code
- Test pointer arithmetic if any

---

## Estimated Complexity

| Category | Complexity | Time Estimate |
|----------|-----------|---------------|
| **Project File Conversion** | LOW | 1-2 hours |
| **NuGet Package Addition** | LOW | 30 min |
| **Csg.* Compatibility** | MEDIUM | 2-4 hours testing |
| **System.Remoting Replacement** | MEDIUM-HIGH | 2-6 hours |
| **BinaryFormatter Replacement** | MEDIUM | 2-4 hours |
| **Build Error Fixes** | MEDIUM | 3-8 hours |
| **Testing & Validation** | MEDIUM | 4-6 hours |
| **Total** | MEDIUM | **15-30 hours** |

---

## Success Criteria

### Build Phase
- ✅ Zero build errors
- ✅ Warnings < 10 (and all justified)
- ✅ DLL produced successfully
- ✅ All Csg.* types resolved

### Runtime Phase
- ✅ Redux API starts without DLL load exceptions
- ✅ BusinessLogic classes instantiate successfully
- ✅ 6 blocked endpoints return 200 OK
- ✅ No regressions in 11 working endpoints

### Testing Phase
- ✅ 17/17 endpoints operational (100% coverage)
- ✅ Performance within acceptable range
- ✅ No memory leaks detected
- ✅ Stable under load

---

## Next Steps

1. ✅ **Complete:** Dependency analysis
2. ⏭️ **Next:** Create working copy and backup
3. ⏭️ **Then:** Modernize project file
4. ⏭️ **Then:** Initial build attempt

---

**Document Status:** Complete
**Ready for Phase 2:** ✅ Yes
