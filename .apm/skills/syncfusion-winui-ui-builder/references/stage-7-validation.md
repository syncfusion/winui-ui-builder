# Stage 7: Validation

**Purpose:** Validate generated code against desktop standards. Binary pass/fail result.

## Validation Checklist

1. **WCAG 2.1 AA Accessibility:**
   - Semantic XAML structure (StackPanel, Grid, Button)
   - AutomationProperties on form fields
   - Keyboard navigation (tab order, focus management)
   - Color contrast ≥ 4.5:1 for text
   - Focus indicators on interactive controls

2. **Security:**
   - No dynamic XAML parsing or injection vulnerabilities
   - No hardcoded secrets/API keys
   - Input sanitized where applicable
   - No unsafe reflection

3. **Performance:**
   - Virtualization for list controls
   - Optimized control refreshes
   - Lazy loading implemented
   - Code optimized

4. **Responsive Design:**
   - Desktop-first approach (1920px down)
   - DockPanel/Grid layouts
   - Adaptive breakpoints
   - Touch targets ≥ 44x44px

5. **XAML Property Validation (CRITICAL):**
   - No WPF-only properties (LabelFormat, DisplayFormat)
   - All properties valid for WinUI
   - Correct namespaces and versions
   - Templates used instead of format strings

6. **Theme Resource Initialization (CRITICAL):**
   - `SyncfusionLicenseProvider.RegisterLicense()` in App constructor
   - `RequestedTheme` set in App.xaml.cs (not in control XAML)
   - Application.Resources properly structured
   - `Syncfusion.UI.Xaml.Core` installed
   - All Syncfusion packages match version (e.g., 25.1.35)

7. **Resource Dictionary Validation (CRITICAL):**
   - All `.xaml` resource files merged into `Application.Resources` in App.xaml
   - Merge format: `<ResourceDictionary.MergedDictionaries>`
   - Namespace URIs correctly mapped (e.g., `using:Namespace`)
   - No circular resource dependencies
   - Resource keys unique and not overwritten
   - Syncfusion theme resources loaded before custom controls

8. **Window/Page Navigation Validation (CRITICAL):**
   - App.xaml.cs configures correct startup window (not default MainPage)
   - `Window.Activate()` called to display window
   - Content control explicitly set (not relying on default navigation)
   - **MANDATORY**: `RootFrame.Navigate(typeof(NewPage))` must be implemented in the startup logic (typically in `OnLaunched`) to route to the generated UI, ensuring it doesn't default to the boilerplate `MainPage`.
   - No blank windows (verify content renders in IDE preview)
   - Navigation logic routes to intended page, not default MainPage
   - App.xaml `StartupUri` matches intended window class

9. **Navigation Implementation Verification (CHECK):**
   - Verify `App.xaml.cs` has been updated to initialize the root frame
   - Verify `Frame.Navigate` is called with the correct `Type` of the generated section/page
   - Verify `Window.Content` is set to the Frame instance
   - Ensure the generated page has a parameterless constructor for the navigation system to use

10. **Runtime Error Validation (CRITICAL):**
   - No "Cannot find a Resource with the Name/Key [ResourceName]" errors
   - All value converters (StringToVisibilityConverter, etc.) defined in resource dictionary
   - Theme resources properly initialized before runtime
   - Missing converter registrations: check ResourceDictionary merged files
   - Missing theme resources: verify Syncfusion theme loaded in App.xaml
   - Test app launch to verify no runtime resource errors
   - Check Output window for "WinRT information" warnings

## MSBuild Compilation & Error Detection

**Why Use MSBuild Over dotnet build:**
- Direct XamlCompiler error reporting with file/line numbers
- Displays specific errors instead of generic `MSB3073` wrapper
- Superior for WinUI XAML validation
- Native Visual Studio integration

**MSB3073 Error:** When XamlCompiler detects XAML property errors, it exits with code 1, which gets wrapped as generic `MSB3073` error. MSBuild provides direct access to the actual error details.

### Identify MSBuild Path

Priority order: VS2026 (v18) → VS2022 → Search standard paths → Environment PATH → Recursive search

