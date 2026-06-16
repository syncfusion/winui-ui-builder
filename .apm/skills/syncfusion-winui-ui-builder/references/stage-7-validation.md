# Stage 7: Final Validation

**Purpose:** Two-phase gate before Stage 8 (Code Insertion):
1. **Pre-Compilation Validation** — validates generated code against skill files, WinUI SDK, and project structure *before* MSBuild
2. **Post-Build Quality Validation** — validates accessibility, security, performance, and Syncfusion integration *after* a clean build

**Input:** All files from Stage 5 + `skill-extraction.json`
**Output:** PASS ✓ / FAIL ✗ per check + overall result

⛔ **Phase 1 must pass before MSBuild. Phase 2 runs only after build succeeds.**

---

## MANDATORY EXECUTION ORDER

```
┌─────────────────────────────────────────────────────────────┐
│ GATE 1: Pre-Check 0 — Startup View Config                   │
│    → FAIL: HALT — no other checks proceed                   │
├─────────────────────────────────────────────────────────────┤
│ GATE 2: Pre-Checks 1–6 — Code Validation                    │
│    → Pre-Check 1: skill-extraction.json PASS + re-read      │
│      skill files → validate controls/properties/events      │
│      FAIL: "Invalid API usage" → HALT, MSBuild NOT invoked  │
│    → Pre-Checks 2–6: WinUI SDK, XAML, Runtime, NuGet, MVVM │
│      ANY FAIL → HALT                                        │
├─────────────────────────────────────────────────────────────┤
│ GATE 3: MSBuild Compilation (VS2026/VS2022 ONLY)            │
│    → ❌ NEVER: dotnet build                                  │
│    → ✅ ALWAYS: MSBuild <solution>.sln                       │
│    → BUILD FAILS → re-read skill file → fix → retry        │
│    → BUILD PASS → Phase 2                                   │
├─────────────────────────────────────────────────────────────┤
│ GATE 4: Post-Build Quality Validation                       │
│    → WCAG, Security, Performance, Resources, Runtime Safety │
│    → FAIL: block Stage 8 · PASS: proceed to Stage 8        │
└─────────────────────────────────────────────────────────────┘
• Do NOT skip any gate · Do NOT use dotnet build — MSBuild ONLY
```

---

## Phase 1: Pre-Compilation Validation

> Refer to `Build.md` for pseudocode, property tables, and auto-fix details.

---

### Pre-Check 0: Startup View Configuration ⛔ GATES ALL CHECKS

```
1. App.xaml.cs exists
   ❌ → HALT: "App.xaml.cs not found"
2. OnLaunched() present and implemented
   ❌ → HALT: "OnLaunched() missing in App.xaml.cs"
3. Startup window defined: new MainWindow() / new StartupWindow() / new <Name>()
   ❌ → HALT: "No startup window in OnLaunched()"
4. window.Activate() called
   ❌ → HALT: "window.Activate() missing"
5. Window class inherits Microsoft.UI.Xaml.Window
   ❌ → HALT: "Startup window must inherit from Microsoft.UI.Xaml.Window"
6. IF Syncfusion controls used: SyncfusionLicenseProvider.RegisterLicense() BEFORE window init
   ❌ → HALT: "License not registered before window initialization"
7. Startup window .xaml exists and is valid
   ❌ → HALT: "<WindowName>.xaml not found or invalid"
8. IF MVVM: DataContext set in code-behind
   ❌ → HALT: "DataContext not set — bindings will fail"
✅ PASS → proceed · ❌ FAIL → HALT, fix before any other check
```

---

### Pre-Check 1: Skill-Based API Validation ⛔ CRITICAL

#### 1A — Validate `skill-extraction.json`
```
CONFIRM file exists at <project-root>/skill-extraction.json
  ❌ → HALT: "skill-extraction.json not found — re-run Stage 5B"
CONFIRM validation_status == "PASS"
  ❌ → HALT: "skill-extraction.json not validated — re-run Stage 5B"
```

