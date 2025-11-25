# Build Errors Analysis - First Build Attempt

**Date:** 2025-11-21
**Build:** Initial .NET 9.0 build
**Result:** FAILED
**Total Error Occurrences:** 19,797

---

## Error Categories

### Category 1: System.Data Types Not Found (CRITICAL)

**Error Pattern:** `Type 'X' is not defined`

**Affected Types:**
- `SqlCommand` - Database command execution
- `DataTable` - Tabular data
- `DataRow` - Table row
- `DataView` - Filtered view of table
- `SqlDataReader` - Forward-only data reader

**Root Cause:**
- `System.Data` namespace not properly imported/available
- Missing explicit imports in files

**Fix Strategy:**
1. Add `Imports System.Data` to files that need it
2. Add `Imports System.Data.SqlClient` for SQL types
3. These should be available via `System.Data.SqlClient` NuGet package (already added)

**Estimate:** This affects hundreds of files but fix is straightforward

---

### Category 2: Csg.AppFramework Types Not Found (CRITICAL)

**Error Pattern:** `Type 'Csg.AppFramework.X' is not defined`

**Affected Types:**
- `Csg.AppFramework.CriteriaBase` - Base criteria class
- `Csg.AppFramework.SafeDataReader` - Data reader wrapper
- `Csg.AppFramework.BusinessBase` - Business object base
- `Csg.AppFramework.BusinessCollectionBase` - Collection base

**Root Cause:**
- Csg.AppFramework.dll (.NET Framework 4.8) types not loading in .NET 9.0
- Framework mismatch causing type resolution issues
- These are core framework types that BusinessLogic heavily depends on

**Fix Strategy:**
**This is the CRITICAL blocker!**

**Option A: Csg.* DLLs incompatible**
- .NET Framework 4.8 DLLs cannot be used directly
- Need Csg.* source code to recompile
- OR need .NET Standard versions

**Option B: Missing imports**
- Try adding explicit `Imports Csg.AppFramework` statements
- May help compiler find types

**Current Status:** Need to investigate if this is import issue or fundamental incompatibility

---

### Category 3: Override Signature Mismatches (MODERATE)

**Error Pattern:** `sub 'X' cannot be declared 'Overrides' because it does not override a sub in a base class`

**Examples:**
- `AddFetchParameters` - Virtual methods from base classes

**Root Cause:**
- Base class signatures changed between framework versions
- Csg.AppFramework base classes may have different signatures

**Fix Strategy:**
- IF Csg.* types load: Adjust override signatures
- IF Csg.* types don't load: This error is secondary to Category 2

---

## Error Statistics

| Category | Unique Errors | Est. Occurrences | Severity |
|----------|--------------|------------------|----------|
| System.Data types | ~10 types | ~10,000 | HIGH |
| Csg.AppFramework types | ~20 types | ~8,000 | CRITICAL |
| Override mismatches | ~50 methods | ~1,000 | MEDIUM |
| Other | Various | ~797 | LOW |

---

## Root Cause Analysis

### Primary Issue: Csg.AppFramework Incompatibility

The .NET Framework 4.8 Csg.AppFramework.dll is NOT loading properly in .NET 9.0.

**Evidence:**
```
error BC30002: Type 'Csg.AppFramework.CriteriaBase' is not defined
error BC30002: Type 'Csg.AppFramework.SafeDataReader' is not defined
error BC30002: Type 'Csg.AppFramework.BusinessBase' is not defined
```

These are core types that should be available if the DLL reference was working.

### Secondary Issue: Missing Imports

Many files don't have explicit `Imports System.Data` statements, relying on project-level imports.

---

## Investigation Needed

### Step 1: Verify Csg.* DLL References

Check if .NET 9.0 can actually load .NET Framework 4.8 DLLs:

```bash
# Test if types are visible
dotnet list package --include-transitive
```

### Step 2: Try Adding Global Imports

Add to project file:
```xml
<ItemGroup>
  <Import Include="System.Data" />
  <Import Include="System.Data.SqlClient" />
  <Import Include="Csg.AppFramework" />
  <Import Include="Csg.AppFramework.Data" />
</ItemGroup>
```

### Step 3: Check Assembly Loading

The .NET Framework compatibility layer may not be working as expected.

---

## Next Steps

### Immediate (Phase 5)

1. **Test DLL Loading:**
   - Create simple test project
   - Try to use Csg.AppFramework types
   - See if DLL loads at all

2. **Add Global Imports:**
   - Add `<Import>` statements to project file
   - This may resolve "not defined" errors

3. **Check for .NET Standard Csg Versions:**
   - One more thorough search
   - Contact dev team

### If Csg.* DLLs Don't Load

**Option A: Find Source Code**
- Search internal repositories
- Contact original developers
- Check backup archives

**Option B: Multi-Target Approach**
```xml
<TargetFrameworks>net9.0-windows;net48</TargetFrameworks>
```
- Build for both frameworks
- Use .NET 4.8 build temporarily

**Option C: Conditional Compilation**
- Isolate Csg-dependent code
- Provide .NET 9.0 alternatives

**Option D: Acceptance**
- 65% test coverage may be sufficient
- Document blocker for production migration
- Proceed with Option A (accept limitations)

---

## Estimated Fix Complexity

| Scenario | Complexity | Time Estimate |
|----------|-----------|---------------|
| **Csg.* DLLs load with imports fix** | LOW | 2-4 hours |
| **Need Csg.* source code to recompile** | HIGH | 40-80 hours |
| **Multi-target workaround** | MEDIUM | 8-16 hours |
| **Cannot resolve** | N/A | Fallback to Option A |

---

## Decision Point

**CRITICAL QUESTION:** Can .NET 9.0 use .NET Framework 4.8 Csg.AppFramework.dll?

**If YES:** Continue with import fixes (Phase 5)
**If NO:** Need alternative strategy

**Next Action:** Phase 5 - Add global imports and test if Csg types become available

---

**Status:** Phase 4 Complete - Error Analysis Done
**Next:** Phase 5 - Attempt fixes