```powershell
$msbuild = $null

# Define search paths in priority order
$searchPaths = @(
  # VS2026 (.NET 10) - Primary
  "C:\Program Files\Microsoft Visual Studio\18\Professional\MSBuild\Current\Bin\MSBuild.exe",
  "C:\Program Files\Microsoft Visual Studio\18\Enterprise\MSBuild\Current\Bin\MSBuild.exe",
  "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe",
  
  # VS2022 (.NET 8/9) - Secondary
  "C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\Bin\MSBuild.exe",
  "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe",
  "C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe",
  
  # VS2019 (.NET Framework) - Tertiary fallback
  "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\MSBuild\Current\Bin\MSBuild.exe",
  "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\MSBuild\Current\Bin\MSBuild.exe",
  "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe"
)

# Search for MSBuild in standard paths
foreach ($path in $searchPaths) {
  if (Test-Path $path) {
    $msbuild = $path
    Write-Host "✓ Found MSBuild at: $msbuild" -ForegroundColor Green
    break
  }
}

# If not found in standard paths, search environment PATH
if (!$msbuild) {
  $msbuildFromPath = Get-Command MSBuild.exe -ErrorAction SilentlyContinue
  if ($msbuildFromPath) {
    $msbuild = $msbuildFromPath.Source
    Write-Host "✓ Found MSBuild in PATH: $msbuild" -ForegroundColor Green
  }
}

# If still not found, search Program Files recursively
if (!$msbuild) {
  Write-Host "⚠ MSBuild not found in standard locations. Searching Program Files..." -ForegroundColor Yellow
  $msbuild = Get-ChildItem -Path "C:\Program Files*" -Filter "MSBuild.exe" -Recurse -ErrorAction SilentlyContinue | Select-Object -First 1 -ExpandProperty FullName
  if ($msbuild) {
    Write-Host "✓ Found MSBuild via recursive search: $msbuild" -ForegroundColor Green
  }
}

# Final check: if still not found, error
if (!$msbuild) {
  Write-Host "❌ MSBuild not found. Please install Visual Studio 2022 or later." -ForegroundColor Red
  exit 1
}
```

### Compile with MSBuild

```bash
& $msbuild /t:Build /p:Configuration=Debug /p:Platform=x64 /v:detailed YourProject.csproj 2>&1 | Tee-Object build.log
```

**Key Parameters:**
- `/t:Build` - Build target (Build, Clean, Rebuild)
- `/p:Configuration` - Debug or Release
- `/p:Platform` - x64, x86, or ARM64
- `/v:detailed` - Verbosity (minimal, normal, detailed, diagnostic)

### Compilation Methods (Priority Order)

| Priority | Method | Details | Use When |
|----------|--------|---------|----------|
| 1 | **MSBuild Compiler** | Direct XAML error reporting with file/line; native XamlCompiler integration | Primary method for WinUI validation |
| 2 | **Visual Studio IDE** | Interactive debugging; Error List shows all XAML issues | Local development needed |
| 3 | **dotnet build** | CLI-based compilation; good for CI/CD pipelines | MSBuild not available or automation required |
| 4 | **Manual XAML inspection** | Spot-check for known WPF patterns | Quick local verification |

## XAML Error Resolution Workflow

**Step 1: Run MSBuild Compilation**
```bash
# First, discover MSBuild path (VS2026 > VS2022)
$msbuild = "C:\Program Files\Microsoft Visual Studio\18\Professional\MSBuild\Current\Bin\MSBuild.exe"
if (!(Test-Path $msbuild)) { $msbuild = "C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\Bin\MSBuild.exe" }
& $msbuild /t:Build /p:Configuration=Debug /p:Platform=x64 /v:detailed YourProject.csproj 2>&1 | Tee-Object build.log
```

**Step 2: Parse Errors from Log**
```powershell
Select-String -Path "build.log" -Pattern "error|Error" | ForEach-Object { $_.Line }
```
Look for: `Unknown member 'PropertyName' on 'ControlName'` (File.xaml, Line XX)

**Step 3: Cross-Reference Against Skill Definitions**
- Open: `.apm/skills/syncfusion-winui-ui-builder/references/syncfusion-{controlname}.md`
- Verify supported properties and versions
- Check namespace/assembly mappings in `.csproj`

**Step 4: Apply Corrections**
- Replace format string properties with DataTemplate approach
- Update deprecated properties to WinUI equivalents
- Ensure all Syncfusion packages match version

**Step 5: Rebuild & Verify**
```bash
& $msbuild /t:Rebuild /p:Configuration=Debug /p:Platform=x64 YourProject.csproj
```
Confirm: exit code 0

## dotnet build Fallback Method

Use this when MSBuild is unavailable or in automated CI/CD pipelines where only .NET CLI is available.

**Compile with dotnet build:**
```bash
dotnet build YourProject.csproj --configuration Debug --verbosity diagnostic 2>&1 | Tee-Object build.log
```

**Extract XAML Errors from Log:**
```powershell
Select-String -Path "build.log" -Pattern "Unknown member|Type not found|Resource not found" | ForEach-Object { Write-Host $_.Line -ForegroundColor Red }
```

**Advantages of dotnet build:**
- No MSBuild/Visual Studio installation required
- Works on any .NET SDK-equipped system
- Good for CI/CD pipelines (GitHub Actions, Azure Pipelines)
- Cross-platform support

**Limitations:**
- Generic `MSB3073` wrapper errors may hide actual XAML issues
- Errors less detailed than MSBuild direct output
- Diagnostic output verbose and harder to parse

**Verbosity Levels for dotnet build:**
```bash
# Minimal output
dotnet build YourProject.csproj --verbosity minimal

# Normal output (default)
dotnet build YourProject.csproj --verbosity normal

# Detailed output (shows more compilation info)
dotnet build YourProject.csproj --verbosity detailed

# Diagnostic output (maximum detail, suitable for troubleshooting)
dotnet build YourProject.csproj --verbosity diagnostic 2>&1 | Tee-Object build-diagnostic.log
```

**CI/CD Pipeline Example (dotnet build):**
```powershell
# GitHub Actions / Azure Pipelines compatible
$projectFile = "YourProject.csproj"
$logFile = "build-diagnostic.log"

# Run compilation with diagnostic verbosity
dotnet build $projectFile --configuration Debug --verbosity diagnostic > $logFile 2>&1

# Parse for XAML errors
$xamlErrors = Select-String -Path $logFile -Pattern "Unknown member|Type not found|Resource not found"

if ($xamlErrors.Count -gt 0) {
    Write-Host "XAML Compilation Errors Found:" -ForegroundColor Red
    $xamlErrors | ForEach-Object { Write-Host "  $($_.Line)" -ForegroundColor Red }
    exit 1
}

if ($LASTEXITCODE -eq 0) {
    Write-Host "✓ Build succeeded - No XAML errors" -ForegroundColor Green
    exit 0
}

Write-Host "✗ Build failed - Review log file: $logFile" -ForegroundColor Red
exit 1
```

**Comparison: MSBuild vs dotnet build**

| Aspect | MSBuild | dotnet build |
|--------|---------|--------------|
| Error Detail | Direct, specific XAML errors | Generic wrapper (MSB3073) with buried details |
| Performance | Faster native compilation | Slightly slower (CLI overhead) |
| Availability | Requires Visual Studio | Requires .NET SDK only |
| CI/CD Friendly | Good with path discovery | Better for automated systems |
| Local Development | Recommended | Alternative |
| File/Line Info | Precise line numbers | May be vague |
| Format Strings | Full error context | May require log parsing |

**When to Use Fallback (dotnet build):**
- Visual Studio/MSBuild not installed on CI/CD agent
- Cross-platform builds (Linux, macOS)
- Lightweight container environments
- Automated scripts without Visual Studio dependency
- Development on systems with .NET SDK only

## Validation Result

**Binary Result: PASS ✓ or FAIL ✗**

**If PASS:**
- All standards met (accessibility, security, performance, responsive design, XAML properties, theme resources)
- Proceed to next stage
- "✓ Validation Passed: API signatures verified against skill definitions"

**If FAIL:**
- Identify specific errors with file/line references
- Apply corrections from property mapping table
- Rebuild and verify with MSBuild
- Iterate until all errors resolved

**User Decision:** Confirm validation passes before proceeding to dependencies stage.