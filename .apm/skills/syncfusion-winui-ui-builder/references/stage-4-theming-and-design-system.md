# Stage 4: Design System Selection

**Purpose:** Lock all design system decisions before Stage 5 code generation. This stage is decision-making only — no code is generated here.

**Input:** Stage 2 (project config, .NET version) + Stage 3 (control-mapping.json)
**Output:** All decisions below locked and ready for Stage 5 implementation.

---

## Table of Contents

1. [WinUI Application Philosophy](#1-winui-application-philosophy)
2. [Color Mode Selection](#2-color-mode-selection)
3. [Color System Architecture](#3-color-system-architecture-mandatory)
4. [Spacing & Typography Systems](#4-spacing--typography-systems-mandatory)
5. [Responsive Strategy (DPI-Aware Scaling)](#5-responsive-strategy-dpi-aware-scaling-mandatory)
6. [Motion & Accessibility Standards (MANDATORY)](#6-motion--accessibility-standards-mandatory)
7. [XAML Styling Token Architecture](#7-xaml-styling-token-architecture-critical)
8. [Syncfusion WinUI Control Integration](#8-syncfusion-winui-control-integration-mandatory)
9. [MVVM Integration Bridge](#9-mvvm-integration-bridge-critical)
10. [Load Your Application Reference](#10-load-your-application-reference-mandatory)
11. [Stage 4 Decision Checklist (ENFORCED)](#11-stage-4-decision-checklist-enforced)
12. [Stage 5 Integration Guidance](#12-stage-5-integration-guidance)

---

## ⛔ ERROR HANDLING: Resource Issues

**Common errors in Stage 4-5:**
- ❌ Resource key not found in `ResourceDictionary`
- ❌ Build fails: "Type not found" for custom resources

**Root cause:** Missing resource key definitions or incorrect `ResourceDictionary` merge

**Mandatory fixes:**
1. ✅ Define all custom app resources in `App.xaml` `<Application.Resources>` or merged custom resource files
2. ✅ Use `{ThemeResource}` for system-provided brush/color keys; use `{StaticResource}` for custom app resources
3. ❌ NEVER reference a resource key that has not been explicitly defined in a loaded `ResourceDictionary`
4. ✅ If custom app resources needed: Define ONLY in `App.xaml` or separate non-theme files
5. ⛔ If build fails: Check skill file for correct resource key names before modifying code

---

## 1. WinUI Application Philosophy

Confirm the application type detected in Stage 2. This choice drives all downstream color, and layout decisions.

| Application Type | Design Priority |
|---|---|
| **Enterprise** | Data density, task efficiency |
| **Consumer** | Clarity, modern aesthetics |
| **LOB** | Domain workflows, expert users |
| **Creative/Tool** | Dark mode, visual customization |

**Rules:**
- ✅ Confirm or override Stage 2's detected type — document the reason if overriding
- ❌ Do not mix application philosophies (e.g., consumer simplicity in an enterprise data grid)
- ✅ Proceed to Section 9 after confirming application type

---

## 2. Color Mode Selection

Select the color mode. This choice determines whether Light, Dark, or both resource dictionaries are generated in Stage 5.

| Scenario | Color Mode |
|---|---|
| Windows 11 light appearance | Light |
| Windows 11 dark appearance | Dark |
| Follows OS light/dark preference | System default (both Light and Dark resources defined) |

**Rules:**
- ✅ Light only → generate only Light resource tokens
- ✅ Dark support → plan `Themes/DarkTheme.xaml` overrides; apply via `RequestedTheme` on the root element
- ❌ Do not mix Light and Dark resource tokens in the same resource file

---

## 3. Color System Architecture (MANDATORY)

**VALIDATION:** ⛔ **If color tokens are not defined → FAIL: "Design system incomplete"**

Define a semantic color palette. Apply as `SolidColorBrush` resources in `Themes/Colors.xaml`.

**Required color roles (document all):**
- **Primary** — brand color, CTAs, key actions
- **Secondary** — supporting actions, highlights
- **Background** — default surface color
- **Semantic** — success (`#4CAF50`), warning (`#FFC107`), error (`#F44336`), info (`#2196F3`)
- **Neutral scale** — text, backgrounds, borders
- **Surface** — cards, containers (optional; use neutrals if sufficient)

**Mandatory checks:**
- ✅ Define all PRIMARY, SECONDARY, BACKGROUND, and SEMANTIC color tokens in a documented table
- ✅ Use `{ThemeResource}` for system-provided colors; `{StaticResource}` for custom app colors
- ✅ Use tinted neutrals that lean toward your brand hue
- ✅ Verify contrast ≥ 4.5:1 for all text and UI controls (WCAG AA)
- ✅ Document dark mode decision explicitly: Light only / Dark support / System default
- ❌ Do not hardcode hex values directly in XAML controls
- ❌ Do not deviate from semantic naming convention

**Dark mode decision (MANDATORY — document choice now):**
- Light only → no action needed
- Dark support → plan `Themes/DarkTheme.xaml` overrides; apply via `RequestedTheme` on the root element

---

## 4. Spacing & Typography Systems (MANDATORY)

**VALIDATION:** ⛔ **If inconsistent sizing detected → FAIL**

### Spacing (DPI-Aware)

Use a 4pt base grid defined in `Themes/Spacing.xaml`. Document all tokens explicitly.

```xaml
<x:Double x:Key="SpaceXSmall">4</x:Double>
<x:Double x:Key="SpaceSmall">8</x:Double>
<x:Double x:Key="SpaceMedium">12</x:Double>
<x:Double x:Key="SpaceLarge">16</x:Double>
<x:Double x:Key="SpaceXLarge">24</x:Double>
```

- ✅ Define all spacing tokens before Stage 5 generation
- ❌ Never hardcode pixel values in XAML controls

### Typography (MANDATORY)

Define consistent modular scale in `Themes/Typography.xaml`. Recommended ratio: **1.25** (major third).

```xaml
<FontFamily x:Key="FontFamilyDefault">Segoe UI Variable</FontFamily>
<x:Double x:Key="FontSizeSmall">10</x:Double>
<x:Double x:Key="FontSizeBody">12</x:Double>
<x:Double x:Key="FontSizeLarge">14</x:Double>
<x:Double x:Key="FontSizeHeading">18</x:Double>
<x:Double x:Key="FontSizeTitle">24</x:Double>
```

- ✅ Define line heights for body (1.4–1.6) and headings (1.2) explicitly
- ✅ Minimum body font: 12pt (96 DPI baseline)
- ❌ Do not use inconsistent font sizes across similar control types

---

## 5. Responsive Strategy (DPI-Aware Scaling) (MANDATORY)

**VALIDATION:** ⛔ **If DPI strategy missing → FAIL**

**Design approach:** Fluid layouts — no fixed resolutions. Document your strategy now.

| Window Category | Min Size | Use Case |
|---|---|---|
| Small | 600×400 | Dialogs, utilities |
| Medium | 1024×768 | Standard business apps |
| Large | 1280×1024+ | Dashboards, multi-pane |

**MANDATORY layout strategy (choose one or combine):**
- ✅ `Grid` with `*` star sizing for flexible multi-column layouts
- ✅ `StackPanel` for single-column or narrow views
- ✅ `RelativePanel` for adaptive, constraint-based layouts
- ✅ Document which panels are used in your main view

**DPI awareness plan:**
- ✅ WinUI handles per-monitor DPI automatically via Windows App SDK
- ✅ Set minimum window dimensions only where usability requires it
- ❌ Do not use hardcoded pixel widths for layout columns

---

## 6. Motion & Accessibility Standards (MANDATORY)

**VALIDATION:** ⛔ **If accessibility is not defined → FAIL**

### Animation Timing (MANDATORY)

| Duration | Use |
|---|---|
| 100ms | Hover states, instant feedback |
| 300ms | Transitions, dropdowns, state changes |
| 500ms | Major layout reveals |

- ✅ Document your animation strategy (use above timing or justify alternatives)
- ✅ Respect `prefers-reduced-motion` — set duration to 0ms when OS enables this setting (WCAG requirement)
- ❌ Do not animate for aesthetics alone

### Accessibility (MANDATORY)

- ✅ Minimum touch/click target: **44×44 device-independent units** — document all interactive control sizes
- ✅ Minimum spacing between targets: **8px**
- ✅ Color contrast: **≥ 4.5:1** for all text and UI controls (WCAG 2.1 AA) — document verification
- ✅ Apply `AutomationProperties.Name` and `AutomationProperties.HelpText` on all interactive controls
- ✅ Plan keyboard navigation: tab order strategy and focus ring visibility
- ❌ Do not assume accessibility can be added later — it must be part of initial design

---

## 7. XAML Styling Token Architecture (CRITICAL)

**VALIDATION:** ⛔ **If styles are not centralized → FAIL: "Inline styling detected"**

### Resource File Structure (CRITICAL)

- ✅ Define ALL custom app resources in `Themes/Colors.xaml`, `Themes/Spacing.xaml`, `Themes/Typography.xaml`
- ✅ Merge custom resource files into `<Application.Resources>` via `<ResourceDictionary.MergedDictionaries>`
- ✅ Use `{ThemeResource}` for system brush keys; use `{StaticResource}` for custom app resources
- ✅ Keep custom resources separate from control-library resources
- ❌ NEVER use inline styling (e.g., `Background="#FF0000"` directly on controls)

### Semantic Naming Convention (ENFORCED)

Use role-based names, not descriptive value-based names:

| ❌ Avoid | ✅ Use |
|---|---|
| `BlueColorBrush600` | `PrimaryColorBrush` |
| `PaddingValue16` | `SpaceLarge` |
| `Font14px` | `HeadingFontSize` |

### Resource Hierarchy (MANDATORY)

1. **Primitives** — base SolidColorBrush, Double spacing, FontSize values
2. **Semantic** — role-based resources (`TextColorBrush`, `ControlGap`)
3. **Control-level** — only for control-specific overrides (`ButtonPadding`)

---

## 8. Syncfusion WinUI Control Integration (MANDATORY)

**VALIDATION:** ⛔ **If control is not validated → FAIL: "Unverified control usage"**

### License Registration (MANDATORY)

**App.xaml.cs — `OnLaunched()`:**
```csharp
using Syncfusion.Licensing;

protected override void OnLaunched(Microsoft.UI.Xaml.LaunchActivatedEventArgs args)
{
    SyncfusionLicenseProvider.RegisterLicense(
        Environment.GetEnvironmentVariable("SYNCFUSION_LICENSE_KEY"));
    m_window = new MainWindow();
    m_window.Activate();
}
```

### Control Validation (MANDATORY)

- ✅ For each Syncfusion control planned: Verify it aligns with design system colors, spacing, and typography
- ✅ Confirm control version matches .NET version detected in Stage 2
- ✅ Verify correct NuGet package name: `Syncfusion.<ControlName>.WinUI`
- ❌ Do not assume a control exists without explicit validation

### Resource Coordination

- ✅ Define color overrides in `Themes/Colors.xaml` using semantic resource keys
- ✅ Reference token resources in control styles — never hardcode values
- ❌ Do not set `Background="#FF0000"` directly on controls

---

## 9. MVVM Integration Bridge (CRITICAL)

**CRITICAL VALIDATION:** 
- ⛔ **If a UI control (Button/Input) has no binding → FAIL: "Missing MVVM connection"**
- ⛔ **If navigation flow is not defined → FAIL: "No UI flow defined"**
- ⛔ **If startup page/view is not defined → FAIL: "Application will open empty window"**

### View → ViewModel Mapping (MANDATORY)

For each planned View, define its corresponding ViewModel:

| View | ViewModel | Purpose |
|---|---|---|
| `MainWindow` | `MainWindowViewModel` | Root shell, navigation orchestration |
| (Add your views) | (Add VMs) | (Add purposes) |

**Mandatory checks:**
- ✅ Every View has exactly one ViewModel
- ✅ All interactive controls have explicit bindings: `{Binding PropertyName}` or `{Binding CommandName}`
- ✅ Commands exist for ALL UI interactions (Button clicks, form submissions, etc.)
- ❌ Do not create views without a corresponding ViewModel
- ❌ Do not assume implicit bindings

### Navigation Flow (MANDATORY)

Document your application's navigation paths. Example:

```
App.xaml
  ↓
App.xaml.cs (OnLaunched) → Register License
  ↓
MainWindow (MainWindowViewModel)
  ├─→ LoginView (LoginViewModel) [if auth required]
  │    └─→ Command: LoginCommand → navigate to Dashboard
  ├─→ DashboardView (DashboardViewModel)
  └─→ SettingsView (SettingsViewModel)
```

**MANDATORY navigation plan:**
- ✅ Identify startup view that will display when app launches
- ✅ Map all navigation paths (Button → View transitions)
- ✅ Define navigation commands in each ViewModel
- ✅ Document backward navigation (back button, cancel)
- ❌ Do not create orphan views without navigation paths
- ❌ Do not leave navigation implicit

### Startup View Validation (CRITICAL)

**FAIL CONDITION:** If no startup view is defined, the application window will open empty.

- ✅ Define which View/ViewModel pair loads when `MainWindow` initializes
- ✅ Set `DataContext` of `MainWindow` to the appropriate root ViewModel
- ✅ Verify initial view renders content (not blank)
- ✅ Document the startup flow clearly in your Stage 4 plan

---

## 10. Load Your Application Reference (MANDATORY)

Based on the application type confirmed in Section 1, load the corresponding skill reference before proceeding to Stage 5. Reference files are located at any of:

```
<skills-root>/syncfusion-winui-ui-builder/references/
<skills-root>/syncfusion-winui-theming/SKILL.md
```

Where `<skills-root>` is one of: `.codestudio/skills`, `.agent/skills`, `.agents/skills`, `.github/skills`, `skills`

| Application Type | Color Mode Focus | Key Reference |
|---|---|---|
| **Enterprise** | Light — data density, professional | `references/winui-dotnet-standards.md` |
| **Consumer** | Light — modern, approachable | `references/winui-dotnet-standards.md` |
| **LOB** | Light — expert workflows, productivity | `references/winui-dotnet-standards.md` |
| **Creative** | Dark — dark mode, customization | `references/winui-dotnet-standards.md` |

**MANDATORY startup view reference:**
- ✅ Identify the application entry point (startup view defined in Section 9)
- ✅ Verify initial view loads automatically when app launches
- ⛔ If no entry reference → FAIL: "Missing application entry point"

⛔ **You cannot proceed to Stage 5 without BOTH application reference AND startup view definition.**

---

## 11. Stage 4 Decision Checklist (ENFORCED)

**HALT Stage 5 if ANY item is missing or unchecked.**

### Design System (Sections 3-7)
- ⛔ **Primary, Secondary, Background colors defined** — (MANDATORY)
- ⛔ **4pt spacing grid and typography scale documented** — (MANDATORY)
- ⛔ **DPI scaling strategy planned** — (MANDATORY)
- ⛔ **Accessibility standards defined (animation, touch targets, contrast)** — (MANDATORY)
- ⛔ **All styles centralized in resource files; no inline styling** — (CRITICAL)

### Controls & Integration (Section 8)
- ⛔ **Every Syncfusion control validated against design system** — (MANDATORY)
- ⛔ **`SyncfusionLicenseProvider.RegisterLicense()` planned in `OnLaunched()`** — (MANDATORY)
- ⛔ **All package versions match Stage 2 detection** — (MANDATORY)

### MVVM & Navigation (Section 9) — CRITICAL BLOCKERS
- ⛔ **View → ViewModel mapping complete** — (CRITICAL)
- ⛔ **Every UI control (Button, Input) has explicit binding or command** — (CRITICAL)
- ⛔ **Navigation flow defined (all paths documented)** — (CRITICAL)
- ⛔ **Startup view identified and entry point clear** — (CRITICAL)
- ⛔ **No orphan views without navigation paths** — (CRITICAL)

### Application Entry Point (Section 10)
- ⛔ **Application reference loaded** — (MANDATORY)
- ⛔ **Startup view will load when app launches** — (MANDATORY)
- ⛔ **`MainWindow` `DataContext` set to root ViewModel** — (MANDATORY)

### XMLA Token Architecture
- ⛔ **Semantic resource naming enforced (role-based, not value-based)** — (CRITICAL)
- ⛔ **Custom resources merged via `<ResourceDictionary.MergedDictionaries>`** — (MANDATORY)
- ⛔ **No resource keys referenced without explicit definition** — (CRITICAL)

---

## 12. Stage 5 Integration Guidance

**Stage 5 generates code ONLY from Stage 4 decisions — no inferences or assumptions.**

| Stage 4 Decision | Stage 5 Output | CRITICAL RULE |
|---|---|---|
| Color tokens (Primary, Secondary, Background) | `Themes/Colors.xaml` with `SolidColorBrush` definitions | ❌ Do NOT infer missing colors |
| Spacing & typography scale | `Themes/Spacing.xaml`, `Themes/Typography.xaml` | ❌ Do NOT create inconsistent sizes |
| DPI scaling strategy | Grid/StackPanel layout with `*` sizing; no hardcoded widths | ❌ Do NOT use pixel-based layouts |
| MVVM mapping (Section 9) | All Views bind to corresponding ViewModels; commands wired | ❌ Do NOT create unbound UI controls |
| Navigation flow (Section 9) | Each navigation path corresponds to documented routes | ❌ Do NOT infer navigation paths |
| Startup view (Section 9) | Root ViewModel loads; initial View renders | ❌ **CRITICAL: If startup view missing, app opens empty** |
| Syncfusion controls (Section 8) | Only validated controls generated; versions match Stage 2 | ❌ Do NOT assume control availability |
| Semantic resources (Section 7) | All controls reference token keys, never hardcoded values | ❌ Do NOT allow inline styling |

**Critical Rules for Stage 5 Execution:**
1. ✅ Use ONLY the design tokens locked in Stage 4
2. ✅ Generate MVVM bindings for every UI control
3. ✅ Follow documented navigation flow exactly
4. ✅ Load startup view automatically
5. ❌ Do NOT infer missing MVVM mappings
6. ❌ Do NOT skip startup view initialization
7. ❌ Do NOT allow incomplete design system
8. ❌ Treat Stage 4 decisions as a pre-implementation contract

**Stage 4 → Stage 5 Handoff:** Validate all Stage 4 decisions are locked before Stage 5 execution begins. If any checklist item (Section 11) is missing or unchecked → **HALT Stage 5 and return to Stage 4 for completion.**