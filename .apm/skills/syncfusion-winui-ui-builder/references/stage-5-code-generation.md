# Stage 5: Code Generation

**Purpose:** Generate production-ready WinUI code (XAML + C# + ViewModel + Services) that is fully wired, compilable, and feature-complete using only skill-file-verified APIs.

**Inputs:** `control-mapping.json` (Stage 3) · `skill-extraction.json` (Stage 5B) · **Stage 4 design decisions** (colors, spacing, typography, MVVM mapping, startup view plan)
**Output:** Complete UI + backend — zero stubs, zero missing handlers, zero assumed APIs. **All code uses Stage 4 tokens & MVVM mapping.**

⛔ **CRITICAL: Read Stage 4 outputs FIRST. No file may be written until all pre-validation gates below pass.**

---

## Workflow Summary

```
Step 1 → Read & validate control-mapping.json
    ↓
Step 1A → Classify each control: SYNCFUSION_VERIFIED or NATIVE_FALLBACK
    ↓                                    ↓
Step 2 (Syncfusion path)        Step 1B (Native fallback path)
Skill folder + getting-started  Use native WinUI SDK controls only
namespace, APIs, NuGet          Standard WinUI properties only
❌ HALT if any missing          Skip Syncfusion extraction
    ↓                                    ↓
    └──────────────── merge ─────────────┘
    ↓
Step 3 → Confirm skill-extraction.json (PASS) · re-read skill file per control
Step 4 → Confirm target framework = WinUI
Step 5 → Validate all namespaces (5A) and properties (5B)
    ↓
Generate: XAML · Code-Behind · ViewModel · Service · Repository · ResourceDictionary
    ↓
Post-Validation: Run all 11 checks → fix failures
    ↓
✅ Pass to Stage 6
```

---

## Pre-Validation Gates (MANDATORY — Execute in Order)

### Step 0 — Read Stage 4 Design Decisions (GATES ALL STEPS) ⛔ CRITICAL
- **Load**: Stage 4 document (MVVM mapping, color tokens, spacing grid, typography scale, startup view plan)
- **Extract & store locally**:
  - ✅ Startup view name: e.g., "MainWindow with MainWindowViewModel"
  - ✅ MVVM mapping: all View → ViewModel pairs + navigation flow
  - ✅ Color palette: Primary, Secondary, Background, Semantic colors (from Stage 4 Section 3)
  - ✅ Spacing tokens: SpaceSmall, SpaceMedium, SpaceLarge, etc. (from Stage 4 Section 4)
  - ✅ Typography scale: FontSizeBody, FontSizeHeading, etc. (from Stage 4 Section 4)
- ❌ If Stage 4 document missing or incomplete → HALT: "Stage 4 not locked; cannot generate"
- ✅ If all extracted → Proceed to Step 1

### Step 1 — Validate `control-mapping.json`

- Locate: `<project-root>/control-mapping.json`
- Simple → `elements[]` present; Complex → `pages[]` with `page_id`, `component_type`, `elements`/`sections`
- Every element must have: `id`, `name`, `type_hint`, mapped control name
- ⛔ Missing or invalid → HALT: return to Stage 3

---

### Step 1A — Control Classification Gate

```
FOR EACH entry in mapped_controls[]:

  IF validation == "✓ VERIFIED" AND skill != null AND score >= 2
  → SYNCFUSION_VERIFIED: proceed to Step 2

  IF control == "NATIVE_XAML" OR validation == "✗ FALLBACK"
  OR validation == "✗ NO_MATCH" OR skill == null OR score < 2
  → NATIVE_FALLBACK: proceed to Step 1B
  → Log: "⚠️ Fallback: <element_name> → native WinUI control"
```

---

### Step 1B — Native Fallback (NATIVE_FALLBACK controls only)

| Intent | Native WinUI Control | Key Properties |
|--------|---------------------|----------------|
| Text input | `TextBox` | `Text`, `PlaceholderText`, `Header`, `IsReadOnly` |
| Password | `PasswordBox` | `Password`, `PlaceholderText`, `PasswordRevealMode` |
| Dropdown | `ComboBox` | `Items`, `SelectedItem`, `PlaceholderText` |
| Button | `Button` | `Content`, `Command`, `IsEnabled` |
| Checkbox | `CheckBox` | `Content`, `IsChecked`, `IsEnabled` |
| Toggle | `ToggleSwitch` | `Header`, `IsOn`, `OnContent`, `OffContent` |
| Date | `CalendarDatePicker` | `Date`, `Header`, `MinDate`, `MaxDate` |
| Number | `NumberBox` | `Value`, `Minimum`, `Maximum`, `PlaceholderText` |

**Rules:**
- ✅ Use standard WinUI SDK properties only — no Syncfusion namespaces or properties
- ✅ Allowed: no valid Syncfusion mapping (✗ NO_MATCH), score < 2, skill file missing
- ❌ Not allowed: valid Syncfusion control exists → fix properties instead of falling back

---

### Step 2 — Atomic Skill Validation (SYNCFUSION_VERIFIED controls only)

```
FOR EACH SYNCFUSION_VERIFIED control:
  1. LOCATE skill folder: <skills-root>/syncfusion-winui-<control-name>/
     ❌ Not found → HALT: "Skill folder missing for <control-name>"
  2. READ <skill-folder>/references/getting-started.md
     ❌ Not found → HALT: "getting-started.md missing for <control-name>"
  3. EXTRACT namespace → ❌ absent → HALT: "Namespace undefined for <control-name>"
  4. EXTRACT + VERIFY properties, events, methods
     ❌ API not listed → HALT: "Unverified API for <control-name>"
  5. EXTRACT NuGet package + version
     ❌ Not listed → HALT: "Unknown package for <control-name>"
     ❌ Version mismatch → HALT: "Version conflict for <control-name>"
  6. IF advanced features needed: READ SKILL.md + relevant feature guide
     ❌ Not found → HALT: "Feature guide missing"

ANY failure → HALT entire generation; report all failed controls.
```

> **Skill files are the single source of truth. No code may be generated from memory, assumption, or inference.**

---

### Step 2B — XAML Pre-Generation Dry-Run Validation (NEW) ⛔ CRITICAL
**Before writing ANY XAML file:**
```
FOR EACH planned XAML structure:
  · Simulate namespace parsing (no duplicates, all from skill-extraction.json or WinUI base)
  · Simulate resource key resolution: every {StaticResource X} must be defined in some ResourceDictionary
  · Simulate control property assignments: every property valid for element type
  · Simulate binding resolution: {x:Bind Y} — Y exists in ViewModel or code-behind
```
- ✅ Parse succeeds → Proceed to Step 3
- ❌ Parse fails → Report error; HALT; do NOT generate

### Step 3 — PRE-GENERATION SKILL VALIDATION (BLOCKING) ⛔ CRITICAL

**This step MUST complete successfully BEFORE ANY code file is written.**

```
CONFIRM skill-extraction.json exists at <project-root>/
  ❌ Missing → HALT: "skill-extraction.json not found — run Stage 5B first"

CONFIRM validation_status == "PASS"
  ❌ Not PASS → HALT: "Extraction not validated — re-run Stage 5B"

FOR EACH Syncfusion control in control-mapping.json (BEFORE generating ANY file):
  1. FIND entry in skill-extraction.json → controls[] where control == "<ControlName>"
     ❌ Not found → HALT: "Do NOT invent <ControlName> — add to mapping and re-run Stage 5B"

  2. RE-READ skill file listed in controls[].sources_read[0]
     ❌ Unreadable → HALT: "Skill reference unreadable for <ControlName>"

  3. EXTRACT and VERIFY (from re-read skill file):
     ✅ Namespace: exactly matches controls[].namespace
     ✅ All properties used in control-mapping.json exist in controls[].valid_properties[]
     ✅ All events used exist in controls[].valid_events[]
     ✅ All methods used exist in controls[].valid_methods[]
     ✅ NuGet package: controls[].nuget_package + version exact
     ❌ ANY mismatch → HALT: "API mismatch — <PropertyName> not found in skill file for <ControlName>"

  4. BUILD a verified API registry (in memory):
     {
       "<ControlName>": {
         "namespace": "<verified_from_skill>",
         "verified_properties": [ ... ],
         "verified_events": [ ... ],
         "verified_methods": [ ... ],
         "nuget_package": "<verified>"
       }
     }

✅ ALL controls verified → Registry complete → Proceed to Step 4 code generation
❌ ANY control fails → HALT; do NOT write ANY file
```

**Critical Rule:** No code may reference ANY API not in this registry. If API not found in skill file, it does NOT exist — add to control-mapping.json and re-run Stage 5B.

---

### Step 3A — Read `skill-extraction.json` + Skill Files (Validation Gate)

```
USE verified API registry from Step 3 (above) to validate all generated code.

FOR EACH Syncfusion control in any generated file:
  CROSS-CHECK against verified registry:
  · namespace   → exact controls[].namespace (never modify or construct)
  · properties  → only names in controls[].valid_properties[].name
  · events      → only names in controls[].valid_events[].name
  · methods     → only names in controls[].valid_methods[].name
  · nuget       → controls[].nuget_package at controls[].nuget_version

✅ All cross-checks pass → safe to generate
❌ ANY cross-check fails → code generation BLOCKED; report failure
```

---

### Step 4 — Detect Target Framework (BLOCKING)

```
READ Stage 2 → target_framework, platform, dotnet_version

❌ target_framework empty/null → HALT: "Target SDK unknown — re-run Stage 2"
❌ platform ≠ WinUI           → HALT: "Platform is not WinUI"
❌ framework contains 'wpf', 'maui', 'uwp', 'android' → HALT: "Non-WinUI framework detected"

LOCK:
✅ WinUI base namespace: "http://schemas.microsoft.com/winfx/2006/xaml/presentation"
✅ Syncfusion namespaces: from skill-extraction.json → controls[].namespace only
```

---

### Step 5 — Namespace & Property Compatibility (BLOCKING)

#### 5A — Namespace Validation
```
FOR EACH xmlns in planned XAML:
  ✅ Default xmlns MUST be: xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  ✅ Syncfusion xmlns MUST come from: skill-extraction.json → controls[].namespace
  ✅ Format: xmlns:sf="using:Syncfusion.UI.Xaml.<ControlNamespace>"
  ❌ 'clr-namespace', 'assembly=', 'schemas.syncfusion.com/wpf' → HALT: "WPF namespace in WinUI file"
  ❌ Namespace not in WinUI base set AND not in skill-extraction.json → HALT: "Unverified namespace"
```

#### 5B — Property Validation
```
FOR EACH property on any WinUI/Syncfusion control:

  CHECK 1 — WPF blocked properties (HALT if found):
  · {Binding} → use {x:Bind}
  · DockPanel → use Grid or RelativePanel
  · WrapPanel → use ItemsWrapGrid
  · Triggers/DataTrigger → use VisualStateManager
  · System.Windows.* types → use Microsoft.UI.Xaml.*
  · clr-namespace/assembly= xmlns → use using: form

  CHECK 2 — Element-level support:
  · Padding   ✅ Control subclasses, Border  ❌ Grid, StackPanel, Canvas
  · CornerRadius ✅ Border, Control          ❌ Grid, StackPanel
  · Background ✅ Panel, Control, Border     ❌ Plain UIElement/FrameworkElement
  ❌ Property not declared on element → HALT: "Property does not exist on <ElementType> in WinUI SDK"

  CHECK 3 — Syncfusion property:
  ❌ Property not in skill-extraction.json → valid_properties[].name → HALT: "Property not verified in skill file"
```

**Quick-fix reference:**

| Error | Fix |
|-------|-----|
| `Padding` on `Grid`/`StackPanel` | Wrap in `Border` with `Padding` or use `Margin` on children |
| `CornerRadius` on `Grid` | Wrap in `Border` with `CornerRadius` |
| `{Binding}` in XAML | Replace with `{x:Bind}` |

---

## Gate Check (MANDATORY before writing any file)

```
✅ Stage 4 design decisions extracted + stored (Step 0)
✅ XAML dry-run validation passed (Step 2B)
✅ skill-extraction.json exists + validation_status == "PASS"
✅ Every Syncfusion control verified against its skill file (Step 3)
✅ No control absent from skill-extraction.json appears in output
✅ Target framework = WinUI confirmed (Step 4)
✅ All xmlns from skill-extraction.json only — no WPF/UWP/MAUI namespaces (Step 5A)
✅ All properties validated against WinUI SDK + skill-extraction.json (Step 5B)
✅ Startup View will be generated: App.xaml.cs + MainWindow pattern confirmed
✅ MVVM mapping matches Stage 4 exactly
✅ Design tokens (colors, spacing, fonts) from Stage 4 are available for use
✅ ResourceDictionary merge structure planned (Themes/*.xaml in App.xaml)
```

---

## Validation Registries (Build Before Code Generation)

Build four registries from `skill-extraction.json`. Any `BlockingException` halts generation — no catch, no suppression.

```
validControls    = SET  { controls[].control }
propertyRegistry = MAP  { controlName → SET { valid_properties[].name } }
eventRegistry    = MAP  { controlName → SET { valid_events[].name } }
namespaceRegistry= MAP  { controlName → controls[].namespace }

CALL POINTS (mandatory — no exceptions):
① Before any XAML element tag    → validateControl(className)
② Before any Syncfusion attribute → validateProperty(className, attrName)
③ Before any Syncfusion event     → validateEvent(className, eventName)
④ Before any xmlns declaration    → validateNamespace(className, xmlnsValue)

Any BlockingException → HALT; log full message; do NOT continue.
```

**Data source rules — absolute:**

| Data | Source | ❌ Never From |
|------|--------|---------------|
| `xmlns:...` | `controls[].namespace` in `skill-extraction.json` | Memory or inference |
| Properties | `controls[].valid_properties[].name` | Assumption |
| Events | `controls[].valid_events[].name` | Guessing |
| NuGet package/version | `controls[].nuget_package` + `nuget_version` | Any other source |
| Control class name | `controls[].control` | Aliases or abbreviations |

---

## Code Generation Deliverables

Generate all layers together. Never generate UI without the backend.

### Folder Structure
```
App.xaml + App.xaml.cs  # ⛔ MANDATORY: Startup entry point with license registration
MainWindow.xaml + MainWindow.xaml.cs  # ⛔ MANDATORY: Root window with root ViewModel DataContext
Views/<Feature>/        # .xaml + .xaml.cs (for each planned View from Stage 4 MVVM mapping)
ViewModels/             # INotifyPropertyChanged ViewModels (matching Stage 4 mapping)
Services/               # Business logic
Repositories/           # IRepository + in-memory implementation
Models/                 # Data models and DTOs
Themes/                 # Colors.xaml · Spacing.xaml · Typography.xaml (MUST be merged in App.xaml)
```
⛔ **CRITICAL**: If MainWindow.xaml or App.xaml missing → FAIL Stage 5

### Deliverable 1: XAML
- Namespaces from `skill-extraction.json` only — one prefix per namespace, no duplicates.
- All Syncfusion xmlns: `using:Syncfusion.UI.Xaml.<ControlNamespace>` format.
- Only controls/properties verified by the four registries.
- All interactive controls: event/command bindings + `AutomationProperties.Name`.
- Layout: `Grid` with `*` sizing — no hardcoded pixel widths for fluid areas.
- App.xaml resources:
```xaml
<Application.Resources>
  <ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
      <ResourceDictionary Source="Themes/Colors.xaml" />
      <ResourceDictionary Source="Themes/Spacing.xaml" />
      <ResourceDictionary Source="Themes/Typography.xaml" />
    </ResourceDictionary.MergedDictionaries>
  </ResourceDictionary>
</Application.Resources>
```

### Deliverable 2: Code-Behind (.xaml.cs)
- `InitializeComponent()` first; `DataContext = new <Feature>ViewModel()` immediately after.
- All event handlers fully implemented — never empty stubs.
- App.xaml.cs license bootstrap:
```csharp
protected override void OnLaunched(Microsoft.UI.Xaml.LaunchActivatedEventArgs args)
{
    SyncfusionLicenseProvider.RegisterLicense(
        Environment.GetEnvironmentVariable("SYNCFUSION_LICENSE_KEY"));
    m_window = new MainWindow();
    m_window.Activate();
}
```

### Deliverable 3: ViewModel (.cs)
- **Implements `INotifyPropertyChanged`**; all properties raise `OnPropertyChanged`.
- **All commands: `RelayCommand`** with `CanExecute` + `Execute`.
- **Root ViewModel (for MainWindow)**: implements navigation orchestration per Stage 4 flow.
- **Feature ViewModels**: match View names from Stage 4 MVVM mapping exactly.
- Input validation; error message property bound to XAML.
- Calls Service layer — no inline business logic.

### Deliverable 4: Service & Repository
- Service: all business logic (e.g., `AuthService.ValidateCredentials`).
- Repository: `IRepository` interface + in-memory implementation.
- Navigation: success → open target Window, close current; failure → surface error via ViewModel.
- Server-side validation independent of UI.

### Deliverable 5: Navigation Bridge Registration ⛔ CRITICAL

**If any generated page is not the startup view (Step 0), register it in the navigation system:**

```
FOR EACH generated View/ViewModel pair (non-startup pages only):
  1. Create navigation command in MainWindowViewModel:
     - RelayCommand<Type> Navigate<PageName>Command
     - Execute: new <PageName>() → show as Window or transition
     - Register in navigation dictionary: "PageName" → <PageName> constructor

  2. IF multi-page in same window (MVVM Region pattern):
     - Add Content property to MainWindowViewModel
     - Binding: <ContentControl Content="{x:Bind CurrentPage}" />
     - NavigateTo() updates CurrentPage property → raises OnPropertyChanged

  3. IF separate windows (Modal/Dialog pattern):
     - Create command to instantiate + open new window
     - Wire navigation in command Execute() only
     - ViewModel never directly creates windows

  4. Document in navigation metadata:
     - Page name, route, accessible from (parent page), back navigation

✅ All non-startup pages registered → proceed to ResourceDictionary (Deliverable 5)
❌ Page not registered → HALT: "Page orphaned in navigation system"
```

**Critical Rule:** Every generated page must have a path from the startup view. Orphaned pages cause runtime navigation failures.

---

### Deliverable 5: ResourceDictionary ⛔ MANDATORY
Create three files using Stage 4 tokens:
```xaml
<!-- Themes/Colors.xaml — from Stage 4 Section 3 color definitions -->
<SolidColorBrush x:Key="PrimaryColorBrush" Color="#007ACC" />
<SolidColorBrush x:Key="BackgroundColorBrush" Color="#FFFFFF" />
<!-- Add all Stage 4 semantic colors here -->

<!-- Themes/Spacing.xaml — from Stage 4 Section 4 spacing grid -->
<x:Double x:Key="SpaceSmall">8</x:Double>
<x:Double x:Key="SpaceMedium">12</x:Double>

<!-- Themes/Typography.xaml — from Stage 4 Section 4 typography scale -->
<x:Double x:Key="FontSizeBody">12</x:Double>
<x:Double x:Key="FontSizeHeading">18</x:Double>
```
**App.xaml MUST merge all three:**
```xaml
<ResourceDictionary.MergedDictionaries>
  <ResourceDictionary Source="Themes/Colors.xaml" />
  <ResourceDictionary Source="Themes/Spacing.xaml" />
  <ResourceDictionary Source="Themes/Typography.xaml" />
</ResourceDictionary.MergedDictionaries>
```
❌ If any merge missing → XAML parse exception at runtime

---

## Post-Generation Validation (MANDATORY — Fix All Before Stage 6)

| # | Check | Fail Condition |
|---|-------|----------------|
| 1 | Control scope | Any control not in `skill-extraction.json → controls[]` |
| 2 | Namespace | Namespace not from `controls[].namespace`; duplicates or constructed prefixes |
| 3 | Property & event | Not in `valid_properties`/`valid_events`; event in XAML with no handler |
| 4 | No empty handlers | Any event handler or command Execute is a stub |
| 5 | DataContext | Any Window/UserControl missing `DataContext` assignment |
| 6 | Binding resolution | `{x:Bind X}` where `X` not in ViewModel or code-behind |
| 7 | Command resolution | `{x:Bind XCommand}` where `XCommand` not an `ICommand` |
| 8 | Service completeness | Service method called from ViewModel but not implemented |
| 9 | Navigation | Complex layout: success path does not open target Window |
| 10 | Resource integrity | `{StaticResource X}` key not defined; ResourceDictionary not merged; duplicate `x:Key`; invalid ARGB |
| 11 | Component type | `generate_window` → `Window`; `generate_usercontrol` → `UserControl` |
| 12 | ⛔ Startup View | App.xaml.cs + MainWindow.xaml + MainWindow.xaml.cs exist; DataContext set to root ViewModel |
| 13 | ⛔ Stage 4 MVVM Mapping | ViewModels match Stage 4 MVVM mapping exactly; navigation commands follow Stage 4 flow |
| 14 | ⛔ Stage 4 Design Tokens | All colors use Stage 4 palette; spacing uses Stage 4 grid; fonts use Stage 4 scale |
| 15 | ⛔ ResourceDictionary Merge | Themes/Colors.xaml, Themes/Spacing.xaml, Themes/Typography.xaml created & merged in App.xaml |

⛔ **Failure on ANY check → HALT; fix before Stage 6. CRITICAL: Checks 12-15 are new gates.**

---

## MANDATORY Rules

| Rule | Enforcement |
|------|-------------|
| Never guess a control name | HALT — read skill file first |
| Never guess a property/event | HALT — validate against `valid_properties[]` / `valid_events[]` |
| Properties only from code blocks in skill file | REJECT if from plain text |
| `skill-extraction.json` must exist and PASS | HALT if missing or status ≠ PASS |
| Syncfusion namespace from skill file only | HALT if constructed or guessed |
| No WPF-style namespaces (`clr-namespace`, `assembly=`) | HALT if detected |
| Never generate UI without backend | Generate all layers together |
| No business logic in code-behind | Move to ViewModel/Service |

---

## DO ✅ / DON'T ❌

**DO:**
- ✅ **Read Stage 4 design decisions FIRST** (Step 0) — extract tokens before any code generation.
- ✅ **Generate MainWindow.xaml + App.xaml.cs** — startup view is MANDATORY, not optional.
- ✅ **Use Stage 4 tokens**: `{StaticResource PrimaryColorBrush}` not `#007ACC`; `{StaticResource SpaceLarge}` not `16`.
- ✅ **Create & merge Themes/Colors.xaml, Themes/Spacing.xaml, Themes/Typography.xaml** in App.xaml.
- ✅ **Match Stage 4 MVVM mapping exactly** — View → ViewModel pairs, navigation flow.
- ✅ Read skill file before referencing any control, property, event, or namespace.
- ✅ Build all four registries before writing any file.
- ✅ Use `{x:Bind}` for all bindings.
- ✅ Generate UI + backend together as one cohesive feature.

**DON'T:**
- ❌ **Skip Step 0** — Stage 4 decisions MUST be read before generation.
- ❌ **Omit MainWindow.xaml or App.xaml.cs** — application will open empty window.
- ❌ **Hardcode colors/spacing** — use Stage 4 tokens via `{StaticResource}`.
- ❌ **Skip ResourceDictionary merge** — XAML parse exception at runtime if not merged.
- ❌ Guess or assume any control name, namespace, property, or method.
- ❌ Use `{Binding}`, `DockPanel`, `WrapPanel`, `Triggers`, or `System.Windows.*`.
- ❌ Leave any event handler or command Execute as an empty stub.
- ❌ Generate code before all pre-validation steps pass.

---

## Code Generation Standards

| Standard | Rule |
|----------|------|
| Syncfusion APIs | Only from `skill-extraction.json → valid_properties/events/methods` |
| Native APIs | WinUI SDK properties only (no Syncfusion props on native controls) |
| NuGet packages | Only from `skill-extraction.json → nuget_package + nuget_version` |
| Fallback safety | Native fallback only when score < 2 or ✗ NO_MATCH |
| MVVM | Business logic → Service; coordination → ViewModel; UI-only → code-behind |
| Accessibility | `AutomationProperties.Name` + `HelpText`; min 44×44 DIP touch target |
| Bindings | `{x:Bind Prop, Mode=TwoWay}` for inputs; `{x:Bind Prop, Mode=OneWay}` for display |
| Responsive | `Grid` with `*` sizing; never hardcode fluid column widths |
| Performance | Virtualization on `SfDataGrid` for large lists; `async/await` for I/O |
| Security | No hardcoded credentials or secrets in XAML or code-behind |