#### 1B — Re-read each skill file
```
FOR EACH control in skill-extraction.json → controls[]:
  READ sources_read[0] (getting-started.md) + all advanced_features_read[] files
  ❌ Unreadable → HALT: "Skill file unreadable for <ControlName> — re-run Stage 5B"
```

#### 1C — Validate generated files against skill data
```
FOR EACH Syncfusion control in .xaml / .cs files:
  1. FIND in skill-extraction.json → controls[]
     ❌ → FAIL: "Invalid API usage — <ControlName> not in skill-extraction.json"
  2. Namespace matches controls[].namespace exactly
     ❌ → FAIL: "Namespace mismatch — Used: <x> | Expected: <y>"
  3. Every property in controls[].valid_properties[].name
     ❌ → FAIL: "Invalid API usage — '<propName>' not in skill docs for <ControlName>"
  4. Every event in controls[].valid_events[].name
     ❌ → FAIL: "Invalid API usage — '<eventName>' not in skill docs"
  5. Every method in controls[].valid_methods[].name
     ❌ → FAIL: "Invalid API usage — '<methodName>' not verified in skill file"
ANY failure → HALT: "Validation incomplete — cannot compile. Fix all skill errors first."
ALL pass → ✅ Skill validation complete — pre-build gate cleared
```

---

### Pre-Check 2: WinUI SDK Compliance

**Blocked WPF properties (HALT if found):**

| Property / API | WinUI Alternative |
|---|---|
| `{Binding}` classic | `{x:Bind}` |
| `DockPanel` | `Grid` or `RelativePanel` |
| `WrapPanel` | `ItemsWrapGrid` |
| `ContextMenu` | `MenuFlyout` |
| `System.Windows.*` | `Microsoft.UI.Xaml.*` |
| `Triggers` / `DataTrigger` | `VisualStateManager` |

**Element-level violations (HALT if found):**

| Property | ✅ Supported On | ❌ Not Supported On |
|---|---|---|
| `Padding` | `Control`, `Border` | `Grid`, `StackPanel`, `Canvas` |
| `CornerRadius` | `Border`, `Control` | `Grid`, `StackPanel` |
| `Background` | `Panel`, `Control`, `Border` | Plain `UIElement`/`FrameworkElement` |
| `Content` | `ContentControl` subclasses | `Panel`, `TextBlock`, `Image` |
| `Text` | `TextBox`, `TextBlock` | `Button`, `Border`, `Grid` |
| `IsChecked` | `ToggleButton` subclasses | `Button`, non-ToggleButton |
| `Orientation` | `StackPanel` | `Grid`, `Canvas` |

```
IF blocked property found → HALT: "Property '<n>' not valid in WinUI SDK — File: <f>, Line: <l>"
IF property not in element's class hierarchy → HALT: "Property '<n>' does not exist on '<ElementType>'"
```

---

### Pre-Check 3: XAML Structural Validation

```
FOR EACH .xaml file — simulate XamlReader.Load() (dry-run):
  · Tag mismatch       → HALT: "XAML tag mismatch in <file> at line <n>"
  · Invalid XML        → HALT: "Invalid XAML in <file>: <message>"
  · Wrong base xmlns   → HALT: "Missing xmlns='http://schemas.microsoft.com/winfx/2006/xaml/presentation'"
  · WPF/UWP namespace  → HALT: "Non-WinUI namespace '<ns>' — mixed framework not allowed"

Auto-fix loop (max 3 attempts):
  Fixable error → apply fix → retry
  Cascading errors → HALT: "Cascading XAML errors — manual review required"
  3 attempts exhausted → HALT: "XAML parse failed — return to Stage 5"
```

---

### Pre-Check 4: Runtime Error Prevention ⛔ NEW

Scan all generated code for patterns that compile but fail at runtime:

```
XAML Runtime Checks:
  · {StaticResource X} key not in any merged ResourceDictionary
    → HALT: "StaticResource '<X>' undefined — XamlParseException at runtime"
  · {ThemeResource X} not resolvable by WinUI theme system
    → HALT: "ThemeResource '<X>' undefined — will fail at runtime"
  · Converter referenced in XAML but not declared in ResourceDictionary
    → HALT: "Converter '<Name>' not in ResourceDictionary — XamlParseException at runtime"
  · xmlns:local or converter namespace does not map to correct assembly
    → HALT: "Converter namespace '<ns>' cannot be resolved"
  · Effect.Color not a valid Brush/ARGB string
    → HALT: "Invalid Effect property — XamlParseException at runtime"

C# Runtime Checks:
  · DataContext not assigned in Window/UserControl constructor
    → HALT: "DataContext not set in '<Window>' — bindings return null at runtime"
  · Event handler declared in XAML but method missing in code-behind
    → HALT: "Handler '<name>' missing in code-behind — NullReferenceException at runtime"
  · Async void event handler with unhandled exception path
    → HALT: "Async handler '<name>' has no exception handling — silent crash risk"
  · Task.Wait() or .Result on UI thread
    → HALT: "Blocking call on UI thread — deadlock at runtime"
  · ICommand.CanExecute never returns true for required commands
    → HALT: "Command '<name>' CanExecute always false — button permanently disabled"
  · ObservableCollection modified from non-UI thread without Dispatcher
    → HALT: "Collection cross-thread modification — InvalidOperationException at runtime"
  · Navigation target Window/Page class does not exist
    → HALT: "Navigation target '<ClassName>' not found — runtime crash on navigate"
```

---

### Pre-Check 5: NuGet Dependency Validation

```
FOR EACH nuget_package in skill-extraction.json → controls[].nuget_package:
  IF missing from .csproj:
    → dotnet add package <nuget_package>; dotnet restore
    → FAIL if restore fails: "Package '<name>' could not be installed"

Core packages (always required):
  ✅ Syncfusion.Core.WinUI  ✅ Syncfusion.Licensing

IF any Syncfusion packages use different versions:
  → HALT: "Version mismatch — all Syncfusion packages must use identical versions"
```

```bash
# Namespace error resolution
dotnet list package | grep Syncfusion   # compare with skill-extraction.json
dotnet add package <name> --version <ver>
```

---

### Pre-Check 6: MVVM & Binding Integrity

```
FOR EACH .xaml file:
  · {x:Bind X} → X exists in ViewModel or code-behind
    → HALT: "Binding '<X>' not found — File: <file>, Line: <line>"
  · {x:Bind XCommand} → ICommand XCommand in ViewModel
    → HALT: "Command '<XCommand>' not found"
  · Event handler X in XAML → method X in code-behind
    → HALT: "Handler '<X>' missing in <file>.xaml.cs"

FOR EACH ViewModel:
  · Implements INotifyPropertyChanged
    → HALT: "<ViewModel> missing INotifyPropertyChanged"
  · No business logic in code-behind
    → HALT: "Business logic in code-behind — move to ViewModel/Service"
```

---

### Build Execution

#### Pre-Build Gate (MANDATORY before any build command)
```
CONFIRM Pre-Check 1 status == COMPLETE AND PASS
  ❌ → HALT: "Validation incomplete — cannot compile. Complete skill validation first."
IF dotnet build invoked at any point:
  → FAIL: "Invalid build tool — use MSBuild from Visual Studio toolchain"
✅ Gate cleared → proceed to Step 1
```

#### Step 1: Resolve MSBuild Path
```
# Priority 1 — VS2026 (Primary)
"C:\Program Files\Microsoft Visual Studio\18\Professional\MSBuild\Current\Bin\MSBuild.exe"
"C:\Program Files\Microsoft Visual Studio\18\Enterprise\MSBuild\Current\Bin\MSBuild.exe"
"C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe"

# Priority 2 — VS2022 (Secondary — only if VS2026 not found)
"C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\Bin\MSBuild.exe"
"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe"
"C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe"

IF no path resolves → HALT: "VS2026/VS2022 MSBuild not found — install Visual Studio"
SET msbuild_exe = first resolved path
```
❌ Do NOT use legacy MSBuild · ❌ Do NOT use `dotnet build`

#### Step 2: Execute Build
```bash
"<msbuild_exe>" <ProjectName>.sln /t:Build /p:Configuration=Debug /p:Platform="x64" /v:detailed /flp:logfile=build.log;verbosity=detailed
```

| Flag | Purpose |
|---|---|
| `/t:Build` | Runs Build target |
| `/p:Configuration=Debug` | Debug configuration |
| `/p:Platform="x64"` | Required for WinUI / Windows App SDK |
| `/v:detailed` | Full task/target trace |
| `/flp:logfile=build.log;verbosity=detailed` | Persists log for post-failure inspection |

**Error codes:** XAML → `XC` (e.g., `XC3001`) · C# → `CS` (e.g., `CS0246`)

#### Step 3: Build Failure Recovery (MANDATORY if build fails)
```
IF MSBuild fails:
  1. READ build.log → extract all error codes and messages
  2. FOR EACH error:
       a. Identify the Syncfusion control or API involved
       b. RE-READ the skill file: skill-extraction.json → controls[].sources_read[0]
          + all advanced_features_read[] files for that control
       c. Compare failing property/event/method against valid_properties[]/valid_events[]/valid_methods[]
       d. IF mismatch found → correct in generated file using skill-file-verified value
       e. IF error is WinUI SDK violation → apply Pre-Check 2 fix table
       f. IF package missing → install via Pre-Check 5 resolution
  3. Re-run Pre-Check 1C (skill validation) on corrected files
  4. Re-run MSBuild
  5. IF still failing after 3 cycles → HALT: "Build not resolved after 3 attempts
                                               Manual review of build.log required"

❌ Do NOT fix build errors by guessing — skill file is the only source of truth
❌ Do NOT fallback to native WinUI/MS controls without confirming no Syncfusion equivalent exists
```

| Build Result | Action |
|---|---|
| **0 errors** | ✅ Proceed to Phase 2 |
| **Errors** | ⛔ Run Step 3 recovery · re-read skill file · retry |
| **Warnings only** | ✅ Proceed; log for review |

---

## Phase 2: Post-Build Quality Validation

Run only after MSBuild 0 errors.

### Check 1: WinUI & MVVM Standards

| Rule | Fail Condition |
|---|---|
| MVVM separation | Business logic in code-behind |
| ViewModel binding | `{x:Bind X}` — `X` not in ViewModel/code-behind |
| Command binding | `{x:Bind XCommand}` — not an `ICommand` |
| Event handlers | Handler in XAML with no implementation |
| DataContext | Window/UserControl missing `DataContext` |
| Navigation | Success path doesn't open target Window |
| Responsive layout | Hardcoded pixel widths for fluid columns |

### Check 2: Accessibility (WCAG 2.1 AA)

| Rule | Requirement |
|---|---|
| Color contrast | ≥ 4.5:1 for all text |
| `AutomationProperties.Name` | All interactive Syncfusion controls |
| `AutomationProperties.HelpText` | Controls needing extra context |
| Keyboard navigation | Logical tab order; Enter/Space on buttons |
| Focus visibility | Visible indicator; ≥ 3:1 contrast |
| No color-only state | Errors via text/icon, not color alone |
| Touch targets | ≥ 44×44 DIP |

### Check 3: Security

| Rule | Fail Condition |
|---|---|
| No hardcoded secrets | Passwords, API keys, connection strings in XAML/C# |
| No unsafe XamlReader | `XamlReader.Load()` on user input |
| Input validation | User input not validated before use |
| License key | `SYNCFUSION_LICENSE_KEY` from env var only |

### Check 4: Performance

| Rule | Requirement |
|---|---|
| List virtualization | `SfDataGrid`/`SfTreeView` 100+ items — enabled |
| Async operations | >100ms → `async/await`; no `Task.Wait()` on UI thread |
| DPI-aware sizing | All sizes in DIP; no hardcoded pixels |
| Memory leaks | Event handlers unsubscribed on `Unloaded` |

### Check 5: Resource Integrity

| Rule | Fail Condition |
|---|---|
| `{StaticResource X}` defined | Key not in merged ResourceDictionaries |
| `{ThemeResource X}` defined | Key not resolvable by WinUI theme system |
| No duplicate `x:Key` | Same key defined twice in same file |
| Valid ARGB colors | Not `#AARRGGBB` or `#RRGGBB` |
| Relative paths only | ResourceDictionary Source uses absolute path |
| Converters declared | Converter in XAML not in ResourceDictionary |

### Check 6: Syncfusion Integration

| Rule | Requirement |
|---|---|
| License registered | `RegisterLicense()` in `OnLaunched()` before any control |
| Controls from skill files | No invented APIs; verified in `getting-started.md` |
| WinUI namespaces | `using:Syncfusion.UI.Xaml.<ControlNamespace>` format |
| NuGet packages | All from `skill-extraction.json` at matching version |

---

## Validation Report

```
╔═════════════════════════════════════════════════════════════╗
║                 STAGE 7: VALIDATION REPORT                  ║
╠═════════════════════════════════════════════════════════════╣
║  ⛔ MANDATORY GATE                                           ║
║  Pre-Check 0 — Startup View Config      ✅ PASS / ❌ FAIL  ║
╠═════════════════════════════════════════════════════════════╣
║  PHASE 1 — PRE-COMPILATION                                  ║
║  Pre-Check 1 — Skill API Validation     ✅ PASS / ❌ FAIL  ║
║  Pre-Check 2 — WinUI SDK Compliance     ✅ PASS / ❌ FAIL  ║
║  Pre-Check 3 — XAML Structure           ✅ PASS / ❌ FAIL  ║
║  Pre-Check 4 — Runtime Error Prevention ✅ PASS / ❌ FAIL  ║
║  Pre-Check 5 — NuGet Dependencies       ✅ PASS / ❌ FAIL  ║
║  Pre-Check 6 — MVVM & Bindings          ✅ PASS / ❌ FAIL  ║
║  Build Tool  (VS2026/VS2022)            ✅ PASS / ❌ FAIL  ║
║  MSBuild     (build.log)                ✅ PASS / ❌ FAIL  ║
╠═════════════════════════════════════════════════════════════╣
║  PHASE 2 — POST-BUILD QUALITY                               ║
║  Check 1 — WinUI & MVVM Standards       ✅ PASS / ❌ FAIL  ║
║  Check 2 — Accessibility WCAG AA        ✅ PASS / ❌ FAIL  ║
║  Check 3 — Security                     ✅ PASS / ❌ FAIL  ║
║  Check 4 — Performance                  ✅ PASS / ❌ FAIL  ║
║  Check 5 — Resource Integrity           ✅ PASS / ❌ FAIL  ║
║  Check 6 — Syncfusion Integration       ✅ PASS / ❌ FAIL  ║
╠═════════════════════════════════════════════════════════════╣
║  Issues: <count> — [check #, rule, fix]                     ║
╠═════════════════════════════════════════════════════════════╣
║  OVERALL: ✅ PASS → Stage 8 · ❌ FAIL → Fix and re-run     ║
╚═════════════════════════════════════════════════════════════╝
```

---

## User Decision

```
  [A] Auto-fix all issues → re-run all checks → Stage 8
  [B] Fix manually → re-run on request
  [C] Proceed to Stage 8 anyway (warn if FAIL)
  [D] Cancel
```

---

## Proceed to Stage 8 Criteria

✅ **Proceed if:** Pre-Check 0 PASS · Phase 1 all PASS · Build tool resolved · MSBuild 0 errors · Phase 2 all PASS (or user confirms C)

⛔ **Do not proceed if:** Pre-Check 0 FAIL · Build tool not found · Build fails after 3 recovery cycles · Any Phase 1 unresolved · User has not reviewed report

**CRITICAL:** Application is NOT COMPLETE until Pre-Check 0 passes and MSBuild succeeds